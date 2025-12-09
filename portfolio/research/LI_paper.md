---
layout: post
title: "Hydrology-Induced Changes in Fish Ecology"
description: "Investigating how water level fluctuations drive fish community dynamics in a Neotropical reservoir using eDNA metabarcoding."
date: 2025-12-09
image: /assets/images/projects/lagoa_ingleses_cover.jpg # Adicione se tiver
---

# Hydrology-Induced Changes Drive Fish eDNA Ecology in a Neotropical Reservoir

This study investigates the ecological dynamics of fish communities in the *Lagoa dos Ingleses* reservoir under varying hydrological conditions. By leveraging **eDNA metabarcoding**, we assessed how water level fluctuations (High vs. Low) influence species richness (alpha diversity) and community composition (beta diversity).

<div style="text-align: center; margin: 2rem 0;">
  <a href="#" class="btn btn-primary" style="background-color: #24292e; color: white; padding: 10px 20px; text-decoration: none; border-radius: 5px; font-weight: bold;">
    <i class="fab fa-github"></i> Project Repository (Coming Soon)
  </a>
</div>

## 🧬 Study Design & Methodology

We monitored 4 sampling sites (P1–P4) across multiple campaigns (2020–2022), capturing distinct hydrological phases.

### Bioinformatics Workflow
Our custom pipeline integrates `dada2` for exact sequence inference and `phyloseq` for ecological analysis.

| Step | Method | Key Action |
| :--- | :--- | :--- |
| **1. Sequencing** | Illumina MiSeq | Paired-end metabarcoding of the *12S rRNA* gene (MiFish primers). |
| **2. Denoising** | DADA2 | Inference of Amplicon Sequence Variants (ASVs) with strict quality filtering. |
| **3. Curation** | BLASTn + Expert Review | Manual curation of taxonomic assignments to resolve ambiguous species (*e.g., Astyanax vs. Psalidodon*). |
| **4. Statistics** | Vegan / PERMANOVA | Testing the effect of water levels on community structure. |

## 📊 Key Findings

### 1. Alpha Diversity & Water Levels
We observed shifts in species richness associated with reservoir water levels. The boxplot below illustrates the observed richness (number of species) detected in High vs. Low water periods.

[chart:78]

*Figure 1: Comparison of Alpha Diversity (Richness) between High and Low water levels. Fluctuations in water volume appear to influence the detectability and presence of distinct fish assemblages.*

### 2. Beta Diversity (Community Structure)
Principal Coordinates Analysis (PCoA) based on Jaccard distances revealed distinct clustering of samples by water level.
*   **High Level:** Communities were more homogenized.
*   **Low Level:** Greater heterogeneity, likely due to habitat fragmentation or concentration effects.
*   **Statistical Significance:** Confirmed via PERMANOVA (`adonis2`), showing that hydrological phase is a significant driver of community composition.

### 3. Rarefaction & Sampling Effort
Species accumulation curves (Collector's curves) demonstrated that our sampling effort was sufficient to capture the majority of the local biodiversity, with the curve reaching an asymptote for both hydrological periods.

## 🛠️ Computational Tools

*   **R Packages:** `phyloseq` (Microbiome analysis), `vegan` (Ordination & Diversity), `ggplot2` (Data Visualization), `DECIPHER` (Sequence alignment).
*   **Reproducibility:** All analyses are scripted in R Markdown/Quarto, ensuring full transparency from raw reads to final figures.

---

### Citation & Contact
This work is part of the manuscript *"Analyses of Hydrology-Induced Changes Drive Fish eDNA Ecology in a Neotropical Reservoir"*.
For more information, please contact: [gabrielmendesbrt@gmail.com](mailto:gabrielmendesbrt@gmail.com)