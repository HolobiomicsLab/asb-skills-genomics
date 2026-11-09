---
name: protein-multiview-feature-vector-concatenation
description: Use when encoding protein sequences for classification (e.g. DBP vs non-DBP)
  requires combining evolutionary-profile, dipeptide-composition and physicochemical-property
  views into one fixed-length vector for downstream feature selection.
when_to_use_negative:
- Input is already a reduced/optimal feature table from a prior feature-selection
  run — re-concatenating raw descriptor blocks would reintroduce the noisy and redundant
  features the selection was meant to remove
- No PSSM profiles are available (or no PSI-BLAST run against nr is feasible) — the
  PSSM-DBT and PSSM-DWT blocks cannot be computed, so the 3240-dim scheme does not
  apply
- The task requires interpreting individual descriptor importance rather than producing
  a single classification-ready matrix — per-descriptor analysis is a different sub-task
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3510
tools:
- name: PSI-BLAST
  role: Generates the PSSM/PSFM evolutionary profiles that the PSSM-DBT and PSSM-DWT
    descriptor blocks are computed from
- name: LIBSVM
  role: Implements the RBF-kernel SVM (C=1, gamma=0.03125) used both for SVM-RFE+CBR
    ranking and final classification of the concatenated vector; libsvm-3.21 ships
    in Code/classifier
  repo: https://githup.com/Zll-codeside/iDBP-DEP
- name: iFeature
  role: Cited as an example toolkit that facilitates feature generation; iDBP-DEP
    instead uses its own DDE/NMBAC/DBT/DWT scripts in Code/feature_Extract
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:5c35e5886ea8fc21ca9a8f75feb8a4d8dbcdcdd2b23c604d1a049b8f36feb005
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LIBSVM
  - PSI-BLAST
  - iFeature
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3510
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# protein-multiview-feature-vector-concatenation

## Summary

Concatenate heterogeneous protein descriptors — DDE (dipeptide deviation), PSSM-DBT (Distance-Bigram Transformation), PSSM-DWT (Discrete Wavelet Transformation) and NMBAC (Normalized Moreau-Broto Autocorrelation) — into a single 3240-dimension feature vector per sequence, which is then pruned by SVM-RFE+CBR feature selection before SVM classification.

## When to use

Use when encoding protein sequences for classification (e.g. DBP vs non-DBP) requires combining evolutionary-profile, dipeptide-composition and physicochemical-property views into one fixed-length vector for downstream feature selection.

## When NOT to use

- Input is already a reduced/optimal feature table from a prior feature-selection run — re-concatenating raw descriptor blocks would reintroduce the noisy and redundant features the selection was meant to remove
- No PSSM profiles are available (or no PSI-BLAST run against nr is feasible) — the PSSM-DBT and PSSM-DWT blocks cannot be computed, so the 3240-dim scheme does not apply
- The task requires interpreting individual descriptor importance rather than producing a single classification-ready matrix — per-descriptor analysis is a different sub-task

## Inputs

- Protein sequences in FASTA form (e.g. PDB1075: 525 DBPs + 550 non-DBPs)
- PSI-BLAST-generated PSSM profiles (PDB1075_seqs_PSSM) against the nr database
- Per-descriptor feature files: DDE.mat, PSSM_DBT.mat, PSSM_DWT.mat, feature_NMBAC.mat (MATLAB .mat)

## Outputs

- Concatenated 3240-dimension multi-view feature vector per sequence (e.g. DDE_DBT_DWT_NMBAC_594_workspace.mat)
- SVM-RFE+CBR ranked feature indices and optimal 424-dimension subset (per-descriptor composition: 51 DDE, 202 PSSM-DBT, 129 PSSM-DWT, 42 NMBAC)
- Trained SVM classifier (LIBSVM RBF, C=1, gamma=0.03125) ready for DBP prediction

## How to apply

Encode each protein sequence with four descriptors: DDE for dipeptide composition, PSSM-DBT on PSI-BLAST-generated PSSM profiles with parameter T=3, PSSM-DWT with 4-scale decomposition computing max/min/mean/standard-deviation statistics of approximation and detailed coefficients (1040 dimensions), and NMBAC with nlag=30 (200 dimensions). Concatenate all four descriptor blocks into a 3240-dimension vector; on the PDB1075 benchmark (525 DBPs, 550 non-DBPs) this yields the full training matrix whose parameters were fixed by 10-fold cross validation (T=3, nlag=30). Since the concatenated space is noisy and redundant, feed it to SVM-RFE+CBR (LIBSVM, RBF kernel, C=1, gamma=0.03125) to rank features and select the optimal subset — for PDB1075 the optimum was 424 dimensions (51 DDE, 202 PSSM-DBT, 129 PSSM-DWT, 42 NMBAC), reaching ~88.00% Jackknife accuracy (MCC 0.760). The repository provides pre-computed per-dataset .mat feature files (e.g. DDE.mat, PSSM_DBT.mat, PSSM_DWT.mat, feature_NMBAC.mat under Data/PDB1075/Feature Data) and PDB1075_workspace .mat files so the concatenation can be reconstructed without re-running PSSM generation.

## Related tools

- **PSI-BLAST** (Generates the PSSM/PSFM evolutionary profiles that the PSSM-DBT and PSSM-DWT descriptor blocks are computed from)
- **LIBSVM** (Implements the RBF-kernel SVM (C=1, gamma=0.03125) used both for SVM-RFE+CBR ranking and final classification of the concatenated vector; libsvm-3.21 ships in Code/classifier) — https://githup.com/Zll-codeside/iDBP-DEP
- **iFeature** (Cited as an example toolkit that facilitates feature generation; iDBP-DEP instead uses its own DDE/NMBAC/DBT/DWT scripts in Code/feature_Extract)

## Examples

```
matlab -batch "run('Code/predictor/train_1075_demo.txt')"  % loads DDE_DBT_DWT_NMBAC workspace .mat, trains LIBSVM C=1 gamma=0.03125
```

## Evaluation signals

- Concatenated vector dimensionality must equal 3240 (400 DDE + 2000 PSSM-DBT + 1040 PSSM-DWT + 200 NMBAC); any other length indicates a wrong descriptor parameter (T, nlag) or missing block
- After SVM-RFE+CBR on the full combination, the optimal subset size must be 424 dimensions with per-descriptor counts 51 DDE / 202 PSSM-DBT / 129 PSSM-DWT / 42 NMBAC
- Jackknife cross-validation accuracy on PDB1075 with the 424-feature subset should be at or near the reported 88.00% (MCC 0.760)
- Per-dataset workspace .mat files must load with matching sample counts: PDB1075 (525 DBPs + 550 non-DBPs), PDB594 (297+297), PDB186 (93+93)
- Descriptor parameters must match the 10-fold-CV choices: PSSM-DBT T=3 and NMBAC nlag=30

## Limitations

- The 3240-dim raw space is noisy and redundant — the concatenated vector alone performs worse than the SVM-RFE+CBR-pruned subset, so concatenation without feature selection is not the intended usage
- No version numbers are stated for PSI-BLAST (only LIBSVM 3.21 appears in the repo), so exact PSSM reproducibility depends on re-running PSI-BLAST with unspecified settings
- Optimal subset composition (424 features) was established for the PDB1075/PDB594/PDB186 benchmarks; other datasets require re-sweeping subset sizes (top 10 to top 3240) to find their own optimum

## Evidence

- [intro] we combined them with PSSM-DWT (1040-dimension) and NMBAC (200-dimension) descriptor to obtain 3240-dimension feature vector: [evidence span withheld — anchor status: not_found]
