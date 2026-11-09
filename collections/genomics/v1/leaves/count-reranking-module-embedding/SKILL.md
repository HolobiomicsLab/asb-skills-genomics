---
name: count-reranking-module-embedding
description: Use when benchmarking co-expression/correlation metrics on simulated
  scRNA-seq data requires synthetic counts with known ground-truth co-expression clusters
  embedded via re-ranking.
when_to_use_negative:
- Data are real experimental scRNA-seq counts with no predefined module structure
  to embed — the re-ranking would destroy genuine biology
- Goal is isoform-level co-usage network construction on real data rather than simulation-based
  metric benchmarking
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: YosefLab/SymSim
  role: R package used to simulate the single-cell count matrix with multiple cell
    populations and optional co-expressed gene modules before re-ranking
  repo: https://github.com/YosefLab/SymSim
- name: acorde
  role: R package providing the count re-ranking strategy and the co-expression correlation
    metrics computed on the re-ranked simulated matrix
  repo: https://github.com/ConesaLab/acorde
- name: R
  role: Environment in which SymSim simulation, acorde re-ranking, correlation computation,
    and clustering are executed
  repo: https://github.com/ConesaLab/acorde
- name: dismay
  role: R package supplying some of the co-expression metrics (e.g., rho proportionality)
    available in acorde for the benchmark comparison
  repo: https://github.com/skinnider/dismay
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:60d9b1edff9948069e6157334b5d5cbd5d1cf1ecaf3a3c96c97444fcffbd5611
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - R
  - YosefLab/SymSim
  - acorde
  - dismay
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# count-reranking-module-embedding

## Summary

Embeds known co-expression module structure into simulated single-cell count data by applying the acorde count re-ranking strategy to SymSim-generated counts, creating ground-truth clusters for benchmarking co-expression metrics such as percentile correlations against Pearson, Spearman, zero-inflated Kendall, and rho proportionality.

## When to use

Use when benchmarking co-expression/correlation metrics on simulated scRNA-seq data requires synthetic counts with known ground-truth co-expression clusters embedded via re-ranking.

## When NOT to use

- Data are real experimental scRNA-seq counts with no predefined module structure to embed — the re-ranking would destroy genuine biology
- Goal is isoform-level co-usage network construction on real data rather than simulation-based metric benchmarking

## Inputs

- SymSim-simulated single-cell count matrix (e.g., 1000 cells, 8 cell types, 3000 synthetic features)
- Module membership assignment specifying the 15 ground-truth co-expression modules

## Outputs

- Re-ranked simulated count matrix with 15 embedded co-expression modules as ground truth
- Ground-truth cluster labels used to compute simulated-vs-calculated cluster Jaccard indices

## How to apply

In R, generate a simulated single-cell count matrix with YosefLab/SymSim using SimulateTrueCounts with approximately 1000 cells, 8 cell types/populations, and 3000 synthetic features, optionally exploiting SymSim's gene_module_prop argument to seed co-expressed gene modules. Then apply the acorde count re-ranking strategy: reorder/rerank the simulated count values so that features belonging to the same designed module share correlated expression profiles, embedding 15 co-expression modules as ground-truth cluster structure. Finally, compute the candidate co-expression metrics on the re-ranked matrix with the acorde package and cluster the correlated features to compare calculated clusters against the simulated modules via Jaccard indices and the proportion of within-cluster metric values exceeding 0.8. The rationale is that re-ranking preserves realistic single-cell count properties (sparsity, zero inflation) while imposing a known correlation structure that traditional metrics on raw simulations cannot be validated against.

## Related tools

- **YosefLab/SymSim** (R package used to simulate the single-cell count matrix with multiple cell populations and optional co-expressed gene modules before re-ranking) — https://github.com/YosefLab/SymSim
- **acorde** (R package providing the count re-ranking strategy and the co-expression correlation metrics computed on the re-ranked simulated matrix) — https://github.com/ConesaLab/acorde
- **R** (Environment in which SymSim simulation, acorde re-ranking, correlation computation, and clustering are executed) — https://github.com/ConesaLab/acorde
- **dismay** (R package supplying some of the co-expression metrics (e.g., rho proportionality) available in acorde for the benchmark comparison) — https://github.com/skinnider/dismay

## Examples

```
library(SymSim); counts <- SimulateTrueCounts(ngenes=3000, ncells=1000, sigma=8, gene_module_prop=0.3); reranked <- acorde::rerank_counts(counts)
```

## Evaluation signals

- Percentile correlation attains 0% unclustered features after cluster expansion on the re-ranked simulated data (vs ~1% for Pearson and ~25% for rho)
- High Jaccard agreement between simulated (ground-truth) and calculated feature clusters
- High proportion of within-cluster metric values exceeding 0.8
- Re-ranked matrix retains realistic single-cell count properties (sparsity, zero inflation) from the original SymSim simulation
- Benchmark results assemble cleanly into coexpression_metrics_benchmark.csv with per-metric comparisons

## Limitations

- Results depend on SymSim simulation parameters (1000 cells, 8 cell types, 3000 features, 15 modules); transferability to other simulation regimes is not established
- Ground truth is imposed by re-ranking rather than arising from a generative co-expression model, so it tests metric recovery of the embedded structure only
- Some acorde co-expression metrics rely on the dismay R package, which is not on CRAN and must be installed from GitHub

## Evidence

- [abstract] percentile correlations overcome data sparsity and yield accurate co-expression estimates from single-cell data: "we develop and validate percentile correlations, an innovative approach that overcomes data sparsity and yields accurate co-expression estimates from single-cell data"
- [readme] acorde computes co-expression relationships using percentile correlations and clustering: "Detection of isoform co-expression relationships using percentile correlations and semi-automated clustering."
- [readme] SymSim can generate co-expressed gene modules via gene_module_prop: [evidence span withheld — anchor status: external_doc]
- [readme] acorde metrics depend on the dismay package, install from GitHub: [evidence span withheld — anchor status: external_doc]
