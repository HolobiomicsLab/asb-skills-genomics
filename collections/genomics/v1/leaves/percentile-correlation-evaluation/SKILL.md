---
name: percentile-correlation-evaluation
description: Use when evaluating or validating co-expression metrics for sparse single-cell
  (scRNA-seq) data, or when deciding which correlation metric to feed into isoform/gene
  module clustering.
when_to_use_negative:
- Input is bulk RNA-seq or otherwise non-sparse expression data, where data sparsity
  — the rationale for percentile summarization — is not an issue
- You need a fully automated clustering procedure; acorde's downstream pipeline is
  semi-automated and includes manual cluster merges
- The goal is differential isoform usage (DIU/coDIU) testing or functional annotation
  rather than comparing co-expression correlation metrics
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: acorde
  role: R package implementing percentile correlations, the count re-ranking strategy,
    and all compared co-expression metrics plus clustering
  repo: https://github.com/ConesaLab/acorde
- name: YosefLab/SymSim
  role: Simulates single-cell RNA-seq count matrices with known co-expressed gene
    modules to provide ground truth for the benchmark
  repo: https://github.com/YosefLab/SymSim
- name: dismay
  role: Provides some of the co-expression metrics available in acorde (e.g. zero-inflated
    Kendall, rho proportionality); must be installed from GitHub
  repo: https://github.com/skinnider/dismay
- name: R
  role: Environment in which the simulation, metric computation, clustering, and benchmarking
    are executed
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:85d160f07a2fafd5f53943c17be47da924e5f611f10387b3dbe0ad9ef76ee50f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - R
  - YosefLab/SymSim
  - acorde
  - dismay
  - dynamicTreeCut
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# percentile-correlation-evaluation

## Summary

Benchmark percentile correlations against extant co-expression metrics (Pearson, Spearman, zero-inflated Kendall, rho proportionality) using acorde on simulated SymSim single-cell data with known co-expression modules, to verify that percentile correlations overcome sparsity and yield accurate co-expression estimates.

## When to use

Use when evaluating or validating co-expression metrics for sparse single-cell (scRNA-seq) data, or when deciding which correlation metric to feed into isoform/gene module clustering.

## When NOT to use

- Input is bulk RNA-seq or otherwise non-sparse expression data, where data sparsity — the rationale for percentile summarization — is not an issue
- You need a fully automated clustering procedure; acorde's downstream pipeline is semi-automated and includes manual cluster merges
- The goal is differential isoform usage (DIU/coDIU) testing or functional annotation rather than comparing co-expression correlation metrics

## Inputs

- Simulated single-cell count matrix generated with SymSim (e.g. 1000 cells, 8 cell types, 3000 synthetic features) with 15 embedded co-expression modules
- Cell-type identity labels for cells (needed to build percentile/decile profiles)
- Optionally, a filtered real isoform expression matrix (e.g. 6794 isoforms, 7 cell types, Tasic mouse visual cortex Smart-Seq2) for validating the percentile correlation computation itself

## Outputs

- coexpression_metrics_benchmark.csv comparing percentile, Pearson, Spearman, zero-inflated Kendall, and rho proportionality
- Per-metric Jaccard indices between simulated and calculated clusters
- Per-metric proportion of within-cluster metric values > 0.8 and unclustered feature percentages before/after cluster expansion
- Symmetric percentile correlation matrix (e.g. 6794 x 6794, unit diagonal) suitable as clustering input

## How to apply

In R, simulate a single-cell count matrix with YosefLab/SymSim (e.g. 1000 cells, 8 cell types, 3000 features) and embed 15 ground-truth co-expression modules using acorde's count re-ranking strategy. Compute all five metrics — percentile, Pearson, Spearman, zero-inflated Kendall, and rho proportionality — on the same simulated matrix via acorde (rho and zero-inflated Kendall come from the dismay package, which must be installed from GitHub since it is not on CRAN). For each metric, cluster features and score the partition against the simulated ground-truth clusters using Jaccard indices, plus the proportion of within-cluster metric values exceeding 0.8. Run cluster expansion at decreasing correlation thresholds (0.9, then 0.8, then 0.7) and record the unclustered feature percentage before and after; acceptance is 0% unclustered for percentile correlation versus ~1% for Pearson and ~25% for rho. Assemble per-metric results into a coexpression_metrics_benchmark.csv for comparison.

## Related tools

- **acorde** (R package implementing percentile correlations, the count re-ranking strategy, and all compared co-expression metrics plus clustering) — https://github.com/ConesaLab/acorde
- **YosefLab/SymSim** (Simulates single-cell RNA-seq count matrices with known co-expressed gene modules to provide ground truth for the benchmark) — https://github.com/YosefLab/SymSim
- **dismay** (Provides some of the co-expression metrics available in acorde (e.g. zero-inflated Kendall, rho proportionality); must be installed from GitHub) — https://github.com/skinnider/dismay
- **R** (Environment in which the simulation, metric computation, clustering, and benchmarking are executed)

## Examples

```
devtools::install_github("skinnider/dismay"); devtools::install_github("YosefLab/SymSim"); devtools::install_github("ConesaLab/acorde"); library(acorde)
```

## Evaluation signals

- Percentile correlation attains 0% unclustered features after cluster expansion (thresholds 0.9, 0.8, 0.7), versus ~1% for Pearson and ~25% for rho proportionality
- High Jaccard agreement between simulated and calculated feature clusters across all 15 embedded modules
- High proportion (>0.8 threshold) of within-cluster metric values for the percentile metric, outperforming rho in all evaluated features
- Correlation matrix is symmetric with unit diagonal and correct dimensions (e.g. 6794 x 6794 on the Tasic isoform data) and usable as clustering input

## Limitations

- Percentile correlation summarizes expression by deciles within each cell type, so it requires reliable cell-type labels and loses single-cell resolution
- Benchmark conclusions rest on SymSim-simulated data with embedded module structure; real-data ground truth is unavailable
- Some compared metrics (dismay-based) require manual GitHub installation and are not on CRAN, complicating reproducibility

## Evidence

- [abstract] Percentile correlations overcome sparsity in scRNA-seq data: "we develop and validate percentile correlations, an innovative approach that overcomes data sparsity and yields accurate co-expression estimates"
- [readme] README: acorde uses percentile correlations for co-expression detection: "Detection of isoform co-expression relationships using percentile correlations
and semi-automated clustering."
- [readme] README: dismay dependency for some co-expression metrics: "Some of the co-expression metrics available in *acorde* are based on 
functions from the [dismay](https://github.com/skinnider/dismay) R package."
- [readme] SymSim README: R package for scRNA-seq simulation with gene modules: [evidence span withheld — anchor status: external_doc]
