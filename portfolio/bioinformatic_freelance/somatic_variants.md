## Somatic Variants Consulting (en-US) 

### About
This project implements a pipeline developed to address the exercise of group 2 "somatic variants". The objective is to perform somatic variant filtering and preparation for Cancer Genome Interpreter (CGI) analysis.

The script developed is available at `/variantes_somaticas/script_variantes_somaticas.sh`.

#### Main objectives:

- Filters VCF files to retain only clinically relevant somatic variants

- Applies filters based on population frequency (MAX_AF ≤ 0.01), impact (excluding LOW), clinical significance (pathogenic, not benign), and gene panel matching

- Generates TSV files formatted for CGI interpretation

- Processes multiple samples in batch mode

#### Key steps:

- Removes variants without consequence annotations (CSQ)

- Filters variants using Ensembl VEP criteria

- Extracts and formats variant data into TSV with specific columns

- Ensures adequate sequencing depth (DP≥20) and allele frequency (AF≥0.1)

The pipeline prepares somatic variant data from annotated VCFs into analysis-ready formats for cancer genomics interpretation.

### Instructions
#### 1. Clone the repository:
``` bash
git clone https://github.com/gbrl-mendes/variantes_somaticas.git
```
#### 2. Access the script:
``` bash
cd variantes_somaticas/ && vi script_variantes_somaticas.sh
```
#### 3.Edit the information:
``` bash
# Define main directories
BCFTOOLS_DIR="/home/gabriel/downloads/bcftools-1.21"
FILTER_VEP_DIR="/home/gabriel/downloads/ensembl-vep"
PROJECT_DIR="/home/gabriel/projetos/variantes_somaticas" 
INPUT_DIR="/home/gabriel/projetos/variantes_somaticas/liftOver-hg38-MF-annotVep"
GENES_FILE="/home/gabriel/projetos/variantes_somaticas/genes/myelofibrosis.txt"
```
#### 4. Run the script:
``` bash
/variantes_somaticas$ ./script_variantes_somaticas.sh
```
The script executes all the filtering steps of the VCF file and the creation of the TSV files so that the results can be interpreted in the Cancer Genome Interpreter (CGI).

The annotation results are made available in the /variantes_somaticas/output directory.

### Contact 
For more information, contact me through my [e-mail](mailto:gabrielmendesbrt@outllok.com) 😊