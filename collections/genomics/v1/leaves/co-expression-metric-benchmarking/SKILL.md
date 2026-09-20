---
name: co-expression-metric-benchmarking
description: Use when sparse scRNA-seq count data (or simulated counts) must be evaluated
  for which co-expression metric (percentile vs Pearson/Spearman/Kendall/rho) recovers
  known co-expression clusters most accurately.
when_to_use_negative:
- You have no ground-truth (or simulated) cluster structure — cluster-recovery Jaccard
  and unclustered-percentage criteria cannot be evaluated on real unlabelled data
- Input is bulk RNA-seq or dense/high-count expression data — the benchmark targets
  the sparsity regime that motivates percentile correlations
- You already have correlation estimates and only need downstream isoform/gene clustering
  or coDIU analysis rather than metric comparison
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: acorde
  role: R package implementing percentile correlations and other co-expression metrics
    (incl. dismay-based ones), clustering, and the count re-ranking strategy
  repo: https://github.com/ConesaLab/acorde
- name: SymSim
  role: Simulates single-cell RNA-seq count matrices with embedded co-expressed gene
    modules (SimulateTrueCounts gene_module_prop argument) to create benchmark ground
    truth
  repo: https://github.com/YosefLab/SymSim
- name: dismay
  role: R package providing the co-expression metric functions (e.g. rho proportionality)
    that acorde re-uses; install from GitHub since it is not on CRAN
  repo: https://github.com/skinnider/dismay
- name: R
  role: Runtime environment for SymSim simulation, acorde metric computation, clustering,
    and Jaccard benchmarking
  repo: https://github.com/ConesaLab/acorde
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:7d471bbee4d8a84e4ebfd653921a5494e3e9457f7de1277640359bc99e8e1e73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - R
  - SymSim
  - YosefLab/SymSim
  - acorde
  - dismay
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# co-expression-metric-benchmarking

## Summary

Benchmark co-expression correlation metrics on simulated single-cell data with known co-expression modules, comparing percentile correlation against Pearson, Spearman, zero-inflated Kendall, and rho proportionality via cluster-recovery evaluation.

## When to use

Use when sparse scRNA-seq count data (or simulated counts) must be evaluated for which co-expression metric (percentile vs Pearson/Spearman/Kendall/rho) recovers known co-expression clusters most accurately.

## When NOT to use

- You have no ground-truth (or simulated) cluster structure — cluster-recovery Jaccard and unclustered-percentage criteria cannot be evaluated on real unlabelled data
- Input is bulk RNA-seq or dense/high-count expression data — the benchmark targets the sparsity regime that motivates percentile correlations
- You already have correlation estimates and only need downstream isoform/gene clustering or coDIU analysis rather than metric comparison

## Inputs

- Simulated single-cell count matrix generated with SymSim (YosefLab/SymSim), e.g. 1000 cells × 3000 features across 8 cell types
- Ground-truth co-expression cluster structure embedded via SymSim gene modules and acorde count re-ranking (15 modules)
- acorde R package with co-expression metric implementations (incl. dismay-based metrics)

## Outputs

- coexpression_metrics_benchmark.csv with per-metric performance results
- Per-metric Jaccard indices between simulated and calculated clusters
- Proportion of within-cluster metric values >0.8 per metric
- Unclustered-feature percentages per metric before and after cluster expansion

## How to apply

In R, simulate a single-cell count matrix with SymSim (e.g. 1000 cells, 8 cell types, 3000 synthetic features), using the SimulateTrueCounts() gene_module_prop argument to embed co-expressed gene modules as ground truth, then apply acorde's count re-ranking strategy to embed 15 co-expression modules. Compute the five correlation metrics (percentile, Pearson, Spearman, zero-inflated Kendall, rho proportionality — the latter via functions from the dismay package, which acorde depends on) on the simulated matrix with the acorde R package. Cluster the correlated features and quantify simulated-vs-calculated cluster agreement with Jaccard indices, plus the proportion of within-cluster metric values exceeding 0.8. Perform cluster expansion and record the percentage of unclustered features before and after: acceptance requires percentile correlation to reach 0% unclustered features after expansion (vs ~1% for Pearson and ~25% for rho), with high Jaccard agreement. Assemble results into coexpression_metrics_benchmark.csv.

## Related tools

- **acorde** (R package implementing percentile correlations and other co-expression metrics (incl. dismay-based ones), clustering, and the count re-ranking strategy) — https://github.com/ConesaLab/acorde
- **SymSim** (Simulates single-cell RNA-seq count matrices with embedded co-expressed gene modules (SimulateTrueCounts gene_module_prop argument) to create benchmark ground truth) — https://github.com/YosefLab/SymSim
- **dismay** (R package providing the co-expression metric functions (e.g. rho proportionality) that acorde re-uses; install from GitHub since it is not on CRAN) — https://github.com/skinnider/dismay
- **R** (Runtime environment for SymSim simulation, acorde metric computation, clustering, and Jaccard benchmarking) — https://github.com/ConesaLab/acorde

## Examples

```
devtools::install_github("skinnider/dismay"); devtools::install_github("YosefLab/SymSim"); devtools::install_github("ConesaLab/acorde")
```

## Evaluation signals

- Percentile correlation attains 0% unclustered features after cluster expansion (acceptance criterion; Pearson ~1%, rho ~25%)
- High Jaccard index agreement between simulated (ground-truth) and calculated co-expression clusters
- Proportion of within-cluster metric values exceeding 0.8 is high for the winning metric
- Benchmark results assembled into coexpression_metrics_benchmark.csv with all five metrics reported
- Simulated matrix reproducibly contains 15 embedded co-expression modules after acorde re-ranking

## Limitations

- Benchmark performance depends on simulation parameters; SymSim gene modules have a minimum size of 10 genes
- Rho proportionality leaves ~25% of features unclustered after expansion, limiting its recovery power on sparse data
- Dismay-based metrics require manual GitHub installation (dismay and propr are not on CRAN), which can complicate reproducibility

## Evidence

- [abstract] Percentile correlations beat sparse-data alternatives: "we develop and validate percentile correlations, an innovative approach that overcomes data sparsity and yields accurate co-expression estimates from single-cell data"
- [intro] Noise-robust correlation strategy in scRNA-seq: "we developed a strategy to obtain noise-robust correlation estimates in scRNA-seq data"
- [readme] Co-expression metrics depend on dismay: [evidence span withheld — anchor status: external_doc]
- [readme] SymSim embeds co-expressed gene modules: [evidence span withheld — anchor status: external_doc]
