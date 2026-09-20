---
name: svm-rbf-classifier-training
description: Use when training a binary DBP/non-DBP classifier on precomputed .mat
  feature workspaces (PDB1075, PDB594, PDB186) with LIBSVM RBF kernel and fixed hyperparameters
  C=1, gamma=0.03125.
when_to_use_negative:
- Input is raw protein sequences or PSSM profiles rather than precomputed numerical
  feature workspaces — use the feature-extraction skill (DDE, PSSM-DBT, PSSM-DWT,
  NMBAC) first
- The task requires hyperparameter tuning — this skill fixes C=1 and gamma=0.03125
  from the paper's grid search rather than re-optimizing
- Feature selection is the goal — use the SVM-RFE+CBR feature-ranking procedure, which
  iteratively removes lowest-weight features, instead of direct classifier training
edam_operation: http://edamontology.org/operation_3094
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0749
tools:
- name: LIBSVM
  role: Implements the SVM model with the radial basis function kernel used for DBP
    detection and Jackknife evaluation
  repo: https://githup.com/Zll-codeside/iDBP-DEP
- name: SVM-RFE-CBR-v1.3
  role: Bundled feature-ranking algorithm that produces the optimal top-424 feature
    subset fed into the SVM classifier
  repo: https://githup.com/Zll-codeside/iDBP-DEP
- name: PSI-BLAST
  role: Generates the PSSM profiles underlying the PSSM-DBT and PSSM-DWT features
    used as classifier input
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:4e3ee88faf409253a1b52b1283b585255dd28ba789e5bf509e3217cc2611e6e4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LIBSVM
  - PSI-BLAST
  - SVM-RFE-CBR-v1.3
  edam_operation: http://edamontology.org/operation_3094
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# svm-rbf-classifier-training

## Summary

Train and evaluate an LIBSVM radial-basis-function SVM for DNA-binding protein prediction on the iDBP-DEP multi-view feature workspaces (3240-dim DDE+PSSM-DBT+PSSM-DWT+NMBAC vectors or SVM-RFE+CBR-selected subsets), using the paper's grid-search-optimized parameters C=1 and gamma=0.03125.

## When to use

Use when training a binary DBP/non-DBP classifier on precomputed .mat feature workspaces (PDB1075, PDB594, PDB186) with LIBSVM RBF kernel and fixed hyperparameters C=1, gamma=0.03125.

## When NOT to use

- Input is raw protein sequences or PSSM profiles rather than precomputed numerical feature workspaces — use the feature-extraction skill (DDE, PSSM-DBT, PSSM-DWT, NMBAC) first
- The task requires hyperparameter tuning — this skill fixes C=1 and gamma=0.03125 from the paper's grid search rather than re-optimizing
- Feature selection is the goal — use the SVM-RFE+CBR feature-ranking procedure, which iteratively removes lowest-weight features, instead of direct classifier training

## Inputs

- Feature workspace .mat files (DDE_DBT_DWT_NMBAC_594_workspace.mat, DDE_workspace.mat, PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, feature_NMBAC_workspace.mat) with labels for PDB1075/PDB594
- Per-descriptor .mat feature files (DDE.mat, DDE594.mat, PSSM_DBT.mat, PSSM_DWT.mat, feature_NMBAC.mat) from ~/Data/*/Feature Data
- Top-424 SVM-RFE+CBR feature index subset over the original 3240-dimension feature space
- PDB186 test feature vectors for independent validation

## Outputs

- Trained LIBSVM RBF-kernel model on the selected feature subset
- Predicted -1/+1 DBP/non-DBP labels for test samples (e.g. PDB186 via Independent_test.m)
- Jackknife and independent-test metrics: Sn, Sp, Acc, MCC, AUC

## How to apply

Load the deposited feature workspace .mat files (e.g. DDE_DBT_DWT_NMBAC workspace for the full 3240-dimension feature space, or per-descriptor DDE/PSSM-DBT/PSSM-DWT/NMBAC workspaces, or the top-424 SVM-RFE+CBR-selected feature indices) for PDB1075, PDB594 or PDB186. Implement the SVM with the LIBSVM package (libsvm-3.21 bundled in the repository) using the radial basis function kernel with C=1 and gamma=0.03125, the values obtained by grid search and used throughout the study. On benchmark datasets, evaluate with the rigorous Jackknife (leave-one-out) test; for the independent validation, train on PDB1075 top-424 features and predict PDB186 labels following Independent_test.m. Compute Sn, Sp, Acc, MCC (equations 16-19) and AUC on the resulting predictions.

## Related tools

- **LIBSVM** (Implements the SVM model with the radial basis function kernel used for DBP detection and Jackknife evaluation) — https://githup.com/Zll-codeside/iDBP-DEP
- **SVM-RFE-CBR-v1.3** (Bundled feature-ranking algorithm that produces the optimal top-424 feature subset fed into the SVM classifier) — https://githup.com/Zll-codeside/iDBP-DEP
- **PSI-BLAST** (Generates the PSSM profiles underlying the PSSM-DBT and PSSM-DWT features used as classifier input)

## Examples

```
run Independent_test.m / train_1075.m in MATLAB with LIBSVM: svmtrain(label, feat(:, top424), '-t 2 -c 1 -g 0.03125')
```

## Evaluation signals

- Pre-selection Jackknife baselines must reproduce Acc 80.84% / MCC 0.617 on PDB1075 and Acc 78.45% / MCC 0.569 on PDB594 with all 3240-dim features
- Post-selection (top-424 features) Jackknife metrics must match Acc 88.00% / MCC 0.760 / AUC 0.943 on PDB1075 and Acc 93.60% / MCC 0.872 / AUC 0.973 on PDB594
- Independent test on PDB186 must yield Acc 80.1%, Sn 93.6%, Sp 66.7%, MCC 0.684, AUC 0.854
- SVM hyperparameters must be exactly C=1 and gamma=0.03125, matching the grid-search values reported in the methods
- MCC computed from TP/TN/FP/FN must follow equations 16-19 and stay consistent with the reported Sn/Sp/Acc triple

## Limitations

- No LIBSVM version is stated in the article text; only the bundled libsvm-3.21 directory in the repository indicates the version used
- Results depend on PSSM profiles generated by PSI-BLAST against the non-redundant (nr) database, so raw-sequence inputs require that costly prior step
- The fixed hyperparameters (C=1, gamma=0.03125) and PSSM-DBT T=3 / NMBAC nlag=30 were tuned for these specific datasets and may not transfer to others

## Evidence

- [methods] According to the experiment search result, C was finally set as 1 and was set as 0.03125.: [evidence span withheld — anchor status: not_found]
