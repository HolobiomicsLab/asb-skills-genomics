---
name: svm-recursive-feature-elimination-feature-selection
description: Use when a concatenated high-dimensional feature vector (e.g. 3240-dimension
  DDE+PSSM-DBT+PSSM-DWT+NMBAC for PDB1075) contains noisy/redundant descriptors and
  an SVM classifier needs a compact, bias-reduced feature subset before training.
when_to_use_negative:
- The input is already a small, non-redundant feature table where wrapper-based elimination
  adds cost without variance reduction
- The task requires a filter or embedded method independent of a specific SVM classifier
  (SVM-RFE+CBR weights are tied to the LIBSVM RBF model with C=1, gamma=0.03125)
- Only individual descriptor spaces are being compared without a concatenated multi-view
  vector — the 424-dimension optimum was derived for the combined 3240-dimension space
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: LIBSVM
  role: Implements the RBF-kernel SVM (C=1, gamma=0.03125) whose feature weights drive
    the recursive elimination and whose Jackknife accuracy scores each candidate subset
- name: SVM-RFE-CBR-v1.3
  role: Bundled SVM-RFE+CBR implementation performing SVM-RFE ranking with Correlation
    Bias Reduction inside the iDBP-DEP classifier code
  repo: https://githup.com/Zll-codeside/iDBP-DEP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:9daf4f08f7defe62511d0798007367e9b556c100dce0e2a8a1ff402106ca381a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LIBSVM
  - SVM-RFE-CBR-v1.3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# SVM-RFE+CBR Recursive Feature Elimination for Optimal Feature Subset Selection

## Summary

Ranks features in high-dimensional protein feature spaces by recursively removing lowest-weight SVM features with Correlation Bias Reduction (SVM-RFE+CBR), then selects the top-ranked subset size maximizing Jackknife cross-validation accuracy — in iDBP-DEP this reduces the 3240-dimension multi-view vector (DDE, PSSM-DBT, PSSM-DWT, NMBAC) to an optimal 424-dimension subset.

## When to use

Use when a concatenated high-dimensional feature vector (e.g. 3240-dimension DDE+PSSM-DBT+PSSM-DWT+NMBAC for PDB1075) contains noisy/redundant descriptors and an SVM classifier needs a compact, bias-reduced feature subset before training.

## When NOT to use

- The input is already a small, non-redundant feature table where wrapper-based elimination adds cost without variance reduction
- The task requires a filter or embedded method independent of a specific SVM classifier (SVM-RFE+CBR weights are tied to the LIBSVM RBF model with C=1, gamma=0.03125)
- Only individual descriptor spaces are being compared without a concatenated multi-view vector — the 424-dimension optimum was derived for the combined 3240-dimension space

## Inputs

- 3240-dimension concatenated multi-view feature vectors (.mat workspaces: DDE_workspace.mat, PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, feature_NMBAC_workspace.mat) for PDB1075 (525 DBPs, 550 non-DBPs), PDB594 (297/297), or PDB186 (93/93)
- Per-descriptor feature blocks: DDE, PSSM-DBT (DBT-encoded PSSM), PSSM-DWT (4-scale DWT statistics), NMBAC with nlag=30
- SVM hyperparameters from grid search: C=1, gamma=0.03125 (RBF kernel)

## Outputs

- Ranked feature index list over all 3240 features
- Accuracy-vs-subset-size curve (Jackknife cross-validation at each elimination step)
- Optimal 424-item feature selection with per-descriptor composition (51 DDE, 202 PSSM-DBT, 129 PSSM-DWT, 42 NMBAC)
- Reduced feature matrix fed into the SVM classifier for DBP detection

## How to apply

Load the concatenated multi-view feature matrices from the deposited workspace files (e.g. PDB1075_workspace: DDE_workspace.mat, PSSM_DBT_workspace.mat, PSSM_DWT_workspace.mat, feature_NMBAC_workspace.mat). Rank all features with SVM-RFE+CBR (SVM-RFE-CBR-v1.3 bundled with libsvm-3.21 in the repo's Code/classifier directory): compute SVM feature weights with the RBF kernel at C=1 and gamma=0.03125, iteratively delete the lowest-weight feature, and apply Correlation Bias Reduction during ranking to mitigate redundancy bias. At each elimination step, train the SVM on the remaining features and record Jackknife cross-validation accuracy until the feature set is empty. Sweep ranked subsets from top 10 to top 3240 in steps of 2 and keep the size with the highest accuracy; for iDBP-DEP on PDB1075 this optimum is 424 dimensions (51 DDE, 202 PSSM-DBT, 129 PSSM-DWT, 42 NMBAC), yielding 88.00% Jackknife accuracy (MCC 0.760). Write the ranked indices and the selected 424 features to an output such as selected_features_424.csv for reuse in classifier training (e.g. train_1075.m).

## Related tools

- **LIBSVM** (Implements the RBF-kernel SVM (C=1, gamma=0.03125) whose feature weights drive the recursive elimination and whose Jackknife accuracy scores each candidate subset)
- **SVM-RFE-CBR-v1.3** (Bundled SVM-RFE+CBR implementation performing SVM-RFE ranking with Correlation Bias Reduction inside the iDBP-DEP classifier code) — https://githup.com/Zll-codeside/iDBP-DEP

## Examples

```
run('Code/predictor/train_1075.m') % loads PDB1075_workspace .mat features, runs SVM-RFE-CBR-v1.3 with libsvm-3.21 (C=1, gamma=0.03125), selects the optimal 424-feature subset
```

## Evaluation signals

- Optimal subset size must equal 424 dimensions for the combined DDE+PSSM-DBT+PSSM-DWT+NMBAC space on PDB1075
- Jackknife cross-validation accuracy at the 424-feature subset should be at or near the reported 88.00% with MCC 0.760
- Per-descriptor composition of the selected subset must match 51 DDE, 202 PSSM-DBT, 129 PSSM-DWT, 42 NMBAC items
- Accuracy curve is monotone-consistent: each point corresponds to one subset size on the fixed SVM-RFE+CBR ranking (steps of 2 from 10 to 3240)
- SVM weights reproducible only with the fixed RBF parameters C=1, gamma=0.03125; reranking with other hyperparameters invalidates the reported selection

## Limitations

- No version numbers stated for LIBSVM beyond the bundled libsvm-3.21 in the repository; the article itself does not state a version
- Feature selection was applied to obtain optimal subsets in each protein feature space — results are tied to the specific PDB1075/PDB594/PDB186 datasets and no accession IDs or provenance are given for them
- The optimum (424-dimension, 88.00% Jackknife accuracy) is dataset-specific; re-running on other datasets requires redoing the full elimination sweep

## Evidence

- [methods] According to the experiment search result, C was finally set as 1 and  was set as 0.03125.: [evidence span withheld — anchor status: not_found]
- [intro] we combined them with PSSM-DWT (1040-dimension) and NMBAC (200-dimension) descriptor to obtain 3240-dimension feature vector: [evidence span withheld — anchor status: not_found]
