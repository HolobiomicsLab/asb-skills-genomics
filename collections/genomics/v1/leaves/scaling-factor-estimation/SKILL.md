---
name: scaling-factor-estimation
description: Use when raw single-cell RNA-seq count matrices (general or sparse Matrix,
  e.g. dgCMatrix) exhibit technical biases from sequencing depth, capture efficiency,
  or dropout, and per-cell scaling factors plus normalized counts are needed without
  linear count-depth or distributional assumptions.
when_to_use_negative:
- The input is not raw single-cell RNA-seq counts (e.g. bulk RNA-seq, already-normalized
  expression values, or a feature/summary table) — LocASN expects a count matrix.
- A method-specific distributional or count-depth assumption is deliberately desired
  (e.g. reproducing SCnorm or sctransform behavior for comparison); scKWARN's design
  specifically avoids restrictive model/distribution assumptions, so results are not
  interchangeable.
- Only cell-level size factors compatible with library-size scaling workflows (e.g.
  scran relative counts) are required without kernel-weighted neighbor-based correction.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scKWARN
  role: Provides the LocASN function that computes kernel-weighted-average robust
    per-cell scaling factors and normalized data
  repo: https://github.com/cyhsuTN/scKWARN
- name: devtools
  role: Used to install scKWARN from GitHub (install_github('cyhsuTN/scKWARN'))
  repo: https://github.com/cyhsuTN/scKWARN
- name: Matrix
  role: R dependency enabling general or sparse matrix input to LocASN and dgCMatrix
    output of normalized data
- name: Rcpp
  role: R dependency (Imports) supporting the compiled implementation of scKWARN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae008
    title: 'scKWARN: Kernel-weighted-average robust normalization for single-cell
      RNA-seq data'
  source_content_hash: sha256:4bc9be20b60a10b9345f6b6e26a523ed2028df752c2d20ac94d9f52d1220f485
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cyhsuTN/scKWARN
  tools:
  - Matrix
  - PsiNorm
  - R
  - Rcpp
  - SCnorm
  - devtools
  - methods
  - scKWARN
  - scran
  - sctransform
  - stats
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# scaling-factor-estimation

## Summary

Estimate per-cell scaling factors for single-cell RNA-seq count matrices using scKWARN's LocASN, a kernel-weighted-average robust normalization that corrects technical confounders (sequencing depth, capture efficiency, dropout) without assuming specific data distributions or count-depth relationships.

## When to use

Use when raw single-cell RNA-seq count matrices (general or sparse Matrix, e.g. dgCMatrix) exhibit technical biases from sequencing depth, capture efficiency, or dropout, and per-cell scaling factors plus normalized counts are needed without linear count-depth or distributional assumptions.

## When NOT to use

- The input is not raw single-cell RNA-seq counts (e.g. bulk RNA-seq, already-normalized expression values, or a feature/summary table) — LocASN expects a count matrix.
- A method-specific distributional or count-depth assumption is deliberately desired (e.g. reproducing SCnorm or sctransform behavior for comparison); scKWARN's design specifically avoids restrictive model/distribution assumptions, so results are not interchangeable.
- Only cell-level size factors compatible with library-size scaling workflows (e.g. scran relative counts) are required without kernel-weighted neighbor-based correction.

## Inputs

- gene-by-cell raw count matrix (general matrix or sparse Matrix, e.g. from rnbinom simulations with gamma-distributed means: mu <- rgamma(G, shape = 2, rate = 2))
- R environment with scKWARN 1.1.2 installed (requires Rcpp, Matrix, stats, methods)

## Outputs

- Result$NormalizedData: normalized count matrix (counts divided by the per-cell scaling factor), returned as a dgCMatrix sparse matrix
- Result$scalingFactor: numeric vector with one scaling factor per cell (e.g. 0.9121116, 1.0050549, ...)
- optionally log1p-transformed normalized data for clustering

## How to apply

Load the scKWARN package in R (installed from scKWARN_1.1.2.tar.gz or via devtools::install_github('cyhsuTN/scKWARN')) and call LocASN(countmatrix = ...) on the raw gene-by-cell count matrix, accepting either a general matrix or a sparse Matrix. LocASN builds a pseudo expression profile for each cell by borrowing information from its fuzzy technical neighbors through a kernel smoother, then compares it against a reference from cells with the same bimodality patterns to determine the normalization factor. Extract Result$NormalizedData (counts divided by the scaling factor) and Result$scalingFactor (one scaling factor per cell); apply log1p(Result$NormalizedData) before downstream clustering. In benchmark settings the method is validated by computing RMSE of normalized values against simulated truth and F1 scores for recovery of planted differential-expression structure, where scKWARN obtained the lowest RMSE and highest F1 across all five scenarios compared with RC, scran, SCnorm, sctransform, and PsiNorm.

## Related tools

- **scKWARN** (Provides the LocASN function that computes kernel-weighted-average robust per-cell scaling factors and normalized data) — https://github.com/cyhsuTN/scKWARN
- **devtools** (Used to install scKWARN from GitHub (install_github('cyhsuTN/scKWARN'))) — https://github.com/cyhsuTN/scKWARN
- **Matrix** (R dependency enabling general or sparse matrix input to LocASN and dgCMatrix output of normalized data)
- **Rcpp** (R dependency (Imports) supporting the compiled implementation of scKWARN)

## Examples

```
library(scKWARN); Result <- LocASN(countmatrix = countsimdata); head(Result$NormalizedData[,1:5]); head(Result$scalingFactor)
```

## Evaluation signals

- Result$NormalizedData equals the raw counts divided by Result$scalingFactor for every cell (counts/scaling factor invariant).
- Result$scalingFactor has exactly one positive numeric entry per cell (length equals ncol of the input matrix), with benchmark values near 1 (README example: 0.9121116, 1.0050549, 1.0181696).
- Normalized output is a 6+ x n sparse dgCMatrix preserving gene/cell dimensionality and names (e.g. rows g_1..., columns c_1...).
- On simulated benchmarks (3000 genes, three subpopulations, five scenarios), RMSE against simulated truth should be lowest and F1 for DE/cluster recovery highest relative to RC, scran, SCnorm, sctransform, and PsiNorm.
- log1p(Result$NormalizedData) yields values suitable for downstream clustering without residual count-depth trends.

## Limitations

- Existing normalization methods may over- or under-correct due to technical biases beyond sequencing depth and restrictive model/distribution assumptions — the motivation for scKWARN's assumption-free design; results depend on the kernel-smoothed pseudo expression profile and bimodality-pattern reference.
- Dropout zeros dominate the normalized output (README example shows many '.' entries), so interpret sparsity when assessing results.
- The method requires the scKWARN package (version 1.1.2 cited), installed locally from a tarball or GitHub; it is not on CRAN.

## Evidence

- [abstract] kernel smoother rationale: "scKWARN generates a pseudo expression profile for each cell by borrowing information from its fuzzy technical neighbors through a kernel smoother."
- [abstract] no distribution assumption: "designed to correct known or hidden technical confounders without assuming specific data distributions or count-depth relationships"
- [methods] LocASN call: "Result <- LocASN(countmatrix = countsimdata)"
- [methods] outputs: "Outputs: NormalizedData (normalized data = counts/scaling factor) and scalingFactor (a scaling factor for each cell)"
- [readme] sparse matrix output: "6 x 5 sparse Matrix of class "dgCMatrix""
- [readme] scaling factor values: "[1] 0.9121116 1.0050549 1.0181696 0.9786187 0.9722913 1.0133925"
- [results] benchmark result: [evidence span withheld — anchor status: not_found]
