---
name: correlation-bias-reduction-feature-ranking
description: Use when a concatenated multi-view feature vector (e.g. 3240-dim DDE,
  PSSM-DBT, PSSM-DWT, NMBAC) must be reduced to a small optimal subset before SVM
  classification, removing noisy and redundant features.
when_to_use_negative:
- Input is already a small, curated feature table where redundancy removal would discard
  informative descriptors
- The classifier is not SVM-based — CBR's correlation-bias correction is designed
  for SVM-RFE weight rankings (paper uses LIBSVM RBF kernel)
- Dataset lacks labels — SVM-RFE+CBR is a supervised wrapper method and cannot rank
  features without class labels
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: LIBSVM
  role: Implements the RBF-kernel SVM (C=1, gamma=0.03125) whose feature weights drive
    the RFE elimination loop and per-iteration Jackknife accuracy evaluation; repo
    bundles libsvm-3.21
  repo: https://githup.com/Zll-codeside/iDBP-DEP
- name: SVM-RFE-CBR-v1.3
  role: Supplies the SVM-RFE with Correlation Bias Reduction ranking algorithm shipped
    in the repository's classifier folder
  repo: https://githup.com/Zll-codeside/iDBP-DEP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/minf.202000006
    title: Sequence‐based Detection of DNA‐binding Proteins using Multiple‐view Features
      Allied with Feature Selection
  source_content_hash: sha256:dcf9d117f880533db2fd7dd146648842ecde68afa1cb3e09aacd567de0423bc8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - LIBSVM
  - SVM-RFE-CBR-v1.3
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# correlation-bias-reduction-feature-ranking

## Summary

Rank high-dimensional multi-view protein features with SVM-RFE+CBR — SVM recursive feature elimination augmented with Correlation Bias Reduction — to strip noisy/redundant descriptors (e.g. the 3240-dim DDE+PSSM-DBT+PSSM-DWT+NMBAC vector) and locate the optimal subset (424 features, Jackknife Acc 88.00%, MCC 0.760 on PDB1075).

## When to use

Use when a concatenated multi-view feature vector (e.g. 3240-dim DDE, PSSM-DBT, PSSM-DWT, NMBAC) must be reduced to a small optimal subset before SVM classification, removing noisy and redundant features.

## When NOT to use

- Input is already a small, curated feature table where redundancy removal would discard informative descriptors
- The classifier is not SVM-based — CBR's correlation-bias correction is designed for SVM-RFE weight rankings (paper uses LIBSVM RBF kernel)
- Dataset lacks labels — SVM-RFE+CBR is a supervised wrapper method and cannot rank features without class labels

## Inputs

- Concatenated multi-view feature matrix, 3240-dimension per protein (DDE, PSSM-DBT, PSSM-DWT, NMBAC), e.g. DDE_DBT_DWT_NMBAC_594_workspace.mat / PDB1075 workspace .mat files
- Class labels for the benchmark dataset (PDB1075: 525 DBPs / 550 non-DBPs; PDB594: 297/297; PDB186: 93/93)

## Outputs

- Ranked feature list ordered by SVM-RFE+CBR weight (e.g. selected_features_424.csv with ranked indices)
- Jackknife accuracy curve over subset sizes (top 10 to top 3240, step 2)
- Optimal feature subset (424 dimensions: 51 DDE, 202 PSSM-DBT, 129 PSSM-DWT, 42 NMBAC items)

## How to apply

Load the concatenated feature matrices (e.g. DDE_DBT_DWT_NMBAC workspace .mat files for PDB1075: 525 DBPs, 550 non-DBPs) and rank all 3240 features with the SVM-RFE+CBR algorithm from the repo's classifier directory (SVM-RFE-CBR-v1.3): compute SVM feature weights with LIBSVM (RBF kernel, C=1, gamma=0.03125 as set by the paper's grid search), iteratively delete the lowest-weight feature, and apply Correlation Bias Reduction to counter redundancy bias that RFE alone introduces when correlated descriptors compete. At each elimination iteration train the SVM on the surviving features and record Jackknife cross-validation accuracy, sweeping ranked subsets from top 10 to top 3240 in steps of 2; the size with highest accuracy is the optimum. The rationale is that RFE ranking is biased toward correlated feature clusters, so CBR re-ranks within each cluster to keep the most informative representative rather than redundant duplicates.

## Related tools

- **LIBSVM** (Implements the RBF-kernel SVM (C=1, gamma=0.03125) whose feature weights drive the RFE elimination loop and per-iteration Jackknife accuracy evaluation; repo bundles libsvm-3.21) — https://githup.com/Zll-codeside/iDBP-DEP
- **SVM-RFE-CBR-v1.3** (Supplies the SVM-RFE with Correlation Bias Reduction ranking algorithm shipped in the repository's classifier folder) — https://githup.com/Zll-codeside/iDBP-DEP

## Examples

```
run SVM-RFE-CBR-v1.3 on PDB1075 workspace features with LIBSVM RBF kernel (C=1, gamma=0.03125) via the repo's train_1075_demo.txt workflow to obtain the ranked top-424 feature subset
```

## Evaluation signals

- Optimal subset size must equal 424 dimensions on the all-features PDB1075 combination
- Jackknife cross-validation accuracy at the optimum at or near the reported 88.00% (MCC 0.760) on PDB1075
- Per-descriptor composition of the selected subset must match 51 DDE, 202 PSSM-DBT, 129 PSSM-DWT, 42 NMBAC items
- Accuracy-vs-subset-size curve must be monotone-checkable across the top-10-to-top-3240 sweep (step 2) with a single peak
- Ranked output must be a full permutation of the 3240 input feature indices with no duplicates or omissions

## Limitations

- No LIBSVM version stated in the article (repo bundles libsvm-3.21), so exact numerical reproduction may vary across LIBSVM builds
- Wrapper-style RFE over 3240 features with Jackknife validation is computationally expensive; the sweep granularity (step of 2) is an approximation of the true optimum
- Optimal subset (424 features) was identified for the PDB1075 benchmark and may not transfer directly to other datasets such as PDB594 or PDB186

## Evidence

- [methods] SVM grid-search parameters C=1, gamma=0.03125: [evidence span withheld — anchor status: not_found]
- [intro] 3240-dimension concatenated feature vector: [evidence span withheld — anchor status: not_found]
