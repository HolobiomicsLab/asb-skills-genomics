---
name: independent-dataset-validation-of-protein-classifier
description: Use when a protein classifier trained on a benchmark set (e.g. PDB1075)
  must be scored on a held-out independent set (e.g. PDB186) to report generalization
  metrics Acc, Sn, Sp, MCC, AUC against published baselines.
when_to_use_negative:
- Benchmarking on the training corpora PDB1075 or PDB594 themselves — the paper evaluates
  those with Jackknife and 10-fold cross-validation, not an independent split
- The test proteins have no precomputed feature .mat files (DDE186.mat, PSSM_DBT_186.mat,
  PSSM_DWT_186.mat, feature_NMBAC_186.mat) — PSSM generation via PSI-BLAST and feature
  extraction (DDE/DBT/DWT/NMBAC) is a separate upstream skill
- Choosing SVM hyperparameters C or gamma — these must be fixed from grid search on
  training data (C=1, gamma=0.03125); using PDB186 to tune them would invalidate the
  independent validation
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: LIBSVM
  role: Implements the RBF-kernel SVM trained on PDB1075 top-424 feature vectors (C=1,
    gamma=0.03125) and used to predict -1/+1 labels for PDB186; bundled as libsvm-3.21
    in the repo's Code/classifier
- name: iDBP-DEP
  role: Provides the MATLAB predictor scripts (train_1075.m, Independent_test.m) and
    the PDB1075/PDB186 .mat feature workspaces executed during the independent validation
  repo: https://githup.com/Zll-codeside/iDBP-DEP
- name: SVM-RFE+CBR
  role: Feature-selection algorithm whose top-424 feature indices define the reduced
    feature vector validated on PDB186; bundled as SVM-RFE-CBR-v1.3
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:3ab915b508e8fa712f4dae3f328f5d57c3590282c12e5973bc89ff94a48ee5d8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LIBSVM
  - PSI-BLAST
  - SVM-RFE+CBR
  - iDBP-DEP
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# independent-dataset-validation-of-protein-classifier

## Summary

Protocol for independent-dataset validation of the iDBP-DEP DNA-binding protein predictor: train an LIBSVM RBF-kernel SVM (C=1, gamma=0.03125) on PDB1075 top-424 features, score the untouched PDB186 set (93 DBPs / 93 non-DBPs), and report Acc, Sn, Sp, MCC and ROC-AUC against published predictors (Table 8).

## When to use

Use when a protein classifier trained on a benchmark set (e.g. PDB1075) must be scored on a held-out independent set (e.g. PDB186) to report generalization metrics Acc, Sn, Sp, MCC, AUC against published baselines.

## When NOT to use

- Benchmarking on the training corpora PDB1075 or PDB594 themselves — the paper evaluates those with Jackknife and 10-fold cross-validation, not an independent split
- The test proteins have no precomputed feature .mat files (DDE186.mat, PSSM_DBT_186.mat, PSSM_DWT_186.mat, feature_NMBAC_186.mat) — PSSM generation via PSI-BLAST and feature extraction (DDE/DBT/DWT/NMBAC) is a separate upstream skill
- Choosing SVM hyperparameters C or gamma — these must be fixed from grid search on training data (C=1, gamma=0.03125); using PDB186 to tune them would invalidate the independent validation

## Inputs

- PDB1075 training feature workspace files: DDE_workspace.mat, PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, feature_NMBAC_workspace.mat
- PDB186 independent-test feature files from ~/Data/PDB186/Feature Data: DDE186.mat, PSSM_DBT_186.mat, PSSM_DWT_186.mat, feature_NMBAC_186.mat
- Top-424 feature index vector selected by SVM-RFE+CBR from the original 3240-dimension feature space
- Trained LIBSVM model (RBF kernel, C=1, gamma=0.03125) produced by train_1075.m

## Outputs

- -1/+1 class predictions for the 93 DBPs / 93 non-DBPs in PDB186
- Independent-test metrics Acc, Sn, Sp, MCC computed from TP/TN/FP/FN (equations 16-19) plus the AUC of the ROC curve
- Comparison of PDB186 metrics against Table 8 predictors (Local-DPP, Wang and Ding's method, DBPPred, etc.) in a CSV output

## How to apply

Load the deposited PDB1075 workspace files (DDE_workspace.mat, PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, feature_NMBAC_workspace.mat) and the PDB186 feature files (DDE186.mat, PSSM_DBT_186.mat, PSSM_DWT_186.mat, feature_NMBAC_186.mat), then project both from the 3240-dimension DDE+PSSM-DBT+PSSM-DWT+NMBAC space onto the top-424 feature indices selected by SVM-RFE+CBR. Train an SVM with RBF kernel in LIBSVM (bundled as libsvm-3.21) on the PDB1075 top-424 vectors with the grid-searched parameters C=1 and gamma=0.03125, never tuning on PDB186 so the set stays independent (PSSM-DBT T=3 and NMBAC nlag=30 were likewise fixed by 10-fold cross-validation on training data). Reproduce the Independent_test.m procedure to predict -1/+1 labels for the 186 test vectors with the trained model. Compute Acc, Sn, Sp and MCC from TP/TN/FP/FN (equations 16-19) and the AUC of the ROC curve on PDB186. Judge success by matching the reported Table 8 independent-test performance (Acc 80.1%, Sn 93.6%, Sp 66.7%, AUC 0.854) and by ranking against the compared predictors (Local-DPP, Wang and Ding's method, DBPPred, etc.) in the output CSV.

## Related tools

- **LIBSVM** (Implements the RBF-kernel SVM trained on PDB1075 top-424 feature vectors (C=1, gamma=0.03125) and used to predict -1/+1 labels for PDB186; bundled as libsvm-3.21 in the repo's Code/classifier)
- **iDBP-DEP** (Provides the MATLAB predictor scripts (train_1075.m, Independent_test.m) and the PDB1075/PDB186 .mat feature workspaces executed during the independent validation) — https://githup.com/Zll-codeside/iDBP-DEP
- **SVM-RFE+CBR** (Feature-selection algorithm whose top-424 feature indices define the reduced feature vector validated on PDB186; bundled as SVM-RFE-CBR-v1.3)

## Examples

```
matlab -batch "run('Code/predictor/train_1075.m'); Independent_test"
```

## Evaluation signals

- Reproduced PDB186 metrics match the reported Table 8 independent-test values: Acc 80.1%, Sn 93.6%, Sp 66.7%, AUC 0.854 (with MCC 0.684 from the abstract)
- TP+TN+FP+FN sums to 186 proteins (93 DBPs, 93 non-DBPs); Acc 80.1% corresponds to ~149 correct predictions
- SVM input dimensionality is the top-424 selected feature indices, not the full 3240-dimension space — verify vector length before training/predicting
- SVM settings are exactly C=1 and gamma=0.03125 with the RBF kernel in LIBSVM; any hyperparameter tuning on PDB186 breaks the independent-validation claim
- Predictions are strictly -1/+1 labels from the trained model, and the AUC is computed from the ROC curve over the same PDB186 decision scores

## Limitations

- PDB186 is small (93 DBPs and 93 non-DBPs), so a few misclassifications shift Sn/Sp/MCC strongly; the reported Sp was only 66.7%
- No version numbers are stated for LIBSVM or PSI-BLAST, and no accession IDs or provenance are given for PDB1075, PDB594 or PDB186 — reproducibility depends entirely on the deposited .mat workspaces and scripts
- Faithful reproduction depends on the deposited SVM-RFE+CBR top-424 feature indices and PSI-BLAST-generated PSSM profiles; re-deriving features or the subset can move the metrics

## Evidence

- [methods] SVM hyperparameters: [evidence span withheld — anchor status: not_found]
