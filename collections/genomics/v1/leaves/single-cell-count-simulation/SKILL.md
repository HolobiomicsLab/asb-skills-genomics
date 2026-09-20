---
name: single-cell-count-simulation
description: Use when you need synthetic scRNA-seq counts with known co-expression
  modules (ground-truth cluster labels) to benchmark correlation metrics or clustering
  pipelines.
when_to_use_negative:
- Input is already real scRNA-seq data with no recoverable ground truth — simulation
  is only needed when benchmarking against known cluster structure
- The goal is downstream biological analysis of actual expression data rather than
  controlled evaluation of co-expression metrics
- You need long-read transcriptome definition or functional annotation — the acorde
  package explicitly excludes these steps
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SymSim
  role: R package used to generate the simulated single-cell count matrix with co-expressed
    gene modules for the benchmark
  repo: https://github.com/YosefLab/SymSim
- name: acorde
  role: Provides the count re-ranking strategy to embed ground-truth co-expression
    modules and the co-expression correlation metric computation on simulated data
  repo: https://github.com/ConesaLab/acorde
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:36ee8acb138bb8e64e61d5888c714a09399d21085e6354961cff4d490ff15a0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - R
  - SymSim
  - YosefLab/SymSim
  - acorde
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# single-cell-count-simulation

## Summary

Simulate single-cell RNA-seq count matrices with known co-expression cluster structure using the SymSim R package, so that co-expression metrics (percentile, Pearson, Spearman, zero-inflated Kendall, rho proportionality) can be benchmarked against ground truth. Used in the acorde study to validate that percentile correlations recover co-expression modules in sparse single-cell data.

## When to use

Use when you need synthetic scRNA-seq counts with known co-expression modules (ground-truth cluster labels) to benchmark correlation metrics or clustering pipelines.

## When NOT to use

- Input is already real scRNA-seq data with no recoverable ground truth — simulation is only needed when benchmarking against known cluster structure
- The goal is downstream biological analysis of actual expression data rather than controlled evaluation of co-expression metrics
- You need long-read transcriptome definition or functional annotation — the acorde package explicitly excludes these steps

## Inputs

- Simulation parameters (R script): number of cells (1000), cell types (8), synthetic features (3000)
- SymSim gene_module_prop setting to induce co-expressed gene modules (smallest module size 10 genes)
- acorde count re-ranking strategy for embedding 15 ground-truth co-expression modules

## Outputs

- Simulated single-cell count matrix with known co-expression cluster labels
- coexpression_metrics_benchmark.csv with per-metric performance
- Cluster Jaccard indices (simulated vs calculated clusters) and proportion of within-cluster metric values > 0.8
- Unclustered feature percentages before and after cluster expansion

## How to apply

In R, install SymSim from GitHub (devtools::install_github("YosefLab/SymSim")) and generate a simulated single-cell count matrix (1000 cells, 8 cell types, 3000 synthetic features) with SymSim::SimulateTrueCounts, using the gene_module_prop argument to induce co-expressed gene modules (minimum module size 10 genes). Embed a ground-truth cluster structure of 15 co-expression modules by applying the acorde count re-ranking strategy to the simulated matrix. Compute the five co-expression correlation metrics (percentile, Pearson, Spearman, zero-inflated Kendall, rho proportionality) on the simulated counts using the acorde package, cluster the correlated features, and evaluate recovery by simulated-vs-calculated cluster Jaccard indices plus the proportion of within-cluster metric values exceeding 0.8. Perform cluster expansion and record unclustered feature percentages before and after expansion; assemble results into coexpression_metrics_benchmark.csv.

## Related tools

- **SymSim** (R package used to generate the simulated single-cell count matrix with co-expressed gene modules for the benchmark) — https://github.com/YosefLab/SymSim
- **acorde** (Provides the count re-ranking strategy to embed ground-truth co-expression modules and the co-expression correlation metric computation on simulated data) — https://github.com/ConesaLab/acorde

## Examples

```
devtools::install_github("YosefLab/SymSim"); sim <- SymSim::SimulateTrueCounts(cells=1000, genes=3000, gene_module_prop=0.5)
```

## Evaluation signals

- Percentile correlation attains 0% unclustered features after cluster expansion (vs Pearson ~1%, rho ~25%)
- High Jaccard agreement between simulated and calculated feature clusters
- High proportion (>0.8 threshold) of within-cluster correlation values for percentile correlation
- Percentile correlation outperforms rho proportionality consistently across all evaluated features
- Benchmark results assembled into a complete coexpression_metrics_benchmark.csv covering all five metrics

## Limitations

- SymSim's smallest co-expression module size is 10 genes, constraining the granularity of embedded ground-truth modules
- Simulated ground truth reflects SymSim's variability model; conclusions may not transfer to all real scRNA-seq sparsity regimes
- acorde's co-expression metrics depend on the dismay R package, which is not on CRAN and must be installed from GitHub (propr may also be needed)

## Evidence

- [readme] SymSim is an R package for simulation of single cell RNA-Seq data: [evidence span withheld — anchor status: external_doc]
- [readme] gene_module_prop generates co-expressed gene modules in SymSim: [evidence span withheld — anchor status: external_doc]
- [abstract] percentile correlations overcome sparsity for accurate co-expression: "we develop and validate percentile correlations, an innovative approach that overcomes data sparsity and yields accurate co-expression estimates from single-cell data"
- [other] SymSim listed as related repository in article: [evidence span withheld — anchor status: not_found]
