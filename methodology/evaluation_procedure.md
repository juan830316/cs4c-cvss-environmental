# Evaluation Procedure

## Overview

This document describes the step-by-step evaluation protocol used in the CS4C study. The procedure is designed to be fully reproducible and is organized in four sequential stages corresponding to the four layers of the CS4C framework.

## Prerequisites

- Access to the [NVD API 2.0](https://nvd.nist.gov/developers) for CVE metadata retrieval.
- Access to the [FIRST EPSS API](https://www.first.org/epss/api) for exploitation probability data.
- CVSS v3.1 calculator implementation (reference: [FIRST CVSS v3.1 Specification](https://www.first.org/cvss/v3-1/specification-document)).
- Python 3.8+ with `requests`, `pandas`, `numpy`, and `cvss` libraries.

---

## Stage 1: CVE Selection (Layer 1 — Base Vector Standardization)

### Objective
Select a representative set of CVEs corresponding to unpatched vulnerabilities in common enterprise infrastructure components and normalize their base vectors.

### Procedure

1. **Query the NVD API** for CVEs published between the study's time window.
   - Use the filter `lastModifiedDate` and `cvssV3Severity` to pre-filter candidates.
   - Include vulnerabilities with available CVSS v3.1 base vectors.

2. **Apply inclusion criteria**:
   - Vulnerability must have at least one known unpatched instance reported in public advisories.
   - Vulnerability must affect a component present in the target environment (see [environment definition](environment_definition.md)).
   - CVSS v3.1 base vector must be complete and properly formed.

3. **Build the CVE dataset** (`data/cve_dataset.csv`):
   - Extract `CVE ID`, `CVSS Base Score`, `Base Vector`, `EPSS Probability`, and `EPSS Percentile`.

4. **Normalize base vectors**:
   - Verify that all vectors conform to the CVSS v3.1 string format.
   - Standardize any incomplete or malformed vectors by inferring default values according to the CVSS specification.

### Expected Outcome
A curated CSV file containing `n` CVEs with clean, standardized base vectors.

---

## Stage 2: Baseline Construction (Layer 1 — Temporal Score Addition)

### Objective
Augment the base vectors with temporal metrics to establish the starting baseline.

### Procedure

1. **Assign temporal metrics** to each CVE based on the current threat landscape at the time of the study:
   - `E` (Exploit Code Maturity): Set to `U` (Unproven) unless public exploit evidence exists.
   - `RL` (Remediation Level): Set to `O` (Official Fix) or `W` (Workaround) depending on patch availability.
   - `RC` (Report Confidence): Set to `C` (Confirmed).

2. **Compute the temporal score** for each CVE using the CVSS v3.1 temporal formula:
   - `TemporalScore = roundup(BaseScore × Exploitability × RemediationLevel × ReportConfidence)`.

3. **Record the baseline vectors** in `data/baseline_vectors.csv` with columns:
   - `CVE`, `CVSS_Vector_Base`, `Base_Score`, `Severity`, `Temporal_Vector`, `Temporal_Score`, `Layer_Applied`, `Notes`.

### Expected Outcome
A baseline CSV that captures the current temporal state of each vulnerability before environmental customization.

---

## Stage 3: Environmental Metrics Modification (Layer 2)

### Objective
Customize CVSS v3.1 environmental metrics to reflect the specific security requirements and infrastructure configuration of the target environment.

### Procedure

1. **Define the target environment profile** (see [environment definition](environment_definition.md)):
   - Classify each asset's criticality using CIA requirements: `CR`, `IR`, `AR` (Low, Medium, High).
   - Document network accessibility constraints: `MAV`, `MAC`, `MPR`, `MUI`.

2. **For each CVE**, construct an environmental vector by appending the following metrics to the base vector:
   - **Modified Base Metrics**: `MAV`, `MAC`, `MPR`, `MUI`, `MS`, `MC`, `MI`, `MA` — values inherited from the base vector unless modified by environment-specific constraints.
   - **Environmental Metrics**: `CR`, `IR`, `AR` — set according to the asset criticality classification.

3. **Compute the environmental score** using the CVSS v3.1 environmental formula:

   ```
   EnvironmentalScore = roundup(
       roundup(min(1.0, 1 - (1 - CR × MC) × (1 - IR × MI) × (1 - AR × MA))) ×
       ExploitabilitySubScore ×
       TemporalScore
   )
   ```

   where `CR`, `IR`, `AR` are the requirement weights and `MC`, `MI`, `MA` are the modified impact sub-scores.

4. **Document modifications** in `data/environmental_vectors.csv` with columns:
   - `CVE`, `Environmental_Vector`, `Environmental_Score`, `Severity`, `Modified_Base_Score`, `EPSS_Probability`, `EPSS_Percentile`, `Composite_Priority`, `Layer_Applied`, `Notes`.

### Expected Outcome
A CSV file with environmental scores that reflect the target infrastructure's risk posture.

---

## Stage 4: EPSS Integration (Layer 3)

### Objective
Incorporate EPSS probability scores to filter out vulnerabilities with negligible exploitation likelihood.

### Procedure

1. **Retrieve EPSS data** for each CVE via the FIRST EPSS API:
   ```
   GET https://api.first.org/data/v1/epss?cve=CVE-2023-XXXX1
   ```
   Extract the `epss` (probability) and `percentile` fields.

2. **Apply the EPSS filter threshold**:
   - Vulnerabilities with `EPSS probability < 0.0004` (approximately the 20th percentile) are flagged as low-priority.
   - This threshold is configurable; the article uses the standard threshold recommended by FIRST.

3. **Update the composite priority** in `data/environmental_vectors.csv`.

### Expected Outcome
EPSS-enriched vulnerability records with a preliminary prioritization filter.

---

## Stage 5: Prioritization Matrix Construction (Layer 4)

### Objective
Combine the environmental CVSS score and EPSS data into a unified composite priority metric.

### Procedure

1. **Normalize the environmental score** to a 0–10 scale (already inherent in CVSS).

2. **Normalize the EPSS probability** to a 0–10 scale:
   ```
   EPSS_norm = EPSS_probability × 10
   ```

3. **Compute the composite priority score**:
   ```
   Composite_Priority = Environmental_Score × EPSS_norm
   ```

   This product emphasizes vulnerabilities that are both contextually severe (high environmental score) and actively threatened (high EPSS probability).

4. **Rank all CVEs** by `Composite_Priority` in descending order.

### Expected Outcome
A ranked priority list that drives remediation decision-making.

---

## Validation

### Cross-Verification Steps

1. **Formula verification**: Recompute all CVSS scores using the [FIRST CVSS v3.1 Calculator](https://www.first.org/cvss/calculator/3.1) to confirm environmental score values.

2. **EPSS cross-check**: Query the EPSS API independently and compare the returned probability values with those recorded in the dataset.

3. **Sensitivity analysis**: Vary the EPSS threshold parameter and observe the impact on the final priority ranking.

### Reproducibility Checklist

- [ ] All CVSS vectors conform to the v3.1 string format.
- [ ] Environmental scores match the FIRST calculator output.
- [ ] EPSS values are timestamped and linked to the API query date.
- [ ] The complete dataset, including all intermediate vectors, is committed to the repository.
- [ ] The computational environment (Python version, library versions) is documented.
