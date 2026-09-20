---
name: single-cell-rna-seq-normalization
description: Use when a raw scRNA-seq count matrix (general or sparse) shows sequencing-depth,
  capture-efficiency, dropout, or composition biases and distribution-free normalization
  is needed before DE or clustering.
when_to_use_negative:
- Input is not a count matrix (e.g. already log-transformed, TPM/CPM-normalized, or
  a feature/summary table) — LocASN expects raw counts and its reference profile uses
  geometric means over expressing cells
- A normalization method assuming a specific count-depth relationship or error distribution
  is explicitly required by the downstream analysis design (scKWARN deliberately avoids
  such assumptions)
- The dataset is single-bulk or non-single-cell transcriptomic data where per-cell
  size factors are not meaningful
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scKWARN
  role: Provides the LocASN function implementing the four-step kernel-weighted-average
    robust normalization (technical neighbors, kernel pseudo-profile, cell-specific
    reference, median-ratio size factor)
  repo: https://github.com/cyhsuTN/scKWARN
- name: R
  role: Runtime environment for simulation, installation, and calling LocASN
- name: devtools
  role: Installs the scKWARN package from GitHub (install_github('cyhsuTN/scKWARN'))
- name: Matrix
  role: Imports/sparse-matrix support so countmatrix can be a general or sparse matrix
    and NormalizedData is returned as dgCMatrix
- name: Rcpp
  role: Declared Imports dependency of the scKWARN package
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_004
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae008
    title: 'scKWARN: Kernel-weighted-average robust normalization for single-cell
      RNA-seq data'
  source_content_hash: sha256:172c84c4e61478bcf1eda711bdca5e6e336cea37838b6e5d39599797506b515d
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
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# single-cell-rna-seq-normalization

## Summary

Normalizes single-cell RNA-seq count matrices with scKWARN's LocASN, a kernel-weighted-average robust method that corrects known or hidden technical confounders without assuming data distributions or count-depth relationships.

## When to use

Use when a raw scRNA-seq count matrix (general or sparse) shows sequencing-depth, capture-efficiency, dropout, or composition biases and distribution-free normalization is needed before DE or clustering.

## When NOT to use

- Input is not a count matrix (e.g. already log-transformed, TPM/CPM-normalized, or a feature/summary table) — LocASN expects raw counts and its reference profile uses geometric means over expressing cells
- A normalization method assuming a specific count-depth relationship or error distribution is explicitly required by the downstream analysis design (scKWARN deliberately avoids such assumptions)
- The dataset is single-bulk or non-single-cell transcriptomic data where per-cell size factors are not meaningful

## Inputs

- Raw scRNA-seq count matrix (genes x cells), general matrix or sparse Matrix (e.g. from 10x Genomics PBMC33K or GEO accessions like GSE29087/GSE118767)
- Optionally, simulated negative-binomial count matrices (e.g. rnbinom with size = 0.1 and gamma-distributed means mu <- rgamma(G, shape = 2, rate = 2))

## Outputs

- NormalizedData: counts divided by per-cell scaling factor, returned as a dgCMatrix sparse matrix
- scalingFactor: one median-ratio normalization factor per cell (e.g. first six cells of the README simulation: 0.9121116, 1.0050549, 1.0181696, 0.9786187, 0.9722913, 1.0133925)

## How to apply

Install scKWARN 1.1.2 via devtools::install_github('cyhsuTN/scKWARN') and load Matrix for sparse support. Run Result <- LocASN(countmatrix = ...), which (a) defines technical neighbors via PC1 of the 25th/50th/75th percentiles of log non-zero counts, (b) builds a gene- and cell-specific pseudo-profile with a Gaussian kernel smoother using SJ bandwidth selection, (c) forms a per-cell reference profile as geometric means over cells expressing each gene, and (d) computes each cell's size factor theta_j = median{Aj/Mj}. Extract Result$NormalizedData (counts divided by the scaling factor, returned as a dgCMatrix) and Result$scalingFactor (one value per cell); apply log1p to NormalizedData before clustering. The PC1-only default is justified by ablations showing PC1+PC2 yields nearly identical RMSE/F1 while costing 38% more runtime, and the default weight cutoff = 2 saves 15–25% runtime without performance loss.

## Related tools

- **scKWARN** (Provides the LocASN function implementing the four-step kernel-weighted-average robust normalization (technical neighbors, kernel pseudo-profile, cell-specific reference, median-ratio size factor)) — https://github.com/cyhsuTN/scKWARN
- **R** (Runtime environment for simulation, installation, and calling LocASN)
- **devtools** (Installs the scKWARN package from GitHub (install_github('cyhsuTN/scKWARN')))
- **Matrix** (Imports/sparse-matrix support so countmatrix can be a general or sparse matrix and NormalizedData is returned as dgCMatrix)
- **Rcpp** (Declared Imports dependency of the scKWARN package)

## Examples

```
library(scKWARN); Result <- LocASN(countmatrix = countsimdata); log1p(Result$NormalizedData)
```

## Evaluation signals

- Result$scalingFactor contains one positive value per cell, all near 1 (README simulation first-six values fall in ~[0.91, 1.02])
- Result$NormalizedData is a sparse dgCMatrix of the same dimensions as the input (e.g. 2000 x 600) and equals the original counts divided by the per-cell scaling factors
- Row/column names (g_1..g_G, c_1..c_n) are preserved from the input count matrix
- Downstream check: clustering on log1p(Result$NormalizedData) shows low correlation between expression PCs and sequencing depth (highest 1-correlation across imbalance scenarios in the paper's benchmarks)
- Benchmark check: on simulated settings, RMSE against truth and F1 for DE/cluster recovery should match or beat RC, scran, SCnorm, sctransform, and PsiNorm

## Limitations

- SCnorm failed (non-termination) on some large datasets in the runtime benchmark; scKWARN was slower than RC and PsiNorm (runtime ordering: RC/PsiNorm < scran/scKWARN < sctransform/SCnorm)
- Performance guarantees are demonstrated for count matrices from simulations (3000-gene, three-subpopulation scenarios) and specific datasets (PBMC33K, GSE29087, GSE118767); behavior on other distributions/count-depth relationships is not established here
- The default cutoff = 2 on weights trades a small (15–25%) runtime saving for weight truncation; disabling it changes the smoothing details

## Evidence

- [abstract] designed to correct known or hidden technical confounders without assuming specific data distributions or count-depth relationships: "designed to correct known or hidden technical confounders without assuming specific data distributions or count-depth relationships"
- [methods] LocASN function interface: "### Inputs: countmatrix (general matrix or sparse matrix)"
- [methods] LocASN outputs: "Outputs: NormalizedData (normalized data = counts/scaling factor) and scalingFactor (a scaling factor for each cell)"
- [readme] README usage with simulation parameters: "set.seed(12345)
G <- 2000; n <- 600 # G: number of genes, n: number of cells
mu <- rgamma(G, shape = 2, rate = 2)
NB_cell <- function(j) rnbinom(G, size = 0.1, mu = mu)"
- [readme] README expected scaling factors: "## [1] 0.9121116 1.0050549 1.0181696 0.9786187 0.9722913 1.0133925"
