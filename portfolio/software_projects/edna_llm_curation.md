# Hybrid eDNA Curation: Deterministic Code + LLM-Assisted Review

A command-line system that turns the **raw output of an eDNA metabarcoding run** (a table of DNA sequences with their BLAST hits) into a **curated, auditable species database**. Its core design principle is a strict division of labor: **deterministic code computes every fact it can, and a Large Language Model (LLM) is only called to give a second opinion on the few sequences that code could not resolve on its own**, always reading evidence that was already computed and never overwriting it.

<div style="text-align: center; margin: 2rem 0;">
  <a href="https://github.com/gbrl-mendes/desafio_Amplo/" class="btn btn-primary" style="background-color: #24292e; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px; font-weight: bold;">
    <i class="fab fa-github"></i> View Full Repository & Code
  </a>
</div>

<img src="/portfolio/images/edna_curation_report.png" alt="HTML report generated at the end of a run: final curated table and alpha-diversity charts" style="width:100%; border:1px solid #ddd; border-radius:6px;"/>

*Figure 1: Self-contained HTML report generated at the end of a run (demo dataset). It shows the final curated table and the alpha-diversity charts (observed richness, Shannon, Simpson) for sampling sites SC1–SC7.*

## 🎯 The Problem

After sequencing, a molecular biology facility delivers a single table: one row per DNA sequence (ASV) per sample, each with its three best BLAST matches against a reference database. That raw table is **not** a species list. It contains:

*   **Laboratory contamination** (human DNA, reagent-derived DNA such as *Bos taurus*, cross-sample carry-over);
*   **Off-target amplification** (organisms outside the group of interest);
*   **Weak or uninformative BLAST hits** (e.g., *"Uncultured organism"*), where the top match cannot be trusted;
*   **Taxonomically implausible hits**, where the best match is a species that does not occur in the study region.

Traditionally, an analyst decides sequence by sequence what is noise, contamination, or a valid identification. This system organizes all the evidence needed for that decision, applies the objective rules automatically, and asks an LLM for a reasoned second opinion only where human-like judgment is genuinely required.

## 💡 Why Deterministic Code *and* an LLM

Each tool is used for what it does best, and the boundary between them is explicit:

| | **Deterministic code (R)** | **LLM (via Groq API)** |
| :--- | :--- | :--- |
| **Role** | Computes facts and applies fixed, documented rules | Judges plausibility when the rules are inconclusive |
| **Examples** | Pseudo-score, contamination fold-change, amplicon length, NCBI taxonomy, phylogenetic neighbors, GBIF record counts | *"Does this species make biological and geographic sense at this site, given all the evidence?"* |
| **Reproducibility** | Fully reproducible from a YAML config | Probabilistic, so it is constrained, logged and kept separate |
| **Called for** | Every sequence | **Only** ambiguous sequences |
| **Can overwrite the other?** | Never touched by the LLM | Writes to its own 3 columns only |

Three safeguards keep the LLM from becoming a black box inside the pipeline:

1.  **Gating.** A sequence reaches the LLM only if its deterministic identification was inconclusive (no reliable BLAST hit, identification stopped above species level, or a species with zero regional records in GBIF). Sequences that already failed quality filters (wrong length, out-of-scope taxon, likely contamination) are never sent, because a second taxonomic opinion would add nothing.
2.  **Evidence-bound prompting.** The prompt contains only evidence computed upstream and explicitly instructs the model not to invent evidence. The answer must be strict JSON: an identification, a confidence level (High/Medium/Low) and a short justification that cites the evidence used.
3.  **Side-by-side output.** The LLM's answer lands in three new columns (`Assisted ID (LLM)`, `Assisted Confidence (LLM)`, `Assisted Justification (LLM)`) next to the deterministic columns, so both layers can be audited independently. The result is never silently merged into the original identification.

The system also **degrades gracefully**: without an API key, or with `--llm-mode off`, the LLM stages are skipped and the deterministic result remains complete and valid.

## 🛠️ Tools & Technologies

*   **Orchestration & LLM layer:** Python 3.11+ (`pandas`, `requests`, `PyYAML`, `Jinja2`), `pytest` for tests.
*   **Deterministic curation & ecology:** R (`tidyverse`, `taxize`, `rgbif`, `DECIPHER`, `Biostrings`, `ape`, `vegan`, `plotly`), Quarto for literate documentation, `testthat`.
*   **External sources:** NCBI Taxonomy, GBIF occurrence records, Groq API (open-weight LLM, configurable model).
*   **Reporting:** Self-contained interactive HTML report (DataTables) and PDF rendered through a headless Chromium browser.
*   **Reproducibility:** one-command installers (`setup.ps1` / `setup.sh`), YAML configuration, per-run JSON logs, MkDocs documentation site.

## 🧬 Architecture Overview

```text
 input CSV (long format: one row per ASV × sample)
        │
        ▼
 ┌─────────────────────────────┐
 │ 0. Input validation (Python)│  refuses bad input before anything runs
 └──────────────┬──────────────┘
                ▼
 ┌─────────────────────────────┐
 │ 1. Deterministic curation(R)│  BLAST refinement → taxonomy → contamination
 │                             │  → length → pseudo-score → tree → GBIF
 └──────────────┬──────────────┘
                ▼   (post-run integrity check)
 ┌─────────────────────────────┐
 │ 2. LLM-assisted review      │  only ambiguous ASVs, once per unique ASV
 │    (Python + Groq)          │
 └──────────────┬──────────────┘
                ▼
 ┌─────────────────────────────┐
 │ 3. Curated ID column        │  LLM suggestion if present, else deterministic;
 │                             │  the analyst can override any value
 └──────────────┬──────────────┘
                ▼
 ┌─────────────────────────────┐
 │ 4. Ecological analysis (R)  │  optional, reads only `Curated ID`
 └──────────────┬──────────────┘
                ▼
 ┌─────────────────────────────┐
 │ 5. Reports                  │  JSON log · HTML report · narrative PDF
 └─────────────────────────────┘
```

### Stage 0: Input validation (Python)

Before any computation, `tools/schema_validation.py` checks the table against a declared schema contract. It **refuses to run** if required columns are missing, if the composite primary key (sequence + sample file) is duplicated, or if the file already contains columns that another pipeline pre-computed (such as a ready-made curated answer), which would let a previous result leak into the new one. Softer problems, such as a known quirk where several controls are packed into one cell, produce a warning in the report instead of silently corrupting a join.

### Stage 1: Deterministic curation (R)

The whole rule-based layer lives in a Quarto document (`curadoria_deterministica.qmd`) documented block by block, and is executed as a separate `Rscript` subprocess. It performs:

| Step | What it does |
| :--- | :--- |
| **BLAST hit refinement** | Picks the best of the top 3 hits, skipping uninformative headers (*"Uncultured…"*, synthetic constructs). If all three are unusable, the sequence is flagged `Match_not_reliable` instead of forcing an identification out of a useless hit. |
| **NCBI taxonomy** | Retrieves the full lineage (genus up to superkingdom) for the selected hit via `taxize`. |
| **Contamination check** | Compares each detection with the negative controls referenced by its own sample, using a **fold change** of relative abundances. Below the threshold (default 10), the detection is marked *Possible contamination*. |
| **Amplicon length** | Checks that the sequence length is compatible with the genetic marker (e.g., 140–200 bp for MiFish2). For undeclared primers, the expected range is estimated from the data itself (quartiles ± 1.5×IQR). |
| **Pseudo-score & identification** | Combines alignment identity and coverage, `(10 × identity + coverage) / 11`, and maps it to the **deepest taxonomic level the evidence supports**: species ≥ 98, genus ≥ 95, family ≥ 90, order ≥ 80, class ≥ 60, otherwise *Unidentified*. The system never claims more resolution than the data allows. |
| **Target-taxon flag** | Marks whether the assigned taxon belongs to the group of interest (e.g., ray-finned fishes), so contaminants such as cattle DNA are flagged as out of scope. |
| **Phylogenetic neighborhood** | Aligns all unique sequences (`DECIPHER`), builds a neighbor-joining tree (`ape`) and extracts each sequence's *k* nearest neighbors (default *k* = 5), which is useful context when a hit is ambiguous. |
| **Regional occurrence (GBIF)** | Builds a bounding box from the sample coordinates (plus a buffer) and counts public records of the species inside it. `NA` (not queried) is kept distinct from `0` (queried, no records). The script only fetches the fact; whether it makes biological sense is left to the next stage. |

Every threshold above is a configuration parameter, not hard-coded logic.

### Stage 2: LLM-assisted review (Python + Groq)

`harness/llm_curation.py` aggregates the long table into **one record per unique ASV** (the same sequence appears in many samples with identical BLAST evidence, so it is reviewed once and the answer is propagated to all its rows, which avoids redundant API calls). For each ASV selected by the gating rule, the prompt includes:

*   the raw and filtered BLAST identification, the pseudo-score and which hit was used;
*   the NCBI lineage (genus, family, order, class);
*   the phylogenetic neighbors;
*   the GBIF record count in the study area;
*   in how many real samples the sequence was detected and how many were flagged as possible contamination;
*   sampling site, habitat and river;
*   optionally, species already recorded at the same sites by **traditional field methods** (treated as evidence *in favor* when it matches, and never as a closed list: absence from that list is not evidence of absence).

Engineering details that make this layer robust: low temperature, forced JSON output with a tolerant parser, exponential back-off on rate limits (HTTP 429) and server errors, **per-ASV error isolation** (one failed query never discards the rest), a configurable model and API key (CLI flag, environment variable or `.env`, never committed), and three execution modes: `live`, `mock` (simulated answers to test the whole chain without network or quota) and `off`.

### Stage 3: The `Curated ID` column

The final table always includes `Curated ID`, pre-filled with the LLM suggestion where it exists and with the deterministic identification otherwise (even with the LLM off). It is a **working suggestion**, not a validated truth: the analyst can review and overwrite any value and then run the ecological analysis on the revised file.

### Stage 4: Ecological analysis (R, optional)

`analise_ecologica.qmd` reads only `Curated ID` and never re-identifies anything. It produces observed richness, Shannon and Simpson diversity per site, species accumulation curves, between-site dissimilarity (Bray-Curtis or Jaccard), taxonomic composition at a configurable rank, exclusive and shared taxa across sites, and, when a traditional-survey table is supplied, a direct **eDNA × traditional methods** comparison. Outputs are CSV tables and interactive plots.

### Stage 5: Reports and traceability

*   **JSON run log** with status, validation summary, what the LLM reviewed, warnings and errors, enough to reconstruct what happened without re-running anything.
*   **Self-contained HTML report** (the screenshot above) with MultiQC-style side navigation: run summary, raw input table, output of each deterministic step, LLM-assisted results, final table and ecological plots. It describes only what that specific run actually did.
*   **Narrative PDF report**, also written with the LLM's help but under the same discipline: the model never sees the raw table and never does arithmetic. It receives aggregates already computed in Python and only turns them into prose.

## 🔬 Generality and Validation

*   **Configuration, not code, adapts it to a new project.** Marker, target taxonomic group, column names (`colunas_alias`), thresholds and study area are all set in a YAML file. The repository ships two demo datasets from different domains: **fish from water eDNA** (12S, MiFish2 primer, river basin in Brazil) and **plants from root metabarcoding** (ITS2, no coordinates, so the GBIF check is skipped with a warning).
*   **Graceful degradation at every layer.** Missing R, no API key or a failing external service never invalidates the part of the result that was already computed and verified.
*   **Post-run verification.** Even when R exits successfully, the harness checks row count and the presence of core columns before declaring success.
*   **Tests.** A `pytest` suite covers the orchestration, schema validation, LLM curation (with an injectable fake caller, so no network or key is needed), report generation and the CLI; R's `testthat` covers the GBIF query logic.
*   **Documented limitations.** The repository includes an explicit domain/contract document and a decisions-and-limitations document (for instance, R-side test coverage is still partial, one run corresponds to one project, and LLM review depends on a shared free-tier quota).

## 🚀 How to Run

#### 1. Clone the repository
``` bash
git clone https://github.com/gbrl-mendes/desafio_Amplo.git
cd desafio_Amplo
```
#### 2. Install dependencies (Python venv + R packages)
``` bash
bash setup.sh                                                   # Linux / macOS / WSL
powershell -ExecutionPolicy Bypass -File .\setup.ps1            # Windows
```
#### 3. Run the full pipeline on the demo data
``` bash
.venv/bin/python -m harness data/example/exemplo_1/eDNA_cipo_subset.csv \
  --reference data/example/exemplo_1/spp_tradicional.csv \
  --ecologia --groq-api-key <your-key>
```
The LLM stages need a free [Groq](https://console.groq.com/keys) key. Without one (or with `--llm-mode off`), the deterministic curation and the ecological analysis still run. Each run writes its CSV, logs, plots and reports to `runs/<run_id>/`, and the HTML report opens automatically in the browser.

### Contact
For more information, contact me through my [e-mail](mailto:gabrielmendesbrt@outlook.com) 😊

---
---

## **Curadoria Híbrida de eDNA: Código Determinístico + LLM (pt-BR)**

### Sobre
Sistema de linha de comando que transforma a **saída bruta de um sequenciamento de metabarcoding de eDNA** (uma tabela de sequências de DNA com seus resultados de BLAST) em uma **base de espécies curada e auditável**. O princípio central é uma divisão de trabalho estrita: **o código determinístico calcula todos os fatos que consegue, e um modelo de linguagem (LLM) só é chamado para dar uma segunda opinião nas poucas sequências que o código não conseguiu resolver sozinho**, sempre lendo evidências já calculadas e nunca sobrescrevendo-as.

### O problema
Depois do sequenciamento, a facility entrega uma tabela com uma linha por sequência (ASV) por amostra, cada uma com os três melhores hits de BLAST. Essa tabela **não é** uma lista de espécies: ela traz contaminação de laboratório, amplificação de organismos fora do grupo de interesse, hits fracos ou pouco informativos (ex.: *"Uncultured organism"*) e hits de espécies que não ocorrem na região. Tradicionalmente, o analista decide sequência por sequência o que é ruído, contaminação ou identificação válida. O sistema organiza a evidência para essa decisão, aplica automaticamente as regras objetivas e só aciona o LLM onde é preciso julgamento.

### Por que código determinístico *e* LLM

| | **Código determinístico (R)** | **LLM (API da Groq)** |
| :--- | :--- | :--- |
| **Papel** | Calcula fatos e aplica regras fixas e documentadas | Julga plausibilidade quando as regras não são conclusivas |
| **Exemplos** | Pseudo-score, fold change de contaminação, tamanho do amplicon, taxonomia NCBI, vizinhos filogenéticos, contagem de registros no GBIF | *"Esta espécie faz sentido biológico e geográfico neste ponto, dadas todas as evidências?"* |
| **Reprodutibilidade** | Total, a partir de um YAML de configuração | Probabilístico, por isso restrito, registrado e mantido separado |
| **Acionado para** | Todas as sequências | **Somente** as ambíguas |
| **Sobrescreve o outro?** | Nunca é alterado pelo LLM | Escreve apenas em 3 colunas próprias |

Três salvaguardas impedem que o LLM vire uma caixa-preta dentro do pipeline:

1.  **Filtro de acionamento.** Uma sequência só chega ao LLM se a identificação determinística foi inconclusiva (sem hit confiável, identificação parou acima de espécie, ou espécie sem nenhum registro regional no GBIF). Sequências já reprovadas nos filtros de qualidade (tamanho, táxon fora do escopo, provável contaminação) nunca são enviadas.
2.  **Prompt preso à evidência.** O prompt contém só evidência calculada antes e instrui o modelo a não inventar nada. A resposta é um JSON estrito: identificação, nível de confiança (Alta/Média/Baixa) e uma justificativa curta que cita a evidência usada.
3.  **Saída lado a lado.** A resposta vai para três colunas novas (`Assisted ID (LLM)`, `Assisted Confidence (LLM)`, `Assisted Justification (LLM)`), ao lado das colunas determinísticas, permitindo auditar as duas camadas separadamente.

O sistema também **degrada com elegância**: sem chave de API, ou com `--llm-mode off`, as etapas de LLM são puladas e o resultado determinístico continua completo e válido.

### Etapas do pipeline

*   **0. Validação da entrada (Python):** recusa a execução se faltarem colunas obrigatórias, se a chave primária composta estiver duplicada ou se o arquivo já trouxer colunas calculadas por outro pipeline (vazamento de gabarito). Problemas menores viram aviso no relatório.
*   **1. Curadoria determinística (R):** refinamento dos hits de BLAST (descarta cabeçalhos não informativos), taxonomia NCBI, checagem de contaminação por fold change contra os controles, faixa de tamanho do amplicon por marcador, **pseudo-score** `(10 × identidade + cobertura) / 11` mapeado para o nível taxonômico mais profundo que a evidência sustenta (espécie ≥ 98, gênero ≥ 95, família ≥ 90, ordem ≥ 80, classe ≥ 60, senão *Unidentified*), flag de táxon-alvo, árvore filogenética ASV-contra-ASV com os *k* vizinhos mais próximos e contagem de registros regionais no GBIF (diferenciando `NA` de `0`). Todos os limiares são parâmetros de configuração.
*   **2. Curadoria assistida por LLM (Python + Groq):** a tabela é agregada em **um registro por ASV única** (a mesma sequência aparece em várias amostras com a mesma evidência, então é revisada uma vez e a resposta é propagada). O prompt reúne: identificação do BLAST e pseudo-score, linhagem NCBI, vizinhos filogenéticos, registros no GBIF, em quantas amostras a sequência apareceu e quantas foram marcadas como possível contaminação, ponto/habitat/rio e, opcionalmente, espécies já registradas nos mesmos pontos por métodos tradicionais (evidência **a favor**, nunca lista fechada). Há temperatura baixa, saída JSON com parser tolerante, nova tentativa com espera progressiva em erro 429/5xx, isolamento de erro por ASV, modelo e chave configuráveis (flag, variável de ambiente ou `.env`) e três modos: `live`, `mock` e `off`.
*   **3. Coluna `Curated ID`:** pré-preenchida com a sugestão do LLM quando existe e com a identificação determinística caso contrário. É uma **sugestão de trabalho**, não uma verdade validada: o analista pode sobrescrever qualquer valor.
*   **4. Análise ecológica (R, opcional):** lê apenas a `Curated ID` e gera riqueza, Shannon, Simpson, curva de acumulação, dissimilaridade entre pontos (Bray-Curtis ou Jaccard), composição taxonômica, táxons exclusivos/compartilhados e comparação **eDNA × métodos tradicionais**.
*   **5. Relatórios:** log JSON da execução, relatório HTML autocontido (a imagem acima) e relatório narrativo em PDF. Neste último, o LLM nunca vê a tabela bruta nem faz conta: recebe agregados já calculados em Python e apenas redige o texto.

### Generalidade e validação
O sistema é adaptado a um novo projeto por **configuração, não por código**: marcador, grupo taxonômico, nomes de coluna e limiares ficam em um YAML. O repositório traz dois conjuntos de demonstração de domínios diferentes (**peixes** por eDNA de água, marcador 12S/MiFish2; e **plantas** por metabarcoding de raízes, ITS2). Há verificação pós-execução da saída do R, suíte de testes em `pytest` (com chamador de LLM falso, sem rede nem chave) e `testthat` para a consulta ao GBIF, além de documentos explícitos de domínio e de decisões e limitações.

### Instruções
#### 1. Clone o repositório:
``` bash
git clone https://github.com/gbrl-mendes/desafio_Amplo.git
cd desafio_Amplo
```
#### 2. Instale as dependências:
``` bash
bash setup.sh                                                   # Linux / macOS / WSL
powershell -ExecutionPolicy Bypass -File .\setup.ps1            # Windows
```
#### 3. Execute o pipeline completo com os dados de demonstração:
``` bash
.venv/bin/python -m harness data/example/exemplo_1/eDNA_cipo_subset.csv \
  --reference data/example/exemplo_1/spp_tradicional.csv \
  --ecologia --groq-api-key <sua-chave>
```
As etapas com LLM precisam de uma chave gratuita da [Groq](https://console.groq.com/keys). Sem ela (ou com `--llm-mode off`), a curadoria determinística e a análise ecológica continuam rodando. Cada execução grava CSV, logs, gráficos e relatórios em `runs/<run_id>/`, e o relatório HTML abre sozinho no navegador.

### Contato
Para mais informações, entre em contato comigo através do meu endereço de [e-mail](mailto:gabrielmendesbrt@outlook.com) 😊
