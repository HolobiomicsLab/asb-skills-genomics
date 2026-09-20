---
name: cluster-recovery-jaccard-quantification
description: Use when evaluating whether a co-expression/correlation metric accurately
  recovers ground-truth feature cluster structure embedded in simulated single-cell
  data, rather than relying on visual or qualitative assessment.
when_to_use_negative:
- Input data has no known ground-truth cluster structure — Jaccard agreement against
  simulated modules cannot be computed on real data
- The goal is isoform functional annotation or DIU/coDIU detection rather than benchmarking
  co-expression metric accuracy
- Evaluation target is differential expression performance rather than co-expression
  cluster recovery
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
tools:
- name: acorde
  role: R package used to apply count re-ranking, compute the five co-expression correlation
    metrics, cluster correlated features, and perform cluster expansion on simulated
    data
  repo: https://github.com/ConesaLab/acorde
- name: SymSim
  role: R package used to generate the simulated single-cell count matrix with known
    co-expression gene modules serving as ground-truth clusters
  repo: https://github.com/YosefLab/SymSim
- name: R
  role: Implementation environment for the simulation, metric computation, clustering,
    and Jaccard quantification workflow
  repo: https://github.com/ConesaLab/acorde
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:2e9af59905505d03b532ffa9b4de238c3df23ff8366d55239d52c252d29195a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - R
  - SymSim
  - YosefLab/SymSim
  - acorde
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cluster-recovery-jaccard-quantification

## Summary

A benchmarking procedure that quantifies how well co-expression correlation metrics (percentile, Pearson, Spearman, zero-inflated Kendall, rho proportionality) recover known simulated cluster structure, using Jaccard indices between simulated and calculated clusters plus the proportion of within-cluster metric values exceeding 0.8.

## When to use

Use when evaluating whether a co-expression/correlation metric accurately recovers ground-truth feature cluster structure embedded in simulated single-cell data, rather than relying on visual or qualitative assessment.

## When NOT to use

- Input data has no known ground-truth cluster structure — Jaccard agreement against simulated modules cannot be computed on real data
- The goal is isoform functional annotation or DIU/coDIU detection rather than benchmarking co-expression metric accuracy
- Evaluation target is differential expression performance rather than co-expression cluster recovery

## Inputs

- Simulated single-cell count matrix from SymSim (e.g. 1000 cells, 8 cell types, 3000 features)
- Ground-truth co-expression module/cluster assignments embedded via acorde count re-ranking (15 modules)
- Co-expression correlation matrices computed per metric (percentile, Pearson, Spearman, zero-inflated Kendall, rho proportionality)

## Outputs

- Simulated-vs-calculated cluster Jaccard indices per metric
- Proportion of within-cluster metric values exceeding 0.8, per metric
- Percentages of unclustered features before and after cluster expansion, per metric
- Benchmark summary table (coexpression_metrics_benchmark.csv)

## How to apply

Generate a simulated single-cell count matrix with SymSim (1000 cells, 8 cell types, 3000 synthetic features) and apply the acorde count re-ranking strategy to embed ~15 co-expression modules as ground-truth cluster structure. Compute the candidate co-expression metrics on the simulated matrix using the acorde package, then cluster the correlated features. Calculate Jaccard indices between simulated and calculated clusters, and compute the proportion of within-cluster metric values exceeding 0.8, to quantify cluster recovery per metric. Additionally perform cluster expansion and record the percentage of unclustered features before and after expansion — for percentile correlation, 0% unclustered after expansion is the acceptance criterion, compared to ~1% for Pearson and ~25% for rho proportionality. Assemble per-metric results (e.g. into coexpression_metrics_benchmark.csv) to enable direct comparison of recovery quality.

## Related tools

- **acorde** (R package used to apply count re-ranking, compute the five co-expression correlation metrics, cluster correlated features, and perform cluster expansion on simulated data) — https://github.com/ConesaLab/acorde
- **SymSim** (R package used to generate the simulated single-cell count matrix with known co-expression gene modules serving as ground-truth clusters) — https://github.com/YosefLab/SymSim
- **R** (Implementation environment for the simulation, metric computation, clustering, and Jaccard quantification workflow) — https://github.com/ConesaLab/acorde

## Examples

```
devtools::install_github("YosefLab/SymSim"); devtools::install_github("ConesaLab/acorde"); sim <- SimulateTrueCounts(ncells=1000, nGeneTypes=3000, gene_module_prop=0.15)
```

## Evaluation signals

- Percentile correlation attains 0% unclustered features after cluster expansion (acceptance criterion)
- High Jaccard agreement between simulated and calculated clusters for the evaluated metric
- High proportion of within-cluster metric values exceeding the 0.8 threshold
- Pearson shows ~1% unclustered features after expansion while rho proportionality shows ~25%, providing reference ranges for comparison
- Benchmark results assemble cleanly into coexpression_metrics_benchmark.csv with one entry per metric

## Limitations

- Ground-truth evaluation is only possible on simulated data; real single-cell data lacks known co-expression cluster assignments
- Rho proportionality performed less consistently than percentile correlation in all evaluated features, illustrating that proportionality metrics may under-recover sparse single-cell co-expression structure
- Traditional (Pearson, Spearman) and zero-inflation-aware (zero-inflated Kendall) metrics were all outperformed by percentile correlations for sparse single-cell data, so metric choice matters for the validity of downstream clustering

## Evidence

- [abstract] Percentile correlations outperform traditional and zero-inflation-aware metrics on simulated clusters: "we develop and validate percentile correlations, an innovative approach that overcomes data sparsity and yields accurate co-expression estimates from single-cell data"
- [intro] Metric computation and clustering are performed in the acorde R package: "This pipeline has been implemented in the R package acorde (https://github.com/ConesaLab/acorde)."
- [readme] SymSim simulates single-cell data with ground-truth gene modules: [evidence span withheld — anchor status: external_doc]
- [abstract] Co-expression clustering into a network is a core acorde workflow step: "acorde uses correlations to cluster co-expressed isoforms into a network, unraveling cell type-specific alternative isoform usage patterns"
- [readme] dismay provides some co-expression metrics available in acorde: [evidence span withheld — anchor status: external_doc]
