## Data Directory
This directory contains instructions for acquiring the TCGA-COAD methylation data used in this analysis. Due to data use agreements and file size (~8 GB), raw data files are not included in this repository.

## Data Source
Repository: The Cancer Genome Atlas (TCGA)
Portal: Genomic Data Commons (GDC)
Project: TCGA-COAD (Colon Adenocarcinoma)
Platform: Illumina HumanMethylation450 BeadChip Array

## Prerequisites
R Version
R version 4.0+ recommended (this project used R 4.5.1)
Required Packages
# Install Bioconductor manager if needed
if (!require("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

# Install TCGAbiolinks
BiocManager::install("TCGAbiolinks")

# Additional packages for preprocessing
BiocManager::install(c("minfi", "wateRmelon", "sesame"))
install.packages(c("matrixStats", "dplyr"))

Data Download Instructions
Step 1: Set Up Directory Structure
# Create project directories
dir.create("00_Raw_Data", showWarnings = FALSE)
dir.create("01_Processed_Data", showWarnings = FALSE)

setwd("00_Raw_Data")
Step 2: Download Clinical Data (~1 minute)
library(TCGAbiolinks)

# Query and download clinical data
clinical <- GDCquery_clinic(project = "TCGA-COAD", type = "clinical")

# Save
saveRDS(clinical, "COAD_clinical.rds")
write.csv(clinical, "COAD_clinical.csv", row.names = FALSE)

cat("Clinical data:", nrow(clinical), "patients\n")
Expected output: ~450–520 patients
Step 3: Download Mutation Data (~15 minutes)
# Query mutation data
query_mut <- GDCquery(
    project = "TCGA-COAD",
    data.category = "Simple Nucleotide Variation",
    data.type = "Masked Somatic Mutation",
    workflow.type = "Aliquot Ensemble Somatic Variant Merging and Masking"
)

# Download and prepare
GDCdownload(query_mut)
mutation <- GDCprepare(query_mut)

# Save
saveRDS(mutation, "COAD_mutations.rds")

cat("Mutation data:", nrow(mutation), "mutations\n")
Expected output: ~85,000–250,000 mutations
Step 4: Download Methylation Data (~45–60 minutes)
# Query methylation data (450K platform)
query_meth <- GDCquery(
    project = "TCGA-COAD",
    data.category = "DNA Methylation",
    platform = "Illumina Human Methylation 450",
    data.type = "Methylation Beta Value"
)

# Download (large file, ~8 GB)
GDCdownload(query_meth, method = "api", directory = "GDC_downloads")

# Prepare into usable format (10–20 minutes)
methylation <- GDCprepare(query_meth, directory = "GDC_downloads")

# Save
saveRDS(methylation, "COAD_methylation_raw.rds")

cat("Methylation data:", nrow(methylation), "probes x", ncol(methylation), "samples\n")
