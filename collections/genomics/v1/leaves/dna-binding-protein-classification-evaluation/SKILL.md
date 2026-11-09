---
name: dna-binding-protein-classification-evaluation
description: Use when you need to isolate the contribution of each individual feature
  descriptor (DDE, PSSM-DBT, PSSM-DWT, NMBAC) to a DNA-binding protein classifier,
  by running per-descriptor feature selection, SVM training, and Jackknife evaluation.
when_to_use_negative:
- The goal is the full fused 3240-dimension combined-feature model or the PDB186 independent
  validation, not per-descriptor isolation.
- Features are already selected or provided as a single pre-computed feature table
  spanning all descriptors — re-running per-descriptor SVM-RFE+CBR would be redundant.
- The input is raw sequences without precomputed PSSM profiles; descriptor generation
  (PSI-BLAST PSSM, DBT, DWT, NMBAC extraction) is a prerequisite handled by other
  skills.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: LIBSVM
  role: libsvm-3.21 implements the RBF-kernel SVM classifier trained on each optimal
    feature subset (C=1, gamma=0.03125)
  repo: https://githup.com/Zll-codeside/iDBP-DEP
- name: SVM-RFE-CBR-v1.3
  role: Performs SVM-RFE+CBR feature selection within each individual descriptor space
    to obtain the optimal feature subset
  repo: https://githup.com/Zll-codeside/iDBP-DEP
- name: MATLAB
  role: Runs the per-descriptor predictor scripts (train_1075.m, before_FS.m, train_1075_demo.txt)
    and loads the .mat feature workspaces
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:d041faae25d65cd12da146af4bc5a6fd79a054c698e4bbb4e7f679be4d37dd88
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LIBSVM
  - MATLAB
  - SVM-RFE-CBR-v1.3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# dna-binding-protein-classification-evaluation

## Summary

Reproduce per-descriptor ablation of iDBP-DEP on PDB1075/PDB594: run SVM-RFE+CBR feature selection independently within each descriptor space (DDE, PSSM-DBT, PSSM-DWT, NMBAC), train LIBSVM RBF-kernel SVMs on each optimal subset, and evaluate with Jackknife (leave-one-out) cross-validation.

## When to use

Use when you need to isolate the contribution of each individual feature descriptor (DDE, PSSM-DBT, PSSM-DWT, NMBAC) to a DNA-binding protein classifier, by running per-descriptor feature selection, SVM training, and Jackknife evaluation.

## When NOT to use

- The goal is the full fused 3240-dimension combined-feature model or the PDB186 independent validation, not per-descriptor isolation.
- Features are already selected or provided as a single pre-computed feature table spanning all descriptors — re-running per-descriptor SVM-RFE+CBR would be redundant.
- The input is raw sequences without precomputed PSSM profiles; descriptor generation (PSI-BLAST PSSM, DBT, DWT, NMBAC extraction) is a prerequisite handled by other skills.

## Inputs

- Per-descriptor MATLAB .mat feature workspaces from the iDBP-DEP repo: DDE_workspace.mat / DDE594_workspace.mat, PSSM_DBT_workspace.mat / PSSM_DBT_594_workspace.mat, PSSM_DWT_workspace.mat / PSSM_DWT_594_workspace.mat, feature_NMBAC_workspace.mat / feature_NMBAC_594_workspace.mat
- Benchmark datasets PDB1075 (525 DBPs, 550 non-DBPs) and PDB594 (297 DBPs, 297 non-DBPs)

## Outputs

- Per-descriptor optimal feature subsets produced by SVM-RFE+CBR (one subset per descriptor space)
- feature_ablation_after_fs.csv with Jackknife Sn, Sp, Acc, MCC and AUC per descriptor per dataset
- Trained LIBSVM RBF-SVM models per descriptor (C=1, gamma=0.03125)

## How to apply

Load the per-descriptor feature workspaces (e.g. DDE_workspace.mat, PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, feature_NMBAC_workspace.mat for PDB1075; *_594_workspace.mat for PDB594) from the iDBP-DEP repository. For each descriptor individually, run the SVM-RFE+CBR algorithm (SVM-RFE-CBR-v1.3 in Code/classifier) to rank features and retain the optimal subset with the highest SVM accuracy; note that PSSM-DBT uses parameter T=3 and NMBAC uses nlag=30. Train the LIBSVM (libsvm-3.21) SVM with the radial basis function kernel using grid-search-optimized parameters C=1 and gamma=0.03125 on each selected feature subset. Evaluate each trained model with the Jackknife (leave-one-out) test, computing Sn, Sp, Acc, MCC per formulas 16-19 plus AUC. Compile per-descriptor metrics for both datasets into feature_ablation_after_fs.csv and validate against the reported individual-descriptor rows, where PSSM-DBT achieves the highest performance (Acc 82.70%, MCC 0.654, AUC 0.908 on PDB1075; Acc 83.67%, MCC 0.674, AUC 0.909 on PDB594).

## Related tools

- **LIBSVM** (libsvm-3.21 implements the RBF-kernel SVM classifier trained on each optimal feature subset (C=1, gamma=0.03125)) — https://githup.com/Zll-codeside/iDBP-DEP
- **SVM-RFE-CBR-v1.3** (Performs SVM-RFE+CBR feature selection within each individual descriptor space to obtain the optimal feature subset) — https://githup.com/Zll-codeside/iDBP-DEP
- **MATLAB** (Runs the per-descriptor predictor scripts (train_1075.m, before_FS.m, train_1075_demo.txt) and loads the .mat feature workspaces)

## Examples

```
svm-train -s 0 -t 2 -c 1 -g 0.03125 PSSM_DBT_workspace_selected_subset.txt
```

## Evaluation signals

- Per-descriptor Jackknife metrics match the reported individual-descriptor rows exactly.
- PSSM-DBT is the top descriptor: Acc 82.70%, MCC 0.654, AUC 0.908 on PDB1075 and Acc 83.67%, MCC 0.674, AUC 0.909 on PDB594.
- SVM hyperparameters are exactly C=1 and gamma=0.03125 as set by grid search.
- Exactly one optimal feature subset is produced per descriptor space (4 subsets per dataset), demonstrating independent per-descriptor feature selection.
- Metrics are computed via formulas 16-19 under the Jackknife (leave-one-out) protocol on the full PDB1075 and PDB594 datasets.

## Limitations

- Only combined-model improvements (1.8% Acc / 3.0% MCC on PDB1075; 7.4% Acc / 14.8% MCC on PDB594) and PDB186 independent-validation Acc 80.1%, MCC 0.684 are reported; per-descriptor ablation values must be reproduced from the workspaces.
- No version numbers stated for LIBSVM in the paper (libsvm-3.21 is only identifiable from the repo); Jackknife testing on PDB1075 is computationally expensive.
- Dataset labels/provenance: PDB1075 compiled by Liu et al., PDB594 and PDB186 constructed by Lou et al., with no accession IDs given.

## Evidence

- [methods] According to the experiment search result, C was finally set as 1 and  was set as 0.03125.: [evidence span withheld — anchor status: not_found]
