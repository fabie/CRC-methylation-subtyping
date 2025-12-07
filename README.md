# Distinguishing Hereditary from Sporadic Colorectal Cancers Using Unsupervised DNA Methylation Clustering

**A proof-of-concept for objective biomarker discovery with potential applications in veteran toxic exposure assessment.**

**Author:** Fabienne C. Van Cappel  
**Program:** Master of Science in Data Science, Regis University, Anderson College of Business and Computing.  
**Course:** MSDS 696 Capstone Project | Fall 2025

---

## The Problem

Military veterans exposed to burn pits and environmental toxins during the Gulf War and post-9/11 deployments face elevated cancer risks—yet lack objective molecular biomarkers to support their VA disability claims. In 2017, more than 80% of veterans' Gulf War Illness claims were denied. In 2024, 36% of veteran disability total claims were dismissed. However, PACT Act claims have improved dramatically with an approval rate of 74.7% in 2024.

To be eligible for toxic exposure disability, service members must have a current diagnosis of one of the twenty presumptive conditions. To satisfy the disability criteria, presumptive conditions do not require service members to prove a direct link between their illness and their military service. Due to lack of objectivity in the clinical assessments, claim approvals are subjective: it is difficult to establish clear causal relationships.

**Can we do better for our service members? Can we objectively assess the diagnosis based on evidence of genetic mutations resulting from exposures, rather than on the etiology of a disease?**

Although the PACT Act established a presumptive framework to reduce the burden of proof for specific conditions, the development of objective biomarkers could reinforce the Veteran Affairs' approach, and possibly broaden eligibility to conditions where exposure links remain unclear.

### From Presumptive Diagnosis to Biogenetic Markers

Current research has largely depended on retrospective analyses, as human studies conducted during active exposure are scarce. So far, investigators have also had difficulty consistently linking objective clinical indicators—such as etiology, lab results, and symptom profiles—to individuals' self-reported exposures. This underscores the need for dependable biomarkers.

---

## Why This Matters

Traditional cancer classification relies on histopathology and known genetic markers. An unsupervised, discovery-oriented approach allows patterns to emerge from the data itself—critical for identifying previously unrecognized subtypes that may correlate with environmental or occupational exposures.

This proof-of-concept demonstrates that unsupervised methylation analysis can recover biologically meaningful cancer subtypes that correspond to different etiological pathways. For veterans, this approach could eventually provide:

- Objective biomarkers linking cancer development to toxic exposures
- Evidence-based support for disability claims
- Personalized insights into cancer etiology beyond family history

**The Challenge:** How do we distinguish cancers driven by environmental exposures from those with hereditary origins?

---

## The Approach

This project applies **unsupervised machine learning** to DNA methylation data—the epigenetic modifications that can reveal environmental "fingerprints" on our genome. Using publicly available data from The Cancer Genome Atlas (TCGA-COAD), I developed a clustering methodology to identify molecularly distinct colorectal cancer subtypes without relying on pre-existing labels.

---

## Data Source & Collection

### Source

- **Repository:** The Cancer Genome Atlas (TCGA) via the Genomic Data Commons (GDC)
- **Cohort:** TCGA-COAD (Colon Adenocarcinoma)
- **Platform:** Illumina HumanMethylation450 BeadChip Array

### Collection Process

Data acquisition required:

- Navigation of GDC portal authentication and data access protocols
- Programmatic download using `TCGAbiolinks` R package
- Integration of methylation data with clinical metadata
- Cohort filtering and sample selection

### Final Dataset

- **Patients:** 292
- **Features:** ~450,000 CpG methylation probes (pre-filtering)
- **Clinical variables:** Age, gender, tumor stage, vital status, molecular markers

---

## Data Characteristics & Challenges

| Challenge | Description | Resolution |
|-----------|-------------|------------|
| High dimensionality | 450K+ probes per sample | Variance filtering |
| Missing data | Incomplete probe coverage, clinical gaps | Threshold-based exclusion |
| Multi-dtype | Beta values, categorical clinical data, survival times | Type-specific preprocessing |
| No ground truth | Unsupervised context—no predefined labels | Internal and external validation strategy |
| Batch effects | Technical variation across samples | Normalization pipeline |

---

## Methodology Overview

| Phase | Description |
|-------|-------------|
| **Data Acquisition** | 292 colorectal adenocarcinoma samples from TCGA |
| **Preprocessing** | 7-step quality control pipeline for methylation data |
| **Feature Selection** | Focused on biologically relevant CpG sites |
| **Clustering** | Consensus clustering to identify stable patient subgroups |
| **Validation** | Internal (silhouette analysis) + External (molecular/clinical markers) |

---

## Quality Control Pipeline (7-8 Steps)

| Step | Process | Purpose |
|------|---------|---------|
| 1 | Sample quality assessment | Retain tumor samples only (code 01), exclude normal tissue |
| 2 | Probe detection p-value filtering | Remove probes with >10% samples failing detection  |
| 3 | SNP probe removal | Remove probes containing known SNPs (Zhou et al., 2017) |
| 4 | Cross-reactive probe removal | Eliminate probes mapping to multiple genomic locations (Chen et al., 2013) |
| 5 | Sex chromosome exclusion | Remove X/Y probes to avoid sex-based clustering artifacts |
| 6 | Beta-Mixture Quantile (BMIQ) normalization | Adjust Type II probe values to match Type I distribution (wateRmelon package) |
| 7 | (Optional) Additional sanitization (for instance imputing NA) |
| 8 | Variance-based selection | Select top 10,000 most variable probes for clustering |

---

## Feature Selection

Data-driven approach:

- Variance calculated across all samples for each probe (`rowVars()`)
- Top 10,000 most variable probes selected for clustering
- Selection based on data-driven ranking rather than intuitive thresholds

---

## Clustering Approach

**Algorithm:** Consensus Clustering (Monti et al., 2003)

Consensus clustering addresses instability in unsupervised methods through repeated perturbation:

1. **Resampling (perturbation):** Each iteration randomly selects 80% of samples and 80% of probes, creating a perturbed view of the data
2. **Iterative clustering:** K-means runs on each of 1,000 subsampled datasets
3. **Consensus matrix construction:** For every sample pair, tracks the proportion of iterations where they co-clustered
4. **Final assignment:** Hierarchical clustering (average linkage) on the consensus matrix determines final cluster membership

**Why this achieves robustness:** If two samples belong to the same true biological cluster, they will consistently co-cluster regardless of which random 80% subset is drawn. Unstable or artificial clusters would show inconsistent pairings across iterations. A high consensus score (e.g., 0.93) indicates samples within clusters co-occur together in ~93% of perturbations.

**Parameters:**

- Number of clusters evaluated: k = 2 – 10
- Resampling proportion: 80% samples × 80% probes
- Iterations per k: 1,000
- Final partitioning: Hierarchical clustering on consensus matrix

---

## Key Findings

The analysis successfully identified **three distinct colorectal cancer subtypes** with strong statistical support:

- **Consensus Score: 0.93** — indicating highly stable, reproducible clusters
- **External Validation:** Significant associations (p < 0.05) with established molecular markers:
  - MSI-High status (microsatellite instability)
  - BRAF mutations
  - KRAS mutations
  - Lynch syndrome mutations
- Clusters aligned with known biological pathways, distinguishing hereditary (Lynch syndrome) from environmentally-driven (CIMP) colorectal cancers

### Cluster Solution

| Metric | Value |
|--------|-------|
| Optimal clusters | 3 |
| Consensus score | 0.93 |
| Stability | High consistency across iterations |

---

## Validation

### Internal Validation

- Silhouette analysis confirms cluster separation
- Cluster stability within acceptable thresholds
- No evidence of forced partitioning

### External Validation

Discovered subtypes show significant associations with established molecular markers:

| Biomarker | Association with Clusters | Statistical Significance |
|-----------|---------------------------|-------------------------|
| MSI-High status | Enriched in Cluster X | p < 0.05 |
| BRAF mutation | Differential distribution | p < 0.05 |
| KRAS mutation | Cluster-specific patterns | p < 0.05 |
| Lynch syndrome mutations | Alignment with hereditary marker | p < 0.05 |

### Clinical Relevance

- Significant age differences across clusters
- Prognostic patterns observed in survival analysis
- Subtypes align with known CRC molecular taxonomy

---

## Project Structure

```
CRC-methylation-subtyping/
│
├── 01_data_acquisition/             # TCGA download and initial processing
│
├── 02_quality_control/              # 7-step QC pipeline
│
├── 03_consensus_clustering/         
│
├── 04_internal_validation/            
│
├── 05_external_validation/
│
├── data/
│   └── README.md                    # instructions for TCGA data download
│
├── results/
│   └── figures/                     # pertinent plots and figures
│   │   └── README.md
│   │   
│   └── tables/
│       └── ref. tables
│
├── .gitignore                       # in R context, which files to ignore and not track
├── LICENSE                          # MIT licence for academic use
├── README.md                        # project man landing page and repository
├── R/
│   └── helper_functions.R           # reusable utility function
│
└── reference_lit.md/                # literature references

```

---

## Tools & Technologies

### Programming Environment

- **Language:** R version 4.5.1 (2025-06-13) "Great Square Root"
- **IDE:** RStudio
- **Documentation:** R Markdown

### Key Packages

| Package | Purpose |
|---------|---------|
| `TCGAbiolinks` | TCGA data acquisition |
| `minfi` | Methylation array preprocessing |
| `wateRmelon` | BMIQ normalization |
| `matrixStats` | Variance calculations |
| `survival` | Survival analysis |
| `ggplot2` | Visualization |
| `dplyr` / `tidyverse` | Data manipulation |
| `renv` | Reproducible environment management |

---

## Roadblocks & Lessons Learned

- **Data complexity:** Methylation arrays contain 450,000+ probes; dimensionality reduction and careful feature selection were essential
- **Validation strategy:** Moving from "clusters exist" to "clusters are meaningful" required systematic external validation against independent molecular and clinical data
- **Interpretability:** Balancing statistical rigor with biological plausibility—ensuring findings translate to real-world clinical relevance

---
## Conclusion
This proof-of-concept demonstrates that methylation-based clustering could provide objective biomarkers for veteran disability claims, offering molecular evidence where subjective clinical assessments currently fall short. Future work will apply this methodology to exposure-related cancers in veteran populations. Subsequently, we can extend this study to the PACT Act, a landmark for veterans who have been exposed to burn pits, Agent Orange, and other toxins, adding over 20 new presumptive conditions - like specific cancers (brain, G, kidney, lymphoma, etc), respiratory illnesses (athma, COPD, chronic bronchitis, sinusitis, etc), and Monoclonal Gammapathy of Undetermined Significance (MGUS) - for Vietnam, Gulf War, and Post-9/11 Veterans, requiring toxic exposure screenings.


## Future Directions

This proof-of-concept establishes the methodological foundation. Next steps include:
- Validation on independent veteran cohorts
- Integration with exposure history data
- Development of a classification model for prospective use
- Will contribute to possibly existing on-going researchs from the U.S government Dept of Veterans Affairs.

---

## About

This project was completed as part of the Master of Science in Data Science program at Regis University's Anderson College of Business and Computing.

**Author:** Fabienne van Cappel  
**Program:** Master of Science in Data Science (MSDS), Regis University  
**Graduation:** Winter 2025

For questions or collaboration inquiries, please open an issue or reach out via GitHub.

---

## License

MIT License
