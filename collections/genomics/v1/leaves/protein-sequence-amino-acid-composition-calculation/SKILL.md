---
name: protein-sequence-amino-acid-composition-calculation
description: Use when encoding raw protein sequences (e.g. from PDB1075/PDB594 FASTA
  sets) into physicochemical-property autocorrelation features plus amino acid composition
  for a DNA-binding-protein classifier.
when_to_use_negative:
- Input is already an extracted feature table or .mat feature matrix (e.g. feature_NMBAC_594.mat)
  — recomputing the descriptor would be redundant
- The target descriptor is PSSM-based (PSSM-DBT or PSSM-DWT) — those skills operate
  on PSI-BLAST PSSM profiles, not raw sequences and phy_6.mat properties
- Sequences contain non-standard or ambiguous amino acid residues — the descriptor
  standardizes only the 20 natural amino acids defined in phy_6.mat
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: iDBP-DEP NMBAC module
  role: MATLAB scripts BAC.m and PYH_6.m with phy_6.mat property data implement the
    NMBAC computation over protein sequences
  repo: https://githup.com/Zll-codeside/iDBP-DEP
- name: iFeature
  role: General-purpose toolkit for protein sequence feature generation cited alongside
    the paper's feature-extraction workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:61abeed7d6588df08bc83ae5c9f2a65c78018c2a1954b7fc75cd8b7812040358
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PSI-BLAST
  - iDBP-DEP NMBAC module
  - iFeature
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# protein-sequence-amino-acid-composition-calculation

## Summary

Computes the Normalized Moreau-Broto Autocorrelation (NMBAC) descriptor from protein sequences in the iDBP-DEP pipeline: 6 physicochemical properties autocorrelated at lags 1-30 plus amino acid frequencies, yielding a 200-dimension vector per protein.

## When to use

Use when encoding raw protein sequences (e.g. from PDB1075/PDB594 FASTA sets) into physicochemical-property autocorrelation features plus amino acid composition for a DNA-binding-protein classifier.

## When NOT to use

- Input is already an extracted feature table or .mat feature matrix (e.g. feature_NMBAC_594.mat) — recomputing the descriptor would be redundant
- The target descriptor is PSSM-based (PSSM-DBT or PSSM-DWT) — those skills operate on PSI-BLAST PSSM profiles, not raw sequences and phy_6.mat properties
- Sequences contain non-standard or ambiguous amino acid residues — the descriptor standardizes only the 20 natural amino acids defined in phy_6.mat

## Inputs

- Protein sequences in FASTA format from benchmark datasets (PDB1075, PDB594, or PDB186)
- Amino acid physicochemical property matrix phy_6.mat (6 properties x 20 standard amino acids, from the AAindex Database)

## Outputs

- 200-dimension NMBAC feature vector per protein (180 autocorrelation terms at nlag = 30 plus 20 amino acid frequencies)
- Per-dataset feature matrix saved as MATLAB .mat file (e.g. feature_NMBAC_workspace.mat, feature_NMBAC_594.mat, feature_NMBAC_186.mat)

## How to apply

Load protein sequences from the benchmark dataset (PDB1075, PDB594, or PDB186) and the six amino acid physicochemical properties (hydrophobicity, solvent accessibility, side-chain volume, polarity, polarizability, net charge index) supplied by phy_6.mat, sourced from the AAindex Database. Standardize each property across the 20 amino acids using its mean and standard deviation. Compute the normalized Moreau-Broto autocorrelation for each property at lags d = 1 to nlag = 30, giving 6 x 30 = 180 features per sequence. Append the frequency of each of the 20 natural amino acids to form the final 200-dimension vector per protein. Assemble per-protein vectors into a feature matrix and save it as feature_NMBAC_workspace.mat (or feature_NMBAC_186.mat / feature_NMBAC_594.mat), using the MATLAB scripts BAC.m and PYH_6.m in the NMBAC module. nlag = 30 was selected via 10-fold cross validation and is the value used by iDBP-DEP.

## Related tools

- **iDBP-DEP NMBAC module** (MATLAB scripts BAC.m and PYH_6.m with phy_6.mat property data implement the NMBAC computation over protein sequences) — https://githup.com/Zll-codeside/iDBP-DEP
- **iFeature** (General-purpose toolkit for protein sequence feature generation cited alongside the paper's feature-extraction workflow)

## Examples

```
matlab -r "run('Code/feature_Extract/NMBAC/PYH_6.m')"
```

## Evaluation signals

- Every protein yields exactly a 200-dimension vector: 180 autocorrelation terms (6 properties x nlag = 30) plus 20 amino acid frequencies
- nlag parameter equals 30, the value chosen by 10-fold cross validation in the paper
- Property values are standardized (mean 0, unit variance) across the 20 amino acids before autocorrelation is computed
- Output matrix filename matches the README convention (feature_NMBAC.mat for PDB1075, feature_NMBAC_594.mat, feature_NMBAC_186.mat)
- The NMBAC matrix concatenates with DDE (400-dim), PSSM-DBT and PSSM-DWT (1040-dim) to reproduce the 3240-dimension iDBP-DEP feature vector

## Limitations

- nlag = 30 is a dataset-specific hyperparameter tuned by 10-fold cross validation; other datasets may need re-tuning
- The descriptor depends on the six AAindex properties hardcoded in phy_6.mat; alternative property sets are not provided in the repo
- No accession IDs or provenance are given for the PDB1075/PDB594/PDB186 benchmark datasets, complicating exact reproduction

## Evidence

- [intro] NMBAC dimensionality within combined vector: [evidence span withheld — anchor status: not_found]
