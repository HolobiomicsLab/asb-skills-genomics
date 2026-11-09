---
name: correlation-matrix-cluster-filtering
description: Use when a single-cell isoform-level expression matrix has been converted
  to a percentile correlation matrix and the goal is to partition all isoforms into
  co-expressed modules across cell types.
when_to_use_negative:
- When co-expression relationships are already known or the input is a pre-computed
  isoform clustering rather than a correlation matrix
- When the analysis target is gene-level (not isoform-level) co-expression from bulk
  data — acorde's percentile correlations are designed for sparse single-cell isoform
  expression
- When the goal is functional annotation of isoforms rather than co-expression module
  detection (that step requires tappAS/IsoAnnotLite and is separate)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: acorde
  role: R package implementing the semi-automated isoform co-expression clustering
    pipeline, including percentile correlation computation and module detection
  repo: https://github.com/ConesaLab/acorde
- name: dynamicTreeCut
  role: Initializes module detection on the correlation matrix (deepSplit=4, pamStage=FALSE,
    minClusterSize=20) before filtering and merging
- name: R
  role: Execution environment for the acorde pipeline and clustering functions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:8d33a91b718d5f370193bc311a6a739027875738fe024e9dd649b450542ba847
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - R
  - acorde
  - dynamicTreeCut
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# correlation-matrix-cluster-filtering

## Summary

Semi-automated clustering of single-cell isoform percentile correlation matrices into co-expression modules, combining dynamicTreeCut initialization with correlation-based filtering, metatranscript-based cluster merging, and recursive assignment of unclustered isoforms.

## When to use

Use when a single-cell isoform-level expression matrix has been converted to a percentile correlation matrix and the goal is to partition all isoforms into co-expressed modules across cell types.

## When NOT to use

- When co-expression relationships are already known or the input is a pre-computed isoform clustering rather than a correlation matrix
- When the analysis target is gene-level (not isoform-level) co-expression from bulk data — acorde's percentile correlations are designed for sparse single-cell isoform expression
- When the goal is functional annotation of isoforms rather than co-expression module detection (that step requires tappAS/IsoAnnotLite and is separate)

## Inputs

- Percentile correlation matrix computed from single-cell isoform expression data
- Isoform-level single-cell expression matrix
- Cell type/state identity labels

## Outputs

- Isoform cluster assignments table (isoform_cluster_assignments.tsv)
- Partition of isoforms into co-expression modules (e.g., 15 clusters covering all analyzed isoforms)

## How to apply

Load the percentile correlation matrix computed from the single-cell isoform expression data (percentile correlations are used specifically because they overcome scRNA-seq data sparsity). Initialize clustering with dynamicTreeCut on the correlation matrix using deepSplit=4, pamStage=FALSE, and minClusterSize=20. Apply correlation-based filtering of the detected modules, then merge redundant clusters via metatranscript-based correlation merging (including manually curated merges where profiles are redundant). Finally, recursively assign any unclustered isoforms by expansion at decreasing correlation thresholds (0.9, then 0.8, then 0.7) to the nearest matching cluster, and write final assignments to isoform_cluster_assignments.tsv. In the reference application, this pipeline yielded 15 distinct clusters covering all 6794 analyzed isoforms across 7 broad cell types of the mouse neural dataset.

## Related tools

- **acorde** (R package implementing the semi-automated isoform co-expression clustering pipeline, including percentile correlation computation and module detection) — https://github.com/ConesaLab/acorde
- **dynamicTreeCut** (Initializes module detection on the correlation matrix (deepSplit=4, pamStage=FALSE, minClusterSize=20) before filtering and merging)
- **R** (Execution environment for the acorde pipeline and clustering functions)

## Examples

```
modules <- cutreeDynamic(dendro = hclust(as.dist(1 - pct_cor_matrix)), deepSplit = 4, pamStage = FALSE, minClusterSize = 20)
```

## Evaluation signals

- Final partition contains exactly 15 distinct clusters
- All 6794 analyzed isoforms receive a cluster assignment (no unclustered isoforms remain after recursive expansion)
- Every module satisfies the dynamicTreeCut minimum size of 20 isoforms
- Redundant cluster profiles are consolidated via metatranscript-based correlation merging, including the two manually merged cluster pairs
- Clusters represent diverse expression modalities reproducible across the 7 broad cell types

## Limitations

- Percentile correlations are required because raw correlation estimates from sparse single-cell data are unreliable; standard Pearson/Spearman on sparse isoform counts is not a substitute
- Clustering is semi-automated: metatranscript merging includes manual cluster pair merges, so full reproduction requires the documented manual merge decisions
- Mechanisms generating the resulting isoform co-expression remain speculative and require further experimental validation

## Evidence

- [abstract] percentile correlations overcome sparsity for accurate co-expression: "we develop and validate percentile correlations, an innovative approach that overcomes data sparsity and yields accurate co-expression estimates from single-cell data"
- [abstract] clustering co-expressed isoforms into a network: "acorde uses correlations to cluster co-expressed isoforms into a network, unraveling cell type-specific alternative isoform usage patterns"
- [intro] semi-automated clustering of co-expressed isoform modules: "a semi-automated clustering approach to detect modules of co-expressed isoforms across cell types"
- [readme] pipeline block implemented in acorde: "Detection of isoform co-expression relationships using percentile correlations and semi-automated clustering."
