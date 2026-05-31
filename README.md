# CS4C — Cyber Security Four Layers for CVSS Contextualization

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CVSS v3.1](https://img.shields.io/badge/CVSS-3.1-blue)](https://www.first.org/cvss/v3-1/)
[![DOI](https://img.shields.io/badge/DOI-pending-lightgrey)](https://doi.org/)

## Project Description

**CS4C** (Cyber Security Four Layers for CVSS Contextualization) is a methodological framework designed to address the persistent challenge of **unpatched vulnerabilities** in operational environments. The model extends the standard CVSS v3.1 base metrics by incorporating a structured four-layer contextualization pipeline that leverages **environmental metrics** and the **Exploit Prediction Scoring System (EPSS)** to produce actionable, priority-aware vulnerability scores.

This repository contains the complete reproducible research package for the article *"CS4C: Cyber Security Four Layers for CVSS Contextualization"*, including datasets, calculation templates, methodological documentation, and figure sources.

## Abstract

Vulnerability management in real-world infrastructures is often hindered by the impossibility of applying patches — due to operational continuity requirements, legacy system constraints, or lack of vendor support. Traditional CVSS scoring provides a static severity assessment that does not reflect the specific risk context of a given deployment. CS4C addresses this gap by defining four sequential contextualization layers:

1. **Layer 1 — Base Vector Standardization**: Normalization of the CVSS base vector to ensure consistent scoring across heterogeneous CVE entries.
2. **Layer 2 — Environmental Metrics Tuning**: Customization of CVSS v3.1 environmental metrics (Confidentiality Requirement, Integrity Requirement, Availability Requirement, Modified Attack Vector, Modified Attack Complexity, etc.) to reflect the target infrastructure profile.
3. **Layer 3 — EPSS Integration**: Incorporation of EPSS probability scores to filter vulnerabilities with negligible exploitation likelihood.
4. **Layer 4 — Prioritization Matrix**: Construction of a composite priority score combining the environmental CVSS score and EPSS data for decision support.

## Objectives

- Propose a reproducible methodology for CVSS contextualization tailored to unpatched vulnerability scenarios.
- Define a four-layer pipeline that integrates CVSS v3.1 environmental metrics with EPSS threat intelligence.
- Provide open-source datasets, calculations, and evaluation procedures to enable full reproducibility of the experimental results.
- Establish a baseline for future research on contextual vulnerability scoring in air-gapped, legacy, or high-availability environments.

## Methodology

The research follows a structured experimental protocol:

1. **CVE Selection**: A curated set of CVEs with known unpatched instances is extracted from the NVD and CVE databases, filtered by relevance to common enterprise infrastructure components.
2. **Baseline Construction**: Each selected CVE is assigned its standard CVSS v3.1 base vector and base score, forming the reference baseline.
3. **Environmental Modification**: Environmental metrics are adjusted according to the target deployment scenario, following the CVSS v3.1 specification for Modified Base Metrics and Environmental Metrics.
4. **Environmental Score Calculation**: The environmental CVSS score is computed using the CVSS v3.1 environmental formula, incorporating the Modified Impact, Exploitability, and Temporal score groups.
5. **EPSS Integration**: EPSS probability and percentile values are retrieved from the FIRST.org EPSS model and combined with the environmental score to produce the final prioritization metric.

## Repository Structure

```
CS4C/
│
├── README.md                 # This file — project overview and reproduction guide
├── LICENSE                   # MIT License
├── CITATION.cff              # Citation metadata (CFF format)
├── .gitignore                # Git exclusion rules
│
├── data/                     # Datasets used in the study
│   ├── cve_dataset.csv       # Selected CVEs with base scores and metadata
│   ├── baseline_vectors.csv  # CVSS v3.1 base vectors for the baseline
│   └── environmental_vectors.csv  # Environmental vectors after Layer 2 modification
│
├── calculations/             # Calculation artifacts
│   ├── cvss_calculations.xlsx  # Spreadsheet with all CVSS computations
│   └── formulas.pdf            # Formal definition of the scoring formulas
│
├── figures/                  # Figure sources and rendered outputs
│   ├── figure1.png           # CS4C architectural overview diagram
│   └── figure2.png           # Results comparison chart (baseline vs environmental)
│
├── methodology/              # Detailed methodological documentation
│   ├── evaluation_procedure.md   # Step-by-step evaluation protocol
│   └── environment_definition.md # Target environment specification
│
└── paper/                    # Published article
    └── RCCI7.0.pdf           # Full article in PDF format
```

## How to Reproduce the Results

### Prerequisites

- Python 3.8+ with `pandas`, `numpy`, and `cvss` (CVSS calculator library).
- Access to the [NVD API](https://nvd.nist.gov/developers) and [FIRST EPSS API](https://www.first.org/epss/api).
- A spreadsheet application (LibreOffice Calc or Microsoft Excel) for `cvss_calculations.xlsx`.

### Reproduction Steps

1. **Clone the repository**:
   ```bash
   git clone https://github.com/your-org/CS4C.git
   cd CS4C
   ```

2. **Verify datasets**:
   Inspect the CSV files in `data/` to confirm CVE identifiers, base vectors, and environmental modifications match the article specification.

3. **Validate environmental calculations**:
   Open `calculations/cvss_calculations.xlsx` and verify that the environmental scores for each CVE match the values reported in the article. The formulas follow the CVSS v3.1 specification as defined in `calculations/formulas.pdf`.

4. **Reproduce figures**:
   The figures in `figures/` can be regenerated using the provided datasets. The plotting scripts are available upon request (see Contact section).

5. **Cross-check with EPSS**:
   Query the EPSS API for each CVE and confirm the probability values used in Layer 3 match the data in `data/environmental_vectors.csv`.

### Expected Output

Following these steps will yield the same environmental scores, priority rankings, and result figures presented in the article. Any deviation indicates a potential error in the reproduction procedure.

## Reference

For the full scientific context, please refer to the published article:

> *"CS4C: Cyber Security Four Layers for CVSS Contextualization"* — RCCI 7.0 Conference Proceedings. Available at `paper/RCCI7.0.pdf`.

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

## Citation

If you use CS4C in your research, please cite it as indicated in the [CITATION.cff](CITATION.cff) file.

## Contact

For questions, issues, or collaboration inquiries, please open an issue on this repository or contact the corresponding author as listed in the article.
