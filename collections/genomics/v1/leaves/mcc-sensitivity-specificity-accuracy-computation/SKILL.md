---
name: mcc-sensitivity-specificity-accuracy-computation
description: Use when evaluating a binary DBP/non-DBP classifier on benchmark sets
  like PDB1075 or PDB594 and you need Sn, Sp, Acc, MCC and AUC comparable to published
  Jackknife-test values.
when_to_use_negative:
- The data is a multi-class or regression problem — the MCC/Sn/Sp formulas here are
  defined for the two-class DBP vs non-DBP setting only
- The task is generating the features themselves (DDE, PSSM-DBT, PSSM-DWT, NMBAC)
  or running SVM-RFE+CBR feature ranking — this skill only consumes classifier outputs
- Evaluation requires a relaxed cross-validation protocol; the reported values are
  only comparable under the rigorous Jackknife test
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3474
tools:
- name: LIBSVM
  role: Produces the Jackknife cross-validated predictions (RBF kernel, C=1, gamma=0.03125)
    whose labels and scores feed the Sn/Sp/Acc/MCC/AUC computations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:ba2c5255305cb0836a0eb4e09a2f91a4c16ba86d9b476bd0d02b556fd56d4335
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LIBSVM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# MCC sensitivity specificity accuracy computation

## Summary

Computes the four standard binary-classification metrics — Sensitivity (Sn), Specificity (Sp), Accuracy (Acc) and Matthew's Correlation Coefficient (MCC), plus AUC — used to score sequence-based DNA-binding protein predictors under the rigorous Jackknife test.

## When to use

Use when evaluating a binary DBP/non-DBP classifier on benchmark sets like PDB1075 or PDB594 and you need Sn, Sp, Acc, MCC and AUC comparable to published Jackknife-test values.

## When NOT to use

- The data is a multi-class or regression problem — the MCC/Sn/Sp formulas here are defined for the two-class DBP vs non-DBP setting only
- The task is generating the features themselves (DDE, PSSM-DBT, PSSM-DWT, NMBAC) or running SVM-RFE+CBR feature ranking — this skill only consumes classifier outputs
- Evaluation requires a relaxed cross-validation protocol; the reported values are only comparable under the rigorous Jackknife test

## Inputs

- Predicted and true labels (or confusion-matrix counts TP, TN, FP, FN) from a Jackknife cross-validated LIBSVM RBF-kernel run on PDB1075 (525 DBPs / 550 non-DBPs) or PDB594 (297/297)
- Continuous decision scores from the same model for AUC computation
- Labelled feature vectors from the deposited workspace .mat files (e.g. DDE_DBT_DWT_NMBAC_594_workspace.mat, DDE_DBT_DWT_NMBAC workspace for PDB1075)

## Outputs

- Jackknife-test Sn, Sp, Acc and MCC for the all-features baseline (expected ~Acc 80.84%, MCC 0.617 on PDB1075; Acc 78.45%, MCC 0.569 on PDB594)
- Jackknife-test Sn, Sp, Acc, MCC and AUC for the feature-selected model (expected Acc 88.00%, MCC 0.760, AUC 0.943 on PDB1075; Acc 93.60%, MCC 0.872, AUC 0.973 on PDB594)
- Independent-test metrics for PDB186 (expected Acc 80.1%, MCC 0.684)

## How to apply

Train the LIBSVM RBF-kernel classifier (C=1, gamma=0.03125) on the 3240-dim DDE+PSSM-DBT+PSSM-DWT+NMBAC feature vectors loaded from the deposited workspace .mat files, and predict labels under Jackknife cross-validation so every sample is held out exactly once. From the resulting confusion-matrix counts (TP, TN, FP, FN) compute Sn = TP/(TP+FN), Sp = TN/(TN+FP), Acc = (TP+TN)/(TP+TN+FP+FN), and MCC = (TP*TN - FP*FN)/sqrt((TP+FP)(TP+FN)(TN+FP)(TN+FN)); collect predicted scores to draw an ROC curve for AUC. Run the metric computation both on the all-features model (pre-selection baseline) and on the model retrained with the SVM-RFE+CBR-selected top feature subset (top 424 features on PDB1075) so improvement from feature selection can be quantified. The article also reports the same metrics for an independent test on PDB186 (93 DBPs vs 93 non-DBPs), where the same formulas apply.

## Related tools

- **LIBSVM** (Produces the Jackknife cross-validated predictions (RBF kernel, C=1, gamma=0.03125) whose labels and scores feed the Sn/Sp/Acc/MCC/AUC computations)

## Examples

```
matlab -batch "run('Code/predictor/train_1075.m')"  % Jackknife-train LIBSVM (C=1, gamma=0.03125) on PDB1075 workspace, then compute Sn/Sp/Acc/MCC
```

## Evaluation signals

- Pre-selection Jackknife baseline must match the published values within a small tolerance: PDB1075 Acc 80.84%, MCC 0.617; PDB594 Acc 78.45%, MCC 0.569
- Post-feature-selection Jackknife metrics must match: PDB1075 Acc 88.00%, MCC 0.760, AUC 0.943; PDB594 Acc 93.60%, MCC 0.872, AUC 0.973
- MCC must fall in the range (-1, 1] and be roughly consistent with Acc: on balanced PDB594 (297/297), MCC > 0 implies Acc > 50%
- Sn and Sp must both lie in [0, 1], and TP + FN + TN + FP must equal the dataset size (1075 for PDB1075, 594 for PDB594, 186 for PDB186)
- Independent-test metrics on PDB186 must be reproducible at Acc 80.1% and MCC 0.684

## Limitations

- Metric values are only comparable when the same protocol (Jackknife test) and the same classifier settings (LIBSVM RBF, C=1, gamma=0.03125) are used; other CV schemes or hyperparameters will not reproduce the numbers
- The paper's repository does not state LIBSVM or PSI-BLAST version numbers, so exact numerical reproduction may vary slightly across library versions
- MCC is sensitive to class imbalance; PDB1075 (525 vs 550) is slightly imbalanced whereas PDB594 and PDB186 are exactly balanced (297/297 and 93/93)

## Evidence
