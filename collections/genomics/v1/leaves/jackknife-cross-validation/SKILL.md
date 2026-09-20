---
name: jackknife-cross-validation
description: Use when benchmarking a sequence-based protein classifier on PDB1075,
  PDB594 or PDB186 with the most rigorous resampling scheme, or when reporting Sn/Sp/Acc/MCC/AUC
  comparable to DBP predictors.
when_to_use_negative:
- Evaluation requires a fast, low-variance estimate — Jackknife (leave-one-out) retraining
  on 1075 samples is computationally expensive; use k-fold cross-validation instead
  for quick iterations
- You need a truly independent test estimate — use the independent PDB186 validation
  set (Independent_test.m) rather than resampling PDB1075/PDB594
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: LIBSVM
  role: SVM implementation with radial basis function kernel used to train each leave-one-out
    model with C=1 and gamma=0.03125
- name: MATLAB
  role: Environment running the predictor scripts (before_FS.m, train_1075.m, train_594_demo.txt)
    that execute the Jackknife evaluation loops
  repo: https://githup.com/Zll-codeside/iDBP-DEP
provenance:
  source_task_ids:
  - task_007
  - task_005
  - task_006
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:1248653e8da6a1265012c0a7e9b5cb039813f8457b5df452d3781edb64f4d589
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LIBSVM
  - MATLAB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# jackknife-cross-validation

## Summary

Applies Jackknife (leave-one-out) cross-validation to evaluate the iDBP-DEP DNA-binding protein predictor, training a LIBSVM RBF-kernel classifier (C=1, gamma=0.03125) on PDB1075/PDB594 feature workspaces and computing Sn, Sp, Acc, MCC and AUC per the paper's formulas 16-19.

## When to use

Use when benchmarking a sequence-based protein classifier on PDB1075, PDB594 or PDB186 with the most rigorous resampling scheme, or when reporting Sn/Sp/Acc/MCC/AUC comparable to state-of-the-art DBP predictors.

## When NOT to use

- Evaluation requires a fast, low-variance estimate — Jackknife (leave-one-out) retraining on 1075 samples is computationally expensive; use k-fold cross-validation instead for quick iterations
- You need a truly independent test estimate — use the independent PDB186 validation set (Independent_test.m) rather than resampling PDB1075/PDB594

## Inputs

- .mat feature workspace files (DDE_workspace.mat, PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, feature_NMBAC_workspace.mat, DDE_DBT_DWT_NMBAC_594_workspace.mat) from the iDBP-DEP repository
- Benchmark dataset feature matrices with labels: PDB1075 (525 DBPs, 550 non-DBPs) and PDB594 (297 DBPs, 297 non-DBPs)
- Selected feature indices (e.g. top 424 ranked features from SVM-RFE+CBR)

## Outputs

- Per-dataset Jackknife metrics: Sensitivity (Sn), Specificity (Sp), Accuracy (Acc), MCC, AUC computed per formulas 16-19
- Comparison table of pre-selection (3240-dim) vs post-selection (424-dim) Jackknife performance
- Comparison against state-of-the-art sequence-based DBP predictors

## How to apply

Load the deposited .mat feature workspaces (per-descriptor DDE, PSSM_DBT, PSSM_DWT, NMBAC files or the combined DDE_DBT_DWT_NMBAC workspace, e.g. for PDB1075's 525 DBPs / 550 non-DBPs). For each protein, hold it out as the test instance, train the LIBSVM SVM with RBF kernel on the remaining samples using grid-search-optimized parameters C=1 and gamma=0.03125, and predict the held-out protein; repeat until every protein has been tested exactly once. Aggregate predictions to compute Sn, Sp, Acc, MCC and AUC using the paper's formulas 16-19, running this both on the full 3240-dimension feature vector (baseline: Acc 80.84%, MCC 0.617 on PDB1075) and on the SVM-RFE+CBR-selected 424-feature subset (Acc 88.00%, MCC 0.760, AUC 0.943 on PDB1075). Compare against the reported state-of-the-art gaps (1.8% Acc / 3.0% MCC on PDB1075; 7.4% Acc / 14.8% MCC on PDB594) to validate the run.

## Related tools

- **LIBSVM** (SVM implementation with radial basis function kernel used to train each leave-one-out model with C=1 and gamma=0.03125)
- **MATLAB** (Environment running the predictor scripts (before_FS.m, train_1075.m, train_594_demo.txt) that execute the Jackknife evaluation loops) — https://githup.com/Zll-codeside/iDBP-DEP

## Examples

```
matlab -batch "run('Code/predictor/train_1075.m')"  % Jackknife LOO training on PDB1075 with libsvm-3.21, C=1, gamma=0.03125
```

## Evaluation signals

- Jackknife accuracy on the full 3240-dimension feature space of PDB1075 should reproduce the pre-selection baseline Acc 80.84%, MCC 0.617 (PDB594: Acc 78.45%, MCC 0.569)
- After selecting the SVM-RFE+CBR top-424 features, Jackknife metrics should match Acc 88.00%, MCC 0.760, AUC 0.943 on PDB1075 and Acc 93.60%, MCC 0.872, AUC 0.973 on PDB594
- Sample counts must be consistent: 1075 proteins for PDB1075 (525 DBPs / 550 non-DBPs) and 594 for PDB594 (297/297), each tested exactly once per fold of the leave-one-out loop
- Improvement over state-of-the-art should fall near 1.8% Acc / 3.0% MCC on PDB1075 and 7.4% Acc / 14.8% MCC on PDB594
- Independent-set check: PDB186 run should yield Acc 80.1% and MCC 0.684

## Limitations

- Jackknife is computationally heavy: n SVM trainings (n = 1075 for PDB1075), each on up to 3240 features
- No accession IDs or provenance are given for PDB1075, PDB594, PDB186, limiting provenance verification of the benchmark inputs
- No LIBSVM version number is stated in the article, though the repository ships libsvm-3.21

## Evidence

- [methods] SVM parameter setting: [evidence span withheld — anchor status: not_found]
