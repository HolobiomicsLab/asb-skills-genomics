---
name: roc-auc-performance-comparison-across-predictors
description: Use when a held-out independent test set (e.g. PDB186, 93 DBPs + 93 non-DBPs)
  must be scored by a pre-trained SVM and its Acc/Sn/Sp/MCC/AUC benchmarked against
  published predictor comparisons.
when_to_use_negative:
- The goal is cross-validated performance on the training benchmarks (Jackknife test
  on PDB1075/PDB594) rather than an independent test — use the train_1075 workflow
  instead of Independent_test.m
- Input sequences lack precomputed PSSM profiles or feature workspace .mat files,
  so the 3240-dimension multi-view feature space (DDE, PSSM-DBT with T=3, PSSM-DWT,
  NMBAC with nlag=30) cannot be assembled
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: LIBSVM
  role: Implements the RBF-kernel SVM classifier used to fit PDB1075 top-424 feature
    vectors and predict PDB186 labels (C=1, gamma=0.03125); repo bundles libsvm-3.21
  repo: https://githup.com/Zll-codeside/iDBP-DEP
- name: PSI-BLAST
  role: Generates the PSSM profiles from the nr database that underlie the PSSM-DBT
    and PSSM-DWT features of the workspace files
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:57da70fcb473680dc5d138d0cf229f0f2acf7b0a908314a2f4e1e6ba80ffc397
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LIBSVM
  - PSI-BLAST
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# roc-auc-performance-comparison-across-predictors

## Summary

Reproduces an independent-validation benchmark of iDBP-DEP on PDB186: train the RBF-kernel SVM (C=1, gamma=0.03125) on PDB1075 top-424 feature vectors, predict PDB186 labels, compute Acc/Sn/Sp/MCC/AUC, and compare against alternative predictors (Local-DPP, DBPPred, etc.) as in Table 8.

## When to use

Use when a held-out independent test set (e.g. PDB186, 93 DBPs + 93 non-DBPs) must be scored by a pre-trained SVM and its Acc/Sn/Sp/MCC/AUC benchmarked against published predictor comparisons.

## When NOT to use

- The goal is cross-validated performance on the training benchmarks (Jackknife test on PDB1075/PDB594) rather than an independent test — use the train_1075 workflow instead of Independent_test.m
- Input sequences lack precomputed PSSM profiles or feature workspace .mat files, so the 3240-dimension multi-view feature space (DDE, PSSM-DBT with T=3, PSSM-DWT, NMBAC with nlag=30) cannot be assembled

## Inputs

- PDB1075 feature workspace files (DDE_workspace.mat, PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, feature_NMBAC_workspace.mat)
- PDB186 feature workspace files (DDE186.mat, feature_NMBAC_186.mat, PSSM_DBT_186.mat, PSSM_DWT_186.mat)
- Top-424 feature index from SVM-RFE+CBR selection over the 3240-dimension feature vector
- LIBSVM training demo (train_1075_demo.txt) and Independent_test.m from the repo's Code/predictor directory

## Outputs

- Predicted -1/+1 DBP labels for the 186 PDB186 sequences
- Confusion-matrix counts (TP/TN/FP/FN) and derived Acc, Sn, Sp, MCC metrics
- ROC curve with AUC value
- Comparison CSV row of iDBP-DEP metrics versus Table 8 predictors (expected: Acc 80.1%, Sn 93.6%, Sp 66.7%, AUC 0.854, MCC 0.684)

## How to apply

Load the deposited PDB1075 and PDB186 workspace .mat files (DDE_workspace.mat, PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, feature_NMBAC_workspace.mat) and compose the top-424 feature index selected by SVM-RFE+CBR from the original 3240-dimension feature space. Train the LIBSVM classifier with RBF kernel using the paper's grid-search parameters C=1 and gamma=0.03125 on the PDB1075 training vectors, then run the Independent_test.m procedure to predict -1/+1 labels for PDB186 test vectors. Compute TP/TN/FP/FN-based Acc, Sn, Sp and MCC (equations 16-19) plus the ROC curve AUC on PDB186. Write the metrics to a CSV alongside the other predictors of Table 8 (Local-DPP, Wang and Ding's method, DBPPred) and check reproduction against the reported Acc 80.1%, Sn 93.6%, Sp 66.7%, AUC 0.854 and MCC 0.684.

## Related tools

- **LIBSVM** (Implements the RBF-kernel SVM classifier used to fit PDB1075 top-424 feature vectors and predict PDB186 labels (C=1, gamma=0.03125); repo bundles libsvm-3.21) — https://githup.com/Zll-codeside/iDBP-DEP
- **PSI-BLAST** (Generates the PSSM profiles from the nr database that underlie the PSSM-DBT and PSSM-DWT features of the workspace files)

## Examples

```
matlab -run "load PDB1075_workspace/DDE_workspace.mat; model = svmtrain(label_1075, feat_1075(:, top424), '-t 2 -c 1 -g 0.03125'); [pred, acc, dec] = svmpredict(label_186, feat_186(:, top424), model);"
```

## Evaluation signals

- Reproduced independent-test metrics match Table 8: Acc 80.1%, Sn 93.6%, Sp 66.7%, AUC 0.854, MCC 0.684
- MCC computed from TP/TN/FP/FN via equations 16-19 equals ~0.684 on the 93 DBP / 93 non-DBP PDB186 set
- Prediction vector contains only -1/+1 labels and covers all 186 PDB186 test samples
- Feature vectors fed to the SVM have exactly the top-424 selected dimensions, not the full 3240
- iDBP-DEP's Acc is the highest among the compared predictors (Local-DPP, Wang and Ding's method, DBPPred) in the output CSV

## Limitations

- No version numbers stated for LIBSVM or PSI-BLAST in the article (README shows libsvm-3.21 bundled in the repo)
- No accession IDs or provenance given for the benchmark datasets PDB1075, PDB594, PDB186
- Independent-test Sn/Sp are imbalanced (93.6% vs 66.7%), reflecting the composition and difficulty of PDB186

## Evidence

- [methods] SVM parameters were set by grid search: C=1 and gamma=0.03125.: [evidence span withheld — anchor status: not_found]
