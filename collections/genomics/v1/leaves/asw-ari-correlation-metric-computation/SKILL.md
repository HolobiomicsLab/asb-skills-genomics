---
name: asw-ari-correlation-metric-computation
description: Use when comparing normalization methods for scRNA-seq on datasets with
  known cell-type labels, especially to quantify whether clustering reflects biology
  rather than sequencing depth across cell-composition imbalance scenarios.
when_to_use_negative:
- Data lack ground-truth labels, so ARI (and label-anchored ASW interpretation) cannot
  be computed
- The question is absolute clustering quality on a single composition rather than
  comparative ranking of normalization methods under imbalance
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scKWARN 1.1.2
  role: Provides the LocASN normalization whose output is clustered and scored; installed
    via devtools from cyhsuTN/scKWARN
  repo: https://github.com/cyhsuTN/scKWARN
- name: Matrix
  role: Supports sparse-matrix (dgCMatrix) input of count matrices fed to LocASN before
    metric computation
  repo: https://github.com/cyhsuTN/scKWARN
- name: stats
  role: R package used for clustering and correlation computations on the log1p-normalized
    profiles
- name: Rcpp
  role: Imported dependency of the scKWARN package used during LocASN execution
  repo: https://github.com/cyhsuTN/scKWARN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae008
    title: 'scKWARN: Kernel-weighted-average robust normalization for single-cell
      RNA-seq data'
  source_content_hash: sha256:49fa1adcac9399ce08ad0da3665264f515b25340376f937ea75d6fd0e94bd64a
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
  - scKWARN 1.1.2
  - stats
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# asw-ari-correlation-metric-computation

## Summary

Evaluate clustering quality of normalized scRNA-seq data using ASW (average silhouette width), ARI (adjusted Rand index against ground-truth labels), and 1-correlation between expression PCs and sequencing depth, as done in the scKWARN benchmark on GSE118767 lung adenocarcinoma data.

## When to use

Use when comparing normalization methods for scRNA-seq on datasets with known cell-type labels, especially to quantify whether clustering reflects biology rather than sequencing depth across cell-composition imbalance scenarios.

## When NOT to use

- Data lack ground-truth labels, so ARI (and label-anchored ASW interpretation) cannot be computed
- The question is absolute clustering quality on a single composition rather than comparative ranking of normalization methods under imbalance

## Inputs

- log1p-transformed normalized expression matrix (log1p of LocASN NormalizedData from general or sparse dgCMatrix counts)
- Ground-truth cell-line labels for the GSE118767 H1975, H2228, HCC827, H838, and A549 cells
- Per-cell sequencing-depth values (library sizes) for the PC–depth correlation metric

## Outputs

- Per-scenario, per-method values of ASW, ARI, and 1-correlation (metrics_table.csv: 5 scenarios x 6 methods x 3 metrics)
- Cluster assignments from the log1p-normalized profiles

## How to apply

After normalizing counts with LocASN (scKWARN) or competing methods, apply log1p to the NormalizedData (counts divided by per-cell scalingFactor) and cluster the log1p-transformed profiles. Evaluate each clustering against ground-truth labels with three metrics: ASW for cohesion/separation, ARI for label agreement, and 1-correlation between expression PCs and sequencing depth to measure residual depth bias. In the scKWARN benchmark this is repeated across five simulated cell-composition scenarios on GSE118767 (balanced 1:1:1:1:1 through extremely imbalanced 36:1:1:1:1, including the H838-driven imbalance variant), yielding metrics for five scenarios by six methods. Assemble all metric values into a metrics_table.csv and compare methods per scenario; scKWARN attained the highest 1-correlation in every scenario.

## Related tools

- **scKWARN 1.1.2** (Provides the LocASN normalization whose output is clustered and scored; installed via devtools from cyhsuTN/scKWARN) — https://github.com/cyhsuTN/scKWARN
- **Matrix** (Supports sparse-matrix (dgCMatrix) input of count matrices fed to LocASN before metric computation) — https://github.com/cyhsuTN/scKWARN
- **stats** (R package used for clustering and correlation computations on the log1p-normalized profiles)
- **Rcpp** (Imported dependency of the scKWARN package used during LocASN execution) — https://github.com/cyhsuTN/scKWARN

## Examples

```
Result <- LocASN(countmatrix = counts); log_dat <- log1p(Result$NormalizedData); cl <- kmeans(t(log_dat), centers = 5)$cluster
```

## Evaluation signals

- 1-correlation between expression PCs and sequencing depth is maximal for scKWARN in every one of the five GSE118767 scenarios
- metrics_table.csv contains exactly 5 scenarios (1:1:1:1:1 to 36:1:1:1:1) x 6 methods x 3 metrics (ASW, ARI, 1-correlation)
- ARI is computed against the known ground-truth cell-line labels, so values are non-negative when clustering matches labels well
- Clustering is performed on log1p(NormalizedData), i.e., counts divided by per-cell scalingFactor, log1p-transformed

## Limitations

- Comparison depends on simulated cell-composition scenarios and specific subsampling of the seven GSE118767 matrices; results may not transfer to other tissues or batch structures
- Metric conclusions are specific to the competing normalization set used in the benchmark (SCnorm handled differently from the others)

## Evidence

- [readme] clustering input transformation: "# log1p(Result$NormalizedData) for clustering"
- [methods] LocASN outputs used for metric computation: "Outputs: NormalizedData (normalized data = counts/scaling factor) and scalingFactor (a scaling factor for each cell)"
- [abstract] benchmark outcome motivating the metrics: "As demonstrated in both simulated and real datasets, scKWARN outperforms existing methods in removing a variety of technical biases while preserving true biological heterogeneity."
