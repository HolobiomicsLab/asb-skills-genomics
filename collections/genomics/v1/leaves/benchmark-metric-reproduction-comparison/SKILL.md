---
name: benchmark-metric-reproduction-comparison
description: Use when a paper's deposited feature matrices (e.g. .mat workspaces)
  plus classifier settings are available and the goal is to numerically reproduce
  reported benchmark metrics (Jackknife Acc, MCC, AUC) within a small tolerance.
when_to_use_negative:
- You need de novo prediction on new protein sequences rather than metric reproduction
  (use the deposited predictors/Independent_test.m path instead of the benchmark-reproduction
  workflow)
- Input is raw protein sequences without precomputed PSSM profiles — the deposited
  workspaces contain already-extracted 3240-dim features; PSI-BLAST runs against nr
  are a prerequisite for re-extraction
- You only need feature generation (DDE, PSSM-DBT, PSSM-DWT, NMBAC) and not comparison
  against published benchmark numbers
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: LIBSVM
  role: RBF-kernel SVM classifier (C=1, gamma=0.03125) trained on the 3240-dim feature
    space and evaluated with Jackknife test; pinned as libsvm-3.21 in the repo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:6cdd9e654ec212e70803e35a64be8643671201136024679b26b255a8bdb05e81
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LIBSVM
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# benchmark-metric-reproduction-comparison

## Summary

Reproduces a paper's published benchmark metrics (Sn, Sp, Acc, MCC, AUC) by retraining the deposited model on the authors' deposited feature data and comparing numerically to the reported values. Here it validates iDBP-DEP's Jackknife-test results on the PDB1075 and PDB594 DNA-binding protein benchmarks.

## When to use

Use when a paper's deposited feature matrices (e.g. .mat workspaces) plus classifier settings are available and the goal is to numerically reproduce reported benchmark metrics (Jackknife Acc, MCC, AUC) within a small tolerance.

## When NOT to use

- You need de novo prediction on new protein sequences rather than metric reproduction (use the deposited predictors/Independent_test.m path instead of the benchmark-reproduction workflow)
- Input is raw protein sequences without precomputed PSSM profiles — the deposited workspaces contain already-extracted 3240-dim features; PSI-BLAST runs against nr are a prerequisite for re-extraction
- You only need feature generation (DDE, PSSM-DBT, PSSM-DWT, NMBAC) and not comparison against published benchmark numbers

## Inputs

- Deposited feature workspace .mat files (DDE_DBT_DWT_NMBAC_594_workspace.mat, PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, DDE_workspace.mat, feature_NMBAC_workspace.mat) with labels
- Feature Data .mat files under ~/Data/PDB1075, ~/Data/PDB594, ~/Data/PDB186 (DDE*.mat, PSSM_DBT*.mat, PSSM_DWT*.mat, feature_NMBAC*.mat)
- Published reference metric values for comparison (Jackknife Sn, Sp, Acc, MCC, AUC; pre- and post-feature-selection)
- Classifier and feature-selection settings from the paper (LIBSVM RBF kernel, C=1, gamma=0.03125; SVM-RFE+CBR v1.3)

## Outputs

- Reproduced Jackknife metric table (Sn, Sp, Acc, MCC, AUC) for the all-features baseline model per dataset
- Reproduced Jackknife metric table for the SVM-RFE+CBR selected-feature model (e.g. top 424 features on PDB1075)
- Comparison/diff against published values (expected PDB1075: Acc 88.00%, MCC 0.760, AUC 0.943; PDB594: Acc 93.60%, MCC 0.872, AUC 0.973)

## How to apply

Load the deposited feature workspaces (e.g. DDE_DBT_DWT_NMBAC_594_workspace.mat and PDB1075 workspace files containing 3240-dimension DDE+PSSM-DBT+PSSM-DWT+NMBAC vectors with labels). Train the LIBSVM RBF-kernel classifier with the paper's tuned hyperparameters (C=1, gamma=0.03125) on the full 3240-dim feature space and evaluate with Jackknife cross-validation to reproduce pre-selection baselines (PDB1075: Acc 80.84%, MCC 0.617; PDB594: Acc 78.45%, MCC 0.569). Next run the SVM-RFE+CBR feature-ranking algorithm, iteratively removing lowest-weight features and recording SVM accuracy, then retrain on the optimal top-ranked subset (top 424 features on PDB1075). Compute Sn, Sp, Acc, MCC and AUC on the selected-feature model and compare against the reported post-selection values (PDB1075: Acc 88.00%, MCC 0.760, AUC 0.943; PDB594: Acc 93.60%, MCC 0.872, AUC 0.973). A reproduction is correct when both baseline and post-selection metrics match the published values within a small tolerance.

## Related tools

- **LIBSVM** (RBF-kernel SVM classifier (C=1, gamma=0.03125) trained on the 3240-dim feature space and evaluated with Jackknife test; pinned as libsvm-3.21 in the repo)

## Examples

```
matlab -batch "run('Code/predictor/train_1075.m')"  % follows train_1075_demo.txt; jackknife LIBSVM (C=1, gamma=0.03125) on 3240-dim features
```

## Evaluation signals

- All-features Jackknife baseline matches published values within tolerance: PDB1075 Acc 80.84%, MCC 0.617; PDB594 Acc 78.45%, MCC 0.569
- Selected-feature Jackknife model matches published values: PDB1075 Acc 88.00%, MCC 0.760, AUC 0.943; PDB594 Acc 93.60%, MCC 0.872, AUC 0.973
- Feature matrix dimensionality check: combined DDE+PSSM-DBT+PSSM-DWT+NMBAC vectors must be 3240-dimension before selection
- Relative-improvement check vs state-of-the-art: +1.8% Acc / +3.0% MCC on PDB1075 and +7.4% Acc / +14.8% MCC on PDB594
- SVM-RFE+CBR run must yield a monotone accuracy-vs-features curve recorded until the feature space is emptied, with the optimum at top 424 features on PDB1075

## Limitations

- No LIBSVM or PSI-BLAST version numbers stated in the paper (the README pins libsvm-3.21 and SVM-RFE-CBR-v1.3); other versions may shift metrics
- Jackknife cross-validation is leave-one-out over the full dataset and is computationally expensive; not practical for very large training sets
- The workspaces ship precomputed features, so reproduction is contingent on PSSM profiles generated by PSI-BLAST against nr; re-extraction requires the PDB*_seqs_PSSM directories
- Repository inconsistency: the PDB1075_workspace directory lists a file named DDE_DBT_DWT_NMBAC_594_workspace.mat, suggesting copy errors in the deposited tree

## Evidence

- [methods] SVM implementation with tuned parameters: "The SVM model was implemented using LIBSVM package[43] with the radial basis function."
- [methods] Classifier hyperparameters: [evidence span withheld — anchor status: not_found]
- [abstract] Published benchmark improvement to reproduce: "the proposed iDBP-DEP achieved 1.8 % and 3.0 % improvements of accuracy (Acc) and Mathew's Correlation Coefficient (MCC), respectively, on PDB1075 dataset"
- [readme] Deposited feature data location: "The feature Data used in our paper from ~/Data/PDB186(PDB1075/PDB594)/feature Data"
