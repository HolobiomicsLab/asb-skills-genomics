---
name: isoform-co-expression-clustering
description: Use when you have a filtered single-cell isoform-level expression matrix
  and need noise-robust isoform-to-isoform co-expression estimates grouped into co-usage
  modules across cell types, prior to DIU/coDIU analysis.
when_to_use_negative:
- Input is gene-level (not isoform-level) count data — the pipeline requires an isoform-level
  quantification matrix
- The goal is bulk-sample isoform co-expression without cell-type structure — percentile
  correlations are designed for sparse single-cell data summarized per cell type
- Only two conditions/groups with no multiple cell types — DIU/coDIU analyses are
  designed to leverage multiple cell types in single-cell data
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: acorde
  role: R package implementing percentile correlations, semi-automated isoform clustering,
    and DIU/coDIU analysis; provides example dataset as internal package data
  repo: https://github.com/ConesaLab/acorde
- name: dynamicTreeCut
  role: R package used to initialize isoform clustering on the percentile correlation
    matrix (deepSplit=4, pamStage=FALSE, minClusterSize=20)
- name: dismay
  role: R package providing co-expression metrics on which some of acorde's co-expression
    functions are based; must be installed from GitHub before acorde
  repo: https://github.com/skinnider/dismay
provenance:
  source_task_ids:
  - task_008
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:526cc85e6955e0e8ebdf3a3727a1d32a46483be15f7cbbf7100706779e1ae686
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - BiocStyle
  - R
  - acorde
  - acorde (R package)
  - dismay
  - dynamicTreeCut
  - knitr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# isoform-co-expression-clustering

## Summary

Semi-automated clustering of isoform co-expression relationships from sparse single-cell data, using acorde's percentile correlations followed by dynamicTreeCut initialization, correlation-based filtering, metatranscript-based cluster merging, and recursive assignment of unclustered isoforms.

## When to use

Use when you have a filtered single-cell isoform-level expression matrix and need noise-robust isoform-to-isoform co-expression estimates grouped into co-usage modules across cell types, prior to DIU/coDIU analysis.

## When NOT to use

- Input is gene-level (not isoform-level) count data — the pipeline requires an isoform-level quantification matrix
- The goal is bulk-sample isoform co-expression without cell-type structure — percentile correlations are designed for sparse single-cell data summarized per cell type
- Only two conditions/groups with no multiple cell types — DIU/coDIU analyses are designed to leverage multiple cell types in single-cell data

## Inputs

- Filtered single-cell isoform-level expression/quantification matrix (e.g. 6794 isoforms x cells from the acorde vignette)
- Isoform-to-gene relationship table
- Cell-level identity labels (cell type or state annotations, e.g. 7 broad cell types or the 5 GABA-ergic subtypes Lamp5, Pvalb, Sncg, Sst, Vip)

## Outputs

- Symmetric percentile isoform-to-isoform correlation matrix (e.g. 6794 x 6794, unit diagonal), saved as percentile_correlation_matrix.rds
- Final isoform cluster assignments (e.g. isoform_cluster_assignments.tsv) partitioning all analyzed isoforms into co-expressed modules
- Merged isoform expression profiles/metatranscripts per cluster used as input to DIU and coDIU analysis

## How to apply

First replace sparse cell-level counts with cell-type-level percentile profiles: for each isoform, compute the p=10 percentile value of counts within each cell type, then compute pairwise Pearson correlations across these percentile-summarized profiles to obtain a symmetric isoform-to-isoform correlation matrix with unit diagonal. Initialize clustering with dynamicTreeCut (deepSplit=4, pamStage=FALSE, minClusterSize=20) on the correlation matrix, then filter detected modules by correlation and merge redundant clusters via metatranscript-based correlation merging (including any manual merges flagged during inspection). Finally, recursively assign remaining unclustered isoforms to the nearest matching cluster at decreasing correlation thresholds (0.9, then 0.8, then 0.7). Rationale: percentile correlations overcome single-cell sparsity by capturing each isoform's expression-distribution signature per cell type, yielding co-expression estimates that better reflect isoform co-usage than cell-level Pearson correlation. In the acorde mouse neural dataset this pipeline yielded 15 clusters covering all 6794 isoforms; on the five Tasic 2018 GABA-ergic subtypes (Lamp5, Pvalb, Sncg, Sst, Vip) the merged profiles reduced to 5 clusters used for downstream coDIU detection.

## Related tools

- **acorde** (R package implementing percentile correlations, semi-automated isoform clustering, and DIU/coDIU analysis; provides example dataset as internal package data) — https://github.com/ConesaLab/acorde
- **dynamicTreeCut** (R package used to initialize isoform clustering on the percentile correlation matrix (deepSplit=4, pamStage=FALSE, minClusterSize=20))
- **dismay** (R package providing co-expression metrics on which some of acorde's co-expression functions are based; must be installed from GitHub before acorde) — https://github.com/skinnider/dismay

## Examples

```
install.packages("devtools"); devtools::install_github("skinnider/dismay"); devtools::install_github("ConesaLab/acorde", build_vignettes = TRUE)
```

## Evaluation signals

- Correlation matrix is symmetric with dimensions 6794 x 6794, unit diagonal, and valid as clustering input
- Final partition assigns all 6794 analyzed isoforms with no unclustered remainder
- Reproduction of the mouse neural dataset yields exactly 15 distinct clusters
- On the Tasic 2018 GABA-ergic subtypes the pipeline produces 5 merged isoform profiles, and downstream DIU/coDIU analysis recovers 22 DIU genes, 16 coDIU genes involving 36 isoforms
- DE filtering upstream yields 568 significant DE isoforms across the five GABA-ergic subtypes before clustering-based coDIU analysis

## Limitations

- Some acorde co-expression metrics depend on the dismay R package, which is not on CRAN and must be installed from GitHub (plus propr if required)
- Functional analysis of the resulting isoform clusters requires a functionally-annotated transcriptome (via isoAnnotLite and tappAS); acorde's package itself does not include long-read transcriptome building or functional analysis
- Percentile correlation requires cell-type labels; isoform clustering quality depends on pre-defined cell type/state annotations
- Manual cluster merges were needed in the published run (two manually merged cluster pairs), so fully automated merging may leave redundant clusters

## Evidence

- [abstract] percentile correlations overcome data sparsity and yield accurate co-expression estimates from single-cell data: "we develop and validate percentile correlations, an innovative approach that overcomes data sparsity and yields accurate co-expression estimates from single-cell data"
- [intro] clustering workflow definition: "a semi-automated clustering approach to detect modules of co-expressed isoforms across cell types"
- [results] dynamicTreeCut parameters: [evidence span withheld — anchor status: not_found]
- [results] recursive expansion thresholds: [evidence span withheld — anchor status: not_found]
- [readme] installation from GitHub: "devtools::install_github("ConesaLab/acorde")"
- [readme] dismay dependency: "Some of the co-expression metrics available in *acorde* are based on 
functions from the [dismay](https://github.com/skinnider/dismay) R package."
