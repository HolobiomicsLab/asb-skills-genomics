---
name: differential-expression-detection-evaluation
description: Use when a normalization method for single-cell RNA-seq must be evaluated
  for its effect on DE detection, using simulated two-group scenarios with known DE
  genes and true fold changes.
when_to_use_negative:
- When no ground truth (planted DE genes or true fold changes) is available, so neither
  F1 nor fold-change RMSE can be computed.
- When the goal is evaluating normalization on clustering or batch-effect removal
  rather than differential expression detection.
- When the dataset is bulk RNA-seq or lacks a two-population comparison structure
  for DE calling.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scKWARN
  role: Method under test; LocASN normalization on scenario count matrices producing
    NormalizedData and per-cell scalingFactor
  repo: https://github.com/cyhsuTN/scKWARN
- name: devtools
  role: Used to install scKWARN from GitHub into R
  repo: https://github.com/cyhsuTN/scKWARN
- name: Rcpp
  role: Imported dependency of the scKWARN package
- name: Matrix
  role: Imported dependency supporting sparse-matrix input to LocASN
- name: stats
  role: Imported R dependency used for DE calling and RMSE computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae008
    title: 'scKWARN: Kernel-weighted-average robust normalization for single-cell
      RNA-seq data'
  source_content_hash: sha256:7922b1a8d756acbd68eae09763f55d17941355988d22b13ddbc6af74b1e6823f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cyhsuTN/scKWARN
  tools:
  - Matrix
  - R
  - Rcpp
  - devtools
  - scKWARN
  - stats
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# differential-expression-detection-evaluation

## Summary

Benchmark differentially expressed gene detection after scRNA-seq normalization by computing F1 scores against planted DE genes and RMSE between true and estimated fold changes across simulated two-population scenarios on the 10x Genomics PBMC33K dataset.

## When to use

Use when a normalization method for single-cell RNA-seq must be evaluated for its effect on DE detection, using simulated two-group scenarios with known DE genes and true fold changes.

## When NOT to use

- When no ground truth (planted DE genes or true fold changes) is available, so neither F1 nor fold-change RMSE can be computed.
- When the goal is evaluating normalization on clustering or batch-effect removal rather than differential expression detection.
- When the dataset is bulk RNA-seq or lacks a two-population comparison structure for DE calling.

## Inputs

- 10x Genomics PBMC33K count matrix
- Simulated two-population count matrices (10%/25% DE genes; 1000:1000 and 1000:100 balanced/unbalanced cells)
- List of planted (true) DE genes per scenario
- True fold-change values per gene per scenario

## Outputs

- Per-method, per-scenario F1 score of DE gene calls against planted DE genes
- Per-method, per-scenario RMSE between true and estimated fold changes
- figure4_f1_rmse_results.csv results table replicating Figure 4

## How to apply

Download the 10x Genomics PBMC33K dataset and construct four simulated two-population scenarios crossing 10% vs 25% DE genes with balanced (1000:1000) vs unbalanced (1000:100) cell groups. Normalize each scenario count matrix with the method under test (e.g., scKWARN's LocASN, which accepts general or sparse matrices and returns NormalizedData as counts/scaling factor plus per-cell scalingFactor) and with baselines RC, scran, SCnorm, sctransform, and PsiNorm. For each method and scenario, call differentially expressed genes between the two populations and compute F1 score against the planted DE gene list, then compute RMSE between true and estimated fold changes. Collate one row per method per scenario into a results table replicating Figure 4. Judge success when a method achieves the highest F1 and lowest RMSE across all four scenarios, indicating fold-change estimates closely match the true planted values while preserving biological heterogeneity.

## Related tools

- **scKWARN** (Method under test; LocASN normalization on scenario count matrices producing NormalizedData and per-cell scalingFactor) — https://github.com/cyhsuTN/scKWARN
- **devtools** (Used to install scKWARN from GitHub into R) — https://github.com/cyhsuTN/scKWARN
- **Rcpp** (Imported dependency of the scKWARN package)
- **Matrix** (Imported dependency supporting sparse-matrix input to LocASN)
- **stats** (Imported R dependency used for DE calling and RMSE computation)

## Examples

```
library(devtools); install_github("cyhsuTN/scKWARN"); Result <- LocASN(countmatrix = countsimdata); Result$NormalizedData; Result$scalingFactor
```

## Evaluation signals

- F1 score of called DE genes against the planted DE gene list is computed for each method-scenario pair.
- RMSE between true and estimated fold changes is computed; lowest RMSE indicates accurate fold-change recovery.
- Results should show scKWARN with the highest F1 and lowest RMSE in all four PBMC33K scenarios (10%/25% DE; 1000:1000 vs 1000:100).
- Normalized output schema check: LocASN must return NormalizedData (counts/scaling factor) and a scalingFactor for each cell.
- Results table contains one row per method per scenario and is saved as figure4_f1_rmse_results.csv.

## Limitations

- Unbalanced designs (1000:100 cells) are explicitly included, so methods may over- or under-correct; existing methods may over- or under-correct due to technical biases beyond sequencing depth and restrictive model/distribution assumptions.
- Evaluation depends on simulated scenarios from negative binomial distributions, so conclusions are tied to the planted DE proportions and group sizes tested.
- scKWARN makes no assumption about specific data distributions or count-depth relationships; behavior outside such settings is not characterized here.

## Evidence

- [abstract] PBMC33K benchmark finding: "scKWARN outperforms existing methods in removing a variety of technical biases while preserving true biological heterogeneity"
- [other] scenario construction: [evidence span withheld — anchor status: not_found]
- [readme] LocASN outputs: "Outputs: NormalizedData (normalized data = counts/scaling factor) and scalingFactor (a scaling factor for each cell)"
- [readme] LocASN input types: "Inputs: countmatrix (general matrix or sparse matrix)"
- [other] metric definitions: [evidence span withheld — anchor status: not_found]
- [abstract] over/under-correction caveat: "these methods may lead to over or under-correction due to presence of technical biases beyond sequencing depth"
