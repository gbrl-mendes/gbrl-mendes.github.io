# Cipó River eDNA Metabarcoding Project

This project evaluates the efficacy of **Environmental DNA (eDNA) metabarcoding** for monitoring fish biodiversity in the Cipó River basin. We compared eDNA results with traditional survey methods (electrofishing and netting) to assess taxonomic coverage, alpha/beta diversity, and the potential of eDNA as a non-invasive monitoring tool.

<div style="text-align: center; margin: 2rem 0;">
  <a href="https://github.com/gbrl-mendes/eDNA_cipo/" class="btn btn-primary" style="background-color: #24292e; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px; font-weight: bold;">
    <i class="fab fa-github"></i> View Full Repository & Code
  </a>
</div>

## 🛠️ Tools & Technologies

*   **Language:** R (v4.3+)
*   **Key Libraries:** `tidyverse` (Data manipulation), `vegan` (Ecological statistics), `phyloseq` (Sequence manipulation), `ggplot2` (Visualization).
*   **Statistical Tests:** Wilcoxon rank-sum test for diversity indices; PERMANOVA for beta diversity.

## 🧬 Methodology Overview

Our bioinformatics pipeline processes raw high-throughput sequencing data (HTS) to identify fish species with high confidence. The workflow involves strict quality control, contamination filtering, and taxonomic assignment using BLASTn against a curated reference database.

### Bioinformatics Pipeline (Summary)

The analysis was performed in **R** using a custom pipeline integrating `dada2`, `phyloseq`, and `vegan`.

| Stage | Description |
| :--- | :--- |
| **1. Raw Screening** | Initial filtering of ASVs by length (120-240bp) and target class (*Actinopteri*). Removal of contaminants found in extraction/PCR controls. |
| **2. Curation** | Validation of taxonomic assignments. Ambiguous hits (e.g., *Brycon aff.*) were manually corrected based on local biogeography. |
| **3. Comparison** | Integration with traditional survey data to compare species richness and composition across 7 sampling sites (SC1–SC7). |

## 📊 Key Results

### 1. Alpha Diversity: eDNA vs. Traditional
eDNA metabarcoding consistently detected equal or higher species richness (Observed) compared to traditional methods across most sampling sites.

[chart:76]

*Figure 1: Comparison of observed species richness between eDNA metabarcoding and traditional methods across sampling sites.*

### 2. Taxonomic Composition (Heatmaps)
We generated compositional heatmaps to visualize the relative abundance of detected taxa. The analysis revealed that eDNA captures cryptic and rare species often missed by traditional gear.

*(Note: High-resolution heatmaps and PCoA ordination plots are available in the [results folder of the repository](https://github.com/gbrl-mendes/eDNA_cipo/tree/main/results/figures)).*

## 📄 Project Structure

The repository is organized to ensure reproducibility:

*   [`scripts/`](https://github.com/gbrl-mendes/eDNA_cipo/tree/main/scripts): Contains the main analysis file (`eDNA_cipo_script.qmd`) and helper functions.
*   [`data/`](https://github.com/gbrl-mendes/eDNA_cipo/tree/main/data): Raw and curated datasets (metabarcoding + traditional).
*   [`results/`](https://github.com/gbrl-mendes/eDNA_cipo/tree/main/results): Generated figures and statistical tables.

---

### Citation & Contact
If you use this code or data, please cite the associated publication (in prep). For questions, please open an issue in the GitHub repository.