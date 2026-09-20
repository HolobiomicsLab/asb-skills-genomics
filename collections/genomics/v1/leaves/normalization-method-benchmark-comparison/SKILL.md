---
name: normalization-method-benchmark-comparison
description: Use when comparing scRNA-seq normalization methods under controlled conditions
  — simulated scenarios varying library size, RNA composition, dropout, population
  balance, or DE level — or against qRT-PCR-validated gold-standard DE genes.
when_to_use_negative:
- When the goal is production normalization of a single dataset rather than method
  comparison — apply LocASN directly instead of benchmarking
- When input is not a raw count matrix (e.g. already normalized or log-transformed
  data), since LocASN expects counts and divides by a scaling factor
- When the data follow a well-characterized count-depth relationship with no additional
  technical confounders, where simpler methods may already suffice
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scKWARN (LocASN)
  role: Normalization method under test; kernel-weighted-average robust normalization
    producing NormalizedData and per-cell scalingFactor from a general or sparse count
    matrix
  repo: https://github.com/cyhsuTN/scKWARN
- name: devtools
  role: Used to install scKWARN from GitHub in the benchmark environment
  repo: https://github.com/cyhsuTN/scKWARN
- name: scran
  role: Competitor baseline providing relative-count (RC) normalization in the comparison
- name: SCnorm
  role: Competitor baseline normalization method in the benchmark
- name: sctransform
  role: Competitor baseline normalization method; its gold-standard recovery proportion
    drops significantly in the all-cells-downsized GSE29087 setting
- name: PsiNorm
  role: Competitor baseline normalization method that, like scKWARN, maintains performance
    in the all-cells-downsized setting
- name: Matrix
  role: R package supporting sparse matrix inputs accepted by LocASN (dgCMatrix)
- name: Rcpp
  role: Imported dependency of the scKWARN package
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae008
    title: 'scKWARN: Kernel-weighted-average robust normalization for single-cell
      RNA-seq data'
  source_content_hash: sha256:2e87a6609d164d39ce1ae68908e4222697715b5469c7541a47500ce381450869
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
  - scKWARN (LocASN)
  - scran
  - sctransform
  - stats
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 24
---

# normalization-method-benchmark-comparison

## Summary

Benchmark single-cell RNA-seq normalization methods (scKWARN/LocASN vs RC/scran, SCnorm, sctransform, PsiNorm) using simulated or gold-standard datasets, scoring F1 for DE-gene recovery and RMSE against true fold changes or normalized values.

## When to use

Use when comparing scRNA-seq normalization methods under controlled conditions — simulated scenarios varying library size, RNA composition, dropout, population balance, or DE level — or against qRT-PCR-validated gold-standard DE genes.

## When NOT to use

- When the goal is production normalization of a single dataset rather than method comparison — apply LocASN directly instead of benchmarking
- When input is not a raw count matrix (e.g. already normalized or log-transformed data), since LocASN expects counts and divides by a scaling factor
- When the data follow a well-characterized count-depth relationship with no additional technical confounders, where simpler methods may already suffice

## Inputs

- Simulated count matrices (general matrix or sparse Matrix) generated from negative binomial distributions, e.g. via rnbinom with gamma-prior means
- Public scRNA-seq count matrices such as 10x Genomics PBMC33K or GEO GSE29087 (7896-gene subset, 92 mouse embryonic cells and fibroblasts)
- Planted DE gene labels or qRT-PCR-validated gold-standard DE gene lists (718 genes for GSE29087)

## Outputs

- Per-cell scalingFactor vector and NormalizedData matrix from LocASN
- F1 score table per method per scenario (e.g. f1_table.csv, 15 rows for 3 settings x 5 scenarios)
- RMSE table of normalized values vs simulated truth (rmse_table.csv)
- Figure 4 style F1/RMSE results (figure4_f1_rmse_results.csv) and Figure 5 style top-N gold-standard proportions per downsizing panel (figure5_topN_proportions.csv)

## How to apply

Install scKWARN 1.1.2 (locally from scKWARN_1.1.2.tar.gz or via devtools::install_github('cyhsuTN/scKWARN')) and run LocASN(countmatrix = ...) on each scenario's count matrix, which accepts general or sparse Matrix input and returns NormalizedData (counts/scaling factor) plus a per-cell scalingFactor. Normalize the same matrices with RC (scran), SCnorm, sctransform, and PsiNorm. For simulation benchmarks (e.g. 3000 genes, three subpopulations, five scenarios, 100 replicates, counts drawn via rnbinom(G, size = 0.1, mu = mu) with gene means from rgamma(G, shape = 2, rate = 2)), compute per-scenario RMSE of normalized values against the simulated truth and F1 for recovery of planted DE/cluster structure. For real-data benchmarks (e.g. PBMC33K with 10%/25% DE at 1000:1000 or 1000:100 cell balance, or GSE29087 with 718 qRT-PCR-validated gold-standard genes under none/half/all cells downsized by Uniform(0.2,1)), rank genes and compare top-N recovery proportions or F1/RMSE. Aggregate results into per-method-per-scenario tables (e.g. 15 rows = 3 settings x 5 scenarios) mirroring the paper's figures.

## Related tools

- **scKWARN (LocASN)** (Normalization method under test; kernel-weighted-average robust normalization producing NormalizedData and per-cell scalingFactor from a general or sparse count matrix) — https://github.com/cyhsuTN/scKWARN
- **devtools** (Used to install scKWARN from GitHub in the benchmark environment) — https://github.com/cyhsuTN/scKWARN
- **scran** (Competitor baseline providing relative-count (RC) normalization in the comparison)
- **SCnorm** (Competitor baseline normalization method in the benchmark)
- **sctransform** (Competitor baseline normalization method; its gold-standard recovery proportion drops significantly in the all-cells-downsized GSE29087 setting)
- **PsiNorm** (Competitor baseline normalization method that, like scKWARN, maintains performance in the all-cells-downsized setting)
- **Matrix** (R package supporting sparse matrix inputs accepted by LocASN (dgCMatrix))
- **Rcpp** (Imported dependency of the scKWARN package)

## Examples

```
library(devtools); install_github("cyhsuTN/scKWARN"); Result <- LocASN(countmatrix = countsimdata); Result$NormalizedData; Result$scalingFactor
```

## Evaluation signals

- scKWARN should attain the lowest RMSE and highest F1 across all five scenarios in each of the three simulation settings, outperforming RC, scran, SCnorm, sctransform, and PsiNorm
- On PBMC33K, scKWARN should show the highest F1 and lowest RMSE in all four scenarios (10%/25% DE x balanced/unbalanced), with estimated fold changes closely matching true values (Figure 4 replication)
- On GSE29087, sctransform should lead the no- and half-downsizing panels while scKWARN and PsiNorm lead in the all-downsized panel, matching Figure 5
- Output CSVs must have the expected schema: 15 rows (3 settings x 5 scenarios) for simulation tables; three downsizing panels x six methods for the GSE29087 table
- LocASN outputs must contain NormalizedData equal to counts divided by the per-cell scalingFactor, with scalingFactor values near 1 (e.g. 0.912–1.018 in the README example)

## Limitations

- Existing methods may over- or under-correct due to technical biases beyond sequencing depth and restrictive assumptions on models and distributions
- sctransform's gold-standard DE recovery proportion drops significantly when all cells are downsized (high dropout) in the GSE29087 benchmark
- Simulation results depend on the generative choices (e.g. NB size = 0.1, gamma shape = 2 rate = 2 prior), so benchmark conclusions are conditional on those assumptions

## Evidence

- [abstract] scKWARN corrects confounders without distributional assumptions: "We present scKWARN, a Kernel Weighted Average Robust Normalization designed to correct known or hidden technical confounders without assuming specific data distributions or count-depth relationships."
- [abstract] scKWARN outperforms existing methods: "As demonstrated in both simulated and real datasets, scKWARN outperforms existing methods in removing a variety of technical biases while preserving true biological heterogeneity."
- [methods] LocASN inputs and installation: "Download scKWARN_1.1.2.tar.gz and locally install it, or execute the following code"
- [methods] LocASN accepts general or sparse matrix and outputs normalized data and scaling factor: "Outputs: NormalizedData (normalized data = counts/scaling factor) and scalingFactor (a scaling factor for each cell)"
- [readme] README simulation generation: "mu <- rgamma(G, shape = 2, rate = 2)
NB_cell <- function(j) rnbinom(G, size = 0.1, mu = mu)"
- [readme] README invocation of LocASN: "Result <- LocASN(countmatrix = countsimdata)"
