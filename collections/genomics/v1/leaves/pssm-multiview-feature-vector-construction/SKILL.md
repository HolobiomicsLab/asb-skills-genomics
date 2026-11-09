---
name: pssm-multiview-feature-vector-construction
description: Use when encoding raw protein sequences (with precomputed PSSM profiles)
  into numeric feature vectors for a sequence-based DBP/non-DBP classifier, following
  the iDBP-DEP DDE+PSSM-DBT+PSSM-DWT+NMBAC recipe.
when_to_use_negative:
- Input is already an encoded feature table or workspace .mat file — skip straight
  to feature selection (SVM-RFE+CBR) and SVM training
- No PSSM profiles available and no PSI-BLAST/nr database access — DWT/DBT descriptors
  cannot be computed from raw sequences alone
- Task is residue-level (binding site) prediction rather than whole-protein DBP/non-DBP
  sequence classification
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0080
tools:
- name: PSI-BLAST
  role: Generates the PSSM/PSFM evolutionary profiles that are then encoded by the
    DBT and DWT descriptors
- name: LIBSVM
  role: Trains the RBF-kernel SVM (C=1, gamma=0.03125) on the selected features from
    the constructed vectors; bundled as libsvm-3.21
  repo: https://githup.com/Zll-codeside/iDBP-DEP
- name: iDBP-DEP
  role: Provides the feature extraction scripts (DDE.py, BAC.m, getDBT.m, getDWT.m)
    and per-dataset feature Data/workspaces
  repo: https://githup.com/Zll-codeside/iDBP-DEP
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:ee8c46b7a3891d0f17e26ebc1f3822d2153e968102fa8587313d9159b98a12f2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LIBSVM
  - PSI-BLAST
  - iDBP-DEP
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pssm-multiview-feature-vector-construction

## Summary

Constructs discriminative multi-view feature vectors for DNA-binding protein prediction by encoding PSI-BLAST PSSM profiles with Distance-Bigram Transformation (DBT) and Discrete Wavelet Transformation (DWT), plus DDE dipeptide and NMBAC physicochemical descriptors into a 3240-dimension feature space, as in iDBP-DEP.

## When to use

Use when encoding raw protein sequences (with precomputed PSSM profiles) into numeric feature vectors for a sequence-based DBP/non-DBP classifier, following the iDBP-DEP DDE+PSSM-DBT+PSSM-DWT+NMBAC recipe.

## When NOT to use

- Input is already an encoded feature table or workspace .mat file — skip straight to feature selection (SVM-RFE+CBR) and SVM training
- No PSSM profiles available and no PSI-BLAST/nr database access — DWT/DBT descriptors cannot be computed from raw sequences alone
- Task is residue-level (binding site) prediction rather than whole-protein DBP/non-DBP sequence classification

## Inputs

- Protein sequences in FASTA/text form for the DBP and non-DBP datasets (PDB1075, PDB594, PDB186)
- PSI-BLAST PSSM profiles per sequence (L×20 matrices, as in the PDB*_seqs_PSSM folders)
- Amino acid physicochemical property table (phy_6.mat) for NMBAC

## Outputs

- Per-dataset descriptor matrices as .mat files: DDE_*.mat, PSSM_DBT_*.mat, PSSM_DWT_*.mat, feature_NMBAC_*.mat
- Concatenated 3240-dimension feature workspace (e.g. DDE_DBT_DWT_NMBAC_*_workspace.mat)
- Post-selection top-424 feature index/vector for SVM training

## How to apply

Generate PSSM profiles for each target protein by searching the non-redundant (nr) database with PSI-BLAST. From each sequence compute four descriptors: DDE dipeptide composition, PSSM-DBT with parameter T=3, PSSM-DWT with 4 rounds of decomposition (recording maximum, minimum, mean and standard deviation of approximation and detail coefficients at each level), and NMBAC with nlag=30; concatenate DDE (80), PSSM-DBT (1920), PSSM-DWT (1040) and NMBAC (200) into a 3240-dimension vector per protein. The repo's Code/feature_Extract directory holds the extractors (DDE.py in Python, BAC.m/DBT/DWT scripts in MATLAB, phy_6.mat physicochemical property table). Store per-dataset feature matrices as .mat workspace files (e.g. DDE_workspace.mat, PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, feature_NMBAC_workspace.mat). Downstream, apply SVM-RFE+CBR feature selection (top 424 indices) before LIBSVM RBF-kernel classification, so encoding must preserve column identity across the four descriptor blocks.

## Related tools

- **PSI-BLAST** (Generates the PSSM/PSFM evolutionary profiles that are then encoded by the DBT and DWT descriptors)
- **LIBSVM** (Trains the RBF-kernel SVM (C=1, gamma=0.03125) on the selected features from the constructed vectors; bundled as libsvm-3.21) — https://githup.com/Zll-codeside/iDBP-DEP
- **iDBP-DEP** (Provides the feature extraction scripts (DDE.py, BAC.m, getDBT.m, getDWT.m) and per-dataset feature Data/workspaces) — https://githup.com/Zll-codeside/iDBP-DEP

## Examples

```
python Code/feature_Extract/DDE/DDE.py  # then in MATLAB: getDBT.m, getDWT.m, BAC.m; load PDB1075_workspace/PSSM_DBT_workspace.mat and run Code/predictor/train_1075.m
```

## Evaluation signals

- Descriptor dimensions check: DDE=80, PSSM-DBT=1920, PSSM-DWT=1040, NMBAC=200, concatenated total = 3240 per protein
- After SVM-RFE+CBR selection, the top-424 feature indices are used; independent-test SVM on PDB186 with these features should reach Acc 80.1%, Sn 93.6%, Sp 66.7%, MCC 0.684, AUC 0.854
- Feature matrices written as MATLAB .mat workspaces (DDE_workspace.mat, PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, feature_NMBAC_workspace.mat) matching the repo layout
- Cross-validated parameter choices reproduce: T=3 (PSSM-DBT) and nlag=30 (NMBAC) from 10-fold CV on PDB1075

## Limitations

- No version numbers are stated for PSI-BLAST or LIBSVM beyond the bundled libsvm-3.21; downstream reproducibility may vary
- Encoding depends on PSI-BLAST against nr, so PSSMs are database- and run-dependent; no accession/provenance IDs are given for PDB1075, PDB594, PDB186
- The 424-feature index was optimized on these specific benchmark datasets and may not transfer to other DBP corpora

## Evidence

- [intro] combined descriptors: [evidence span withheld — anchor status: not_found]
- [methods] DDE descriptor: [evidence span withheld — anchor status: not_found]
