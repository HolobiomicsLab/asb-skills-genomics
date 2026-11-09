---
name: moreau-broto-autocorrelation-feature-extraction
description: Use when encoding protein sequences into fixed-length physicochemical
  autocorrelation features (e.g., as one descriptor block of a multi-view feature
  vector for protein classification such as DBP detection).
when_to_use_negative:
- The input is already a numeric feature matrix (e.g., precomputed DDE or PSSM-DBT
  features) rather than raw protein sequences
- The downstream pipeline requires PSSM-derived features (use PSSM-DBT or PSSM-DWT
  instead); NMBAC derives only from sequence-level physicochemical properties
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: iDBP-DEP NMBAC module (BAC.m, PYH_6.m, phy_6.mat)
  role: MATLAB implementation that standardizes the six physicochemical property sets
    and computes the normalized Moreau-Broto autocorrelation at lags 1–30
  repo: https://githup.com/Zll-codeside/iDBP-DEP
- name: iFeature
  role: Standalone toolkit that also facilitates generation of sequence-based features
    such as autocorrelation descriptors
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:d42a04267f2aaf29ce40ed5fd62c18c9b5829f24e8ef30cc442d28b46ec767bf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PSI-BLAST
  - iDBP-DEP NMBAC module (BAC.m, PYH_6.m, phy_6.mat)
  - iFeature
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# moreau-broto-autocorrelation-feature-extraction

## Summary

Computes the Normalized Moreau-Broto Autocorrelation (NMBAC) descriptor — a 200-dimension physicochemical-property autocorrelation feature vector (180 autocorrelation terms plus 20 amino acid frequencies) — from protein sequences, as implemented in the iDBP-DEP MATLAB module (BAC.m, PYH_6.m, phy_6.mat) for DNA-binding protein prediction.

## When to use

Use when encoding protein sequences into fixed-length physicochemical autocorrelation features (e.g., as one descriptor block of a multi-view feature vector for protein classification such as DBP detection).

## When NOT to use

- The input is already a numeric feature matrix (e.g., precomputed DDE or PSSM-DBT features) rather than raw protein sequences
- The downstream pipeline requires PSSM-derived features (use PSSM-DBT or PSSM-DWT instead); NMBAC derives only from sequence-level physicochemical properties

## Inputs

- Protein sequences from benchmark datasets PDB1075, PDB594, or PDB186
- Amino acid physicochemical property table (six properties for 20 amino acids) supplied as phy_6.mat from the AAindex Database
- nlag parameter (set to 30 in iDBP-DEP)

## Outputs

- 200-dimension NMBAC feature vector per protein (180 autocorrelation terms + 20 amino acid frequencies)
- Feature matrix saved as feature_NMBAC_workspace.mat (e.g., feature_NMBAC_186.mat, feature_NMBAC_594.mat)

## How to apply

Load the protein sequences from the benchmark datasets (PDB1075/PDB594) and retrieve six amino acid physicochemical properties (hydrophobicity, solvent-accessible surface area, side-chain volume, polarity, polarizability, net charge index) from the AAindex Database via phy_6.mat. Standardize each property value set across the 20 amino acids using its mean and standard deviation, then compute the normalized Moreau-Broto autocorrelation for each property at lags d = 1 to nlag = 30, yielding 6 × 30 = 180 features. Append the frequency of each of the 20 natural amino acids to reach the final 200-dimension vector (30 × 6 + 20 = 200). In the iDBP-DEP codebase this is done with the MATLAB scripts NMBAC/BAC.m and NMBAC/PYH_6.m; the nlag = 30 setting was chosen via 10-fold cross validation, and the resulting matrix is saved as feature_NMBAC_workspace.mat for later combination with DDE, PSSM-DBT, and PSSM-DWT descriptors into a 3240-dimension vector.

## Related tools

- **iDBP-DEP NMBAC module (BAC.m, PYH_6.m, phy_6.mat)** (MATLAB implementation that standardizes the six physicochemical property sets and computes the normalized Moreau-Broto autocorrelation at lags 1–30) — https://githup.com/Zll-codeside/iDBP-DEP
- **iFeature** (Standalone toolkit that also facilitates generation of sequence-based features such as autocorrelation descriptors)

## Examples

```
matlab -batch "addpath('Code/feature_Extract/NMBAC'); run PYH_6.m; save('feature_NMBAC_workspace.mat')"
```

## Evaluation signals

- Every protein yields exactly a 200-dimension vector: 180 autocorrelation terms (6 properties × nlag = 30) plus 20 amino acid frequencies
- Output .mat files (feature_NMBAC_workspace.mat, feature_NMBAC_186.mat, feature_NMBAC_594.mat) exist in the expected workspace/Data directories per the README file structure
- The NMBAC block concatenates with DDE + PSSM-DBT + PSSM-DWT to give the reported 3240-dimension combined feature vector (NMBAC contributing 200 dimensions)
- Property values are standardized (zero mean, unit variance across the 20 amino acids) before autocorrelation is computed
- nlag is fixed at 30, the value selected by 10-fold cross validation in the paper

## Limitations

- Descriptor dimensionality depends on nlag; the paper selected nlag = 30 via 10-fold cross validation, so other lag settings change the vector size and may degrade performance
- NMBAC captures only sequence-level physicochemical properties and does not encode evolutionary information (PSSM-based descriptors fill that role in iDBP-DEP)
- Implementation is MATLAB-based (BAC.m/PYH_6.m with phy_6.mat), so a MATLAB runtime is needed unless re-implemented

## Evidence

- [intro] we combined them with PSSM-DWT (1040-dimension) and NMBAC (200-dimension) descriptor to obtain 3240-dimension feature vector: [evidence span withheld — anchor status: not_found]
