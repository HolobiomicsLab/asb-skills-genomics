---
name: confusion-matrix-metric-computation-with-mcc
description: Use when you have a trained binary classifier's predicted labels (e.g.
  LIBSVM -1/+1 outputs on PDB186) and must report independent-test Acc, Sn, Sp, MCC
  and AUC comparable to published DBP predictors.
when_to_use_negative:
- The goal is Jackknife (leave-one-out) evaluation on the benchmark PDB1075 or PDB594
  training datasets — this skill targets the PDB186 independent test only
- Input is raw protein sequences or PSSM profiles rather than predicted -1/+1 labels
  from an already-trained classifier; feature encoding and training must come first
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0082
- http://edamontology.org/topic_3474
tools:
- name: LIBSVM
  role: Implements the RBF-kernel SVM whose -1/+1 predictions on PDB186 provide the
    labels tallied into the confusion matrix (libsvm-3.21 shipped in the repo's Code/classifier)
  repo: https://githup.com/Zll-codeside/iDBP-DEP
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:2ad298834097bf7c8ae2882ec90770d1a10c13aac9bbd2c2e7c0cf8c59082cb7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LIBSVM
  - PSI-BLAST
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0082
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Confusion-matrix metric computation with MCC

## Summary

Computes Acc, Sn, Sp, MCC (from TP/TN/FP/FN via the paper's equations 16-19) plus ROC AUC for binary DNA-binding-protein (DBP vs non-DBP) predictions, using the SVM predicted -1/+1 labels from the iDBP-DEP pipeline.

## When to use

Use when you have a trained binary classifier's predicted labels (e.g. LIBSVM -1/+1 outputs on PDB186) and must report independent-test Acc, Sn, Sp, MCC and AUC comparable to published DBP predictors.

## When NOT to use

- The goal is Jackknife (leave-one-out) evaluation on the benchmark PDB1075 or PDB594 training datasets — this skill targets the PDB186 independent test only
- Input is raw protein sequences or PSSM profiles rather than predicted -1/+1 labels from an already-trained classifier; feature encoding and training must come first

## Inputs

- Predicted -1/+1 class labels for PDB186 test proteins from the trained LIBSVM RBF-kernel SVM model
- True labels for PDB186 (93 DBPs and 93 non-DBPs)
- SVM decision values for ROC/AUC computation
- Trained model from Independent_test.m / train_1075.m using PDB1075 top-424 features

## Outputs

- Confusion-matrix counts (TP, TN, FP, FN) on PDB186
- Acc, Sn, Sp, MCC values per equations 16-19 (target: Acc 80.1%, Sn 93.6%, Sp 66.7%, MCC 0.684)
- ROC curve and AUC value (target: 0.854)
- CSV comparison table against other predictors in Table 8

## How to apply

Run the trained SVM model (LIBSVM, RBF kernel, C=1, gamma=0.03125, trained on the PDB1075 top-424 feature vectors) over the held-out PDB186 feature vectors using the repository's Independent_test.m procedure, which yields -1/+1 predicted labels. Tally TP/TN/FP/FN against the true 93 DBP / 93 non-DBP labels and compute Acc, Sn, Sp and MCC according to equations 16-19 of the paper. Compute the ROC curve AUC from the decision values as an additional threshold-independent metric. Report all metrics in a CSV alongside the comparison predictors from Table 8 (Local-DPP, Wang and Ding's method, DBPPred, etc.). The reproduction is judged correct if metrics match the reported independent-test values (Acc 80.1%, Sn 93.6%, Sp 66.7%, AUC 0.854, MCC 0.684), the highest Acc among compared predictors.

## Related tools

- **LIBSVM** (Implements the RBF-kernel SVM whose -1/+1 predictions on PDB186 provide the labels tallied into the confusion matrix (libsvm-3.21 shipped in the repo's Code/classifier)) — https://githup.com/Zll-codeside/iDBP-DEP

## Examples

```
matlab -r "run('Code/predictor/Independent_test.m')" % predicts PDB186 labels via LIBSVM (C=1, gamma=0.03125) and computes Acc/Sn/Sp/MCC/AUC
```

## Evaluation signals

- Reproduced Acc on PDB186 should equal the reported 80.1% (highest among Table 8 predictors)
- Reproduced Sn 93.6%, Sp 66.7% and AUC 0.854 should match Table 8; Sn > Sp asymmetry (93 DBP vs 93 non-DBP) is expected
- MCC should equal 0.684; MCC must be consistent with the four TP/TN/FP/FN counts under equation 16-19 definitions
- TP + TN + FP + FN must sum to 186 (93 DBPs + 93 non-DBPs)

## Limitations

- Metrics are only directly comparable to Table 8 predictors if the identical PDB186 dataset and top-424 feature subset are used
- The reported Sn/Sp imbalance (93.6% vs 66.7%) means Acc and MCC can mask weaker non-DBP discrimination
- No version numbers stated for LIBSVM in the paper (the README ships libsvm-3.21), so minor numerical differences may arise with other versions

## Evidence
