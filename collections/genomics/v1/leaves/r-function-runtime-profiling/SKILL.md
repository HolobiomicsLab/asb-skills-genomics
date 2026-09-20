---
name: r-function-runtime-profiling
description: Use when comparing computational cost of competing R normalization (or
  other per-function) pipelines across datasets of increasing G (genes) and n (cells)
  and need comparable elapsed-time entries plus explicit failure records.
when_to_use_negative:
- The goal is normalization quality (bias removal or heterogeneity preservation) rather
  than computational cost — use accuracy benchmarks instead
- The methods under comparison require incompatible input types that cannot be standardized
  to a common count-matrix format, making timings non-comparable
edam_operation: http://edamontology.org/operation_2984
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: R
  role: Runtime environment in which the benchmark and system.time timing are executed
  repo: https://github.com/cyhsuTN/scKWARN
- name: scKWARN
  role: Provides the LocASN normalizer timed in the benchmark; installed via devtools::install_github('cyhsuTN/scKWARN')
  repo: https://github.com/cyhsuTN/scKWARN
- name: devtools
  role: Used to install scKWARN from GitHub before benchmarking
- name: Matrix
  role: Supplies sparse matrix support so LocASN can accept sparseMatrix input during
    timing runs
- name: SCnorm
  role: Normalizer timed in the benchmark; fails on some large datasets and is recorded
    as not-completed
- name: sctransform
  role: Normalizer timed in the benchmark; among the slowest methods
- name: scran
  role: Normalizer timed in the benchmark; requires more time than RC and PsiNorm
- name: PsiNorm
  role: Normalizer timed in the benchmark; among the fastest methods
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae008
    title: 'scKWARN: Kernel-weighted-average robust normalization for single-cell
      RNA-seq data'
  source_content_hash: sha256:cfcb4a3666a0a4a91c2a8ef4060810c628d050bbac8b163cded13275fba1cb85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cyhsuTN/scKWARN
  tools:
  - Matrix
  - PsiNorm
  - R
  - SCnorm
  - devtools
  - scKWARN
  - scran
  - sctransform
  edam_operation: http://edamontology.org/operation_2984
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# r-function-runtime-profiling

## Summary

Benchmark wall-clock runtime of R normalization functions by wrapping each method call in system.time across datasets of varying gene/cell size, recording elapsed seconds and failures for methods that error or do not terminate.

## When to use

Use when comparing computational cost of competing R normalization (or other per-function) pipelines across datasets of increasing G (genes) and n (cells) and need comparable elapsed-time entries plus explicit failure records.

## When NOT to use

- The goal is normalization quality (bias removal or heterogeneity preservation) rather than computational cost — use accuracy benchmarks instead
- The methods under comparison require incompatible input types that cannot be standardized to a common count-matrix format, making timings non-comparable

## Inputs

- Count matrix (general R matrix or sparse Matrix) per dataset size, simulated from negative binomial distributions with G genes and n cells
- R functions under benchmark: RC, PsiNorm, scran, scKWARN LocASN, sctransform, SCnorm

## Outputs

- runtime_comparison.csv with one row per method per dataset size containing method name, number of genes, number of cells, elapsed seconds, and status (completed or failed)
- Per-call system.time elapsed wall-clock measurements

## How to apply

Generate or load count matrices at several sizes, sweeping the number of genes (G) and cells (n) over a size grid beyond a baseline such as G=2000, n=600, simulating counts from negative binomial distributions with set.seed(12345), gene means drawn via rgamma(G, shape = 2, rate = 2), and dispersion size = 0.1 via rnbinom. For each dataset size, call each normalizer — here RC, PsiNorm, scran, scKWARN's LocASN, sctransform, and SCnorm, accepting either plain or sparse Matrix input — wrapped in system.time to record elapsed wall-clock seconds. Decision point: if a method errors or does not terminate on a large dataset (e.g. SCnorm on the largest size), record it as a failed/not-completed entry rather than a runtime. Assemble one row per method per size (method, genes, cells, seconds, status) and check the expected runtime hierarchy, here RC/PsiNorm < scran/scKWARN < sctransform/SCnorm, holds.

## Related tools

- **R** (Runtime environment in which the benchmark and system.time timing are executed) — https://github.com/cyhsuTN/scKWARN
- **scKWARN** (Provides the LocASN normalizer timed in the benchmark; installed via devtools::install_github('cyhsuTN/scKWARN')) — https://github.com/cyhsuTN/scKWARN
- **devtools** (Used to install scKWARN from GitHub before benchmarking)
- **Matrix** (Supplies sparse matrix support so LocASN can accept sparseMatrix input during timing runs)
- **SCnorm** (Normalizer timed in the benchmark; fails on some large datasets and is recorded as not-completed)
- **sctransform** (Normalizer timed in the benchmark; among the slowest methods)
- **scran** (Normalizer timed in the benchmark; requires more time than RC and PsiNorm)
- **PsiNorm** (Normalizer timed in the benchmark; among the fastest methods)

## Examples

```
library(devtools); install_github("cyhsuTN/scKWARN"); library(scKWARN); set.seed(12345); G <- 2000; n <- 600; mu <- rgamma(G, shape = 2, rate = 2); countsimdata <- sapply(1:n, function(j) rnbinom(G, size = 0.1, mu = mu)); system.time(Result <- LocASN(countmatrix = countsimdata))
```

## Evaluation signals

- runtime_comparison.csv exists in CSV format with exactly one runtime row per method (6 methods) per dataset size
- Wall-clock ordering RC/PsiNorm < scran/scKWARN < sctransform/SCnorm holds across dataset sizes
- SCnorm is marked as failed/not-completed on the largest dataset size rather than given a runtime
- Each row records number of genes, number of cells, elapsed seconds, and status for its method and size
- Simulated inputs use the fixed seed set.seed(12345) with mu = rgamma(G, shape = 2, rate = 2) and dispersion size = 0.1 so timings are reproducible

## Limitations

- SCnorm fails on some large datasets, so it yields no runtime at those sizes and must be recorded as failed
- Timing runs are specific to the hardware and R environment; the absolute elapsed seconds are not portable across machines
- Runtime conclusions (RC/PsiNorm fastest, sctransform/SCnorm slowest) were derived from the simulated NB datasets of Section 3.3 and may differ for real data

## Evidence

- [results] RC and PsiNorm were the fastest, scran and scKWARN required more time than RC and PsiNorm, and sctransform and SCnorm had the longest computational time: [evidence span withheld — anchor status: not_found]
- [readme] Inputs: countmatrix (general matrix or sparse matrix): "### Inputs: countmatrix (general matrix or sparse matrix)"
- [readme] set.seed(12345); G <- 2000; n <- 600; mu <- rgamma(G, shape = 2, rate = 2); NB_cell <- function(j) rnbinom(G, size = 0.1, mu = mu): "set.seed(12345)
G <- 2000; n <- 600 # G: number of genes, n: number of cells
mu <- rgamma(G, shape = 2, rate = 2)
NB_cell <- function(j) rnbinom(G, size = 0.1, mu = mu)"
- [methods] library(devtools); install_github("cyhsuTN/scKWARN"): "library(devtools)
install_github("cyhsuTN/scKWARN")"
