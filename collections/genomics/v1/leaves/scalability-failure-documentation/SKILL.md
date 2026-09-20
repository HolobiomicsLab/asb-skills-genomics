---
name: scalability-failure-documentation
description: Use when comparing computational scalability of normalization methods
  on simulated scRNA-seq count matrices of varying gene/cell counts and a method errors
  or hangs on large datasets — record the failure as a status, not a runtime.
when_to_use_negative:
- When the goal is accuracy or bias-removal quality of normalization rather than wall-clock
  runtime and scalability behavior
- When the input is real (non-simulated) count data and the article's seeded negative-binomial
  simulation design cannot be applied
- When only a single method is being timed, so no cross-method runtime ordering can
  be validated
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: scKWARN 1.1.2
  role: Normalization method under benchmark; its LocASN function is timed on each
    simulated count matrix
  repo: https://github.com/cyhsuTN/scKWARN
- name: R
  role: Runtime environment; simulation with rnbinom/rgamma and timing via system.time
    are done in R
- name: devtools
  role: Used to install scKWARN from GitHub (install_github)
- name: Matrix
  role: Provides sparse matrix representation accepted as countmatrix input to LocASN
- name: SCnorm
  role: Comparison method with the longest runtime that fails on some large datasets;
    recorded as failed status
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae008
    title: 'scKWARN: Kernel-weighted-average robust normalization for single-cell
      RNA-seq data'
  source_content_hash: sha256:c1b55f8ce85b023e952259a0c1ad473c3b4cb7529767d8b9e9ca88c3387a74ed
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
  - scKWARN 1.1.2
  - scran
  - sctransform
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# scalability-failure-documentation

## Summary

A benchmarking skill for systematically recording the computational runtime of scKWARN (LocASN) against RC, scran, SCnorm, sctransform, and PsiNorm across increasing simulated negative-binomial dataset sizes, explicitly documenting method failures on large inputs rather than silently dropping them.

## When to use

Use when comparing computational scalability of normalization methods on simulated scRNA-seq count matrices of varying gene/cell counts and a method errors or hangs on large datasets — record the failure as a status, not a runtime.

## When NOT to use

- When the goal is accuracy or bias-removal quality of normalization rather than wall-clock runtime and scalability behavior
- When the input is real (non-simulated) count data and the article's seeded negative-binomial simulation design cannot be applied
- When only a single method is being timed, so no cross-method runtime ordering can be validated

## Inputs

- Simulated scRNA-seq count matrices from negative binomial distributions (G genes x n cells), e.g. baseline G=2000, n=600 plus larger size-grid combinations
- Plain R matrices or sparse Matrix (dgCMatrix) objects accepted by LocASN's countmatrix argument

## Outputs

- runtime_comparison.csv with one row per method (6 methods) per dataset size containing method name, number of genes, number of cells, elapsed seconds, and status
- Explicit failed/not-completed status entries for SCnorm on large datasets where it errors or does not terminate

## How to apply

Simulate count matrices from negative binomial distributions with set.seed(12345), drawing gene means as mu <- rgamma(G, shape = 2, rate = 2) and counts via rnbinom(G, size = 0.1, mu = mu), sweeping G (genes) and n (cells) over a size grid in addition to the README baseline G=2000, n=600. At each size, wrap each of the six normalizers (RC, PsiNorm, scran, scKWARN's LocASN accepting general or sparse Matrix input, sctransform, SCnorm) in system.time to capture elapsed wall-clock seconds. Treat SCnorm errors or non-termination on large datasets as a failed/not-completed entry with an explicit status field rather than a runtime. Assemble one row per method per size (method, genes, cells, elapsed seconds, status) into runtime_comparison.csv and check the expected hierarchy RC/PsiNorm < scran/scKWARN < sctransform/SCnorm holds.

## Related tools

- **scKWARN 1.1.2** (Normalization method under benchmark; its LocASN function is timed on each simulated count matrix) — https://github.com/cyhsuTN/scKWARN
- **R** (Runtime environment; simulation with rnbinom/rgamma and timing via system.time are done in R)
- **devtools** (Used to install scKWARN from GitHub (install_github))
- **Matrix** (Provides sparse matrix representation accepted as countmatrix input to LocASN)
- **SCnorm** (Comparison method with the longest runtime that fails on some large datasets; recorded as failed status)

## Examples

```
library(devtools); install_github("cyhsuTN/scKWARN"); Result <- LocASN(countmatrix = countsimdata); system.time(Result <- LocASN(countmatrix = as(countsimdata,"sparseMatrix")))
```

## Evaluation signals

- runtime_comparison.csv exists and contains exactly one runtime row per method (6 methods) per dataset size
- Elapsed wall-clock ordering RC/PsiNorm < scran/scKWARN < sctransform/SCnorm holds across the size grid
- SCnorm is marked as failed/not-completed on the largest dataset size rather than given a numeric runtime
- All sizes use the seeded negative-binomial design (set.seed(12345); mu ~ rgamma(G, shape=2, rate=2); size=0.1) so runs are reproducible
- LocASN runs accept both general and as(countsimdata,"sparseMatrix") inputs at every dataset size

## Limitations

- SCnorm can fail (error or non-termination) on large datasets, so its runtime cannot be recorded at every size
- Timing is wall-clock and therefore machine-dependent; the RC/PsiNorm < scran/scKWARN < sctransform/SCnorm ordering is the validated signal, not absolute seconds
- Runtimes are measured on simulated negative-binomial data (mu ~ Gamma(shape=2, rate=2), dispersion size=0.1), which may not reflect performance on real count matrices

## Evidence

- [other] scran and scKWARN required more time than RC and PsiNorm, and sctransform and SCnorm had the longest computational time (with SCnorm failing on some large datasets): [evidence span withheld — anchor status: not_found]
- [readme] LocASN accepts general or sparse matrix input: "### Inputs: countmatrix (general matrix or sparse matrix)"
- [readme] Simulated count matrix generation code from README: "set.seed(12345)
G <- 2000; n <- 600 # G: number of genes, n: number of cells
mu <- rgamma(G, shape = 2, rate = 2)"
- [methods] Installation from GitHub: "Download scKWARN_1.1.2.tar.gz and locally install it, or execute the following code"
