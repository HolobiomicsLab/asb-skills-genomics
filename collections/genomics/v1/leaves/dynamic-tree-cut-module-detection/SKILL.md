---
name: dynamic-tree-cut-module-detection
description: Use when you have a single-cell isoform expression matrix and a percentile
  correlation matrix, and need to identify modules of co-expressed isoforms across
  cell types without hard-coding a cluster count.
when_to_use_negative:
- You have gene-level (not isoform-level) expression data and want conventional gene
  co-expression modules rather than isoform co-usage modules
- You need a fully automated, parameter-free clustering — this pipeline is explicitly
  semi-automated and includes manual cluster merges
- Your goal is differential isoform usage (DIU/coDIU) detection itself rather than
  the upstream module detection that feeds it
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: dynamicTreeCut
  role: Hybrid tree-cut algorithm used to initialize module detection on the percentile
    correlation matrix (deepSplit=4, pamStage=FALSE, minClusterSize=20)
- name: R
  role: Implementation environment for the clustering pipeline, packaged in acorde
  repo: https://github.com/ConesaLab/acorde
- name: acorde
  role: R package supplying the semi-automated clustering, metatranscript merging,
    and unclustered-isoform expansion steps on top of dynamicTreeCut
  repo: https://github.com/ConesaLab/acorde
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:8a0e870757bf49771a90d3f2036fb07084fae8483c8b3600ce4e60b77626c5b8
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# dynamic-tree-cut-module-detection

## Summary

Semi-automated detection of isoform co-expression modules by running the dynamicTreeCut hybrid algorithm on a percentile correlation matrix of single-cell isoform expression, followed by correlation-based filtering, metatranscript-based cluster merging, and recursive assignment of unclustered isoforms. Used by acorde to partition mouse neural isoforms into cell type-specific co-usage modules.

## When to use

Use when you have a single-cell isoform expression matrix and a percentile correlation matrix, and need to identify modules of co-expressed isoforms across cell types without hard-coding a cluster count.

## When NOT to use

- You have gene-level (not isoform-level) expression data and want conventional gene co-expression modules rather than isoform co-usage modules
- You need a fully automated, parameter-free clustering — this pipeline is explicitly semi-automated and includes manual cluster merges
- Your goal is differential isoform usage (DIU/coDIU) detection itself rather than the upstream module detection that feeds it

## Inputs

- Percentile correlation matrix computed from single-cell isoform expression data
- Isoform-level quantification matrix with isoform-to-gene relationships
- Cell-level identity labels (cell type or state)

## Outputs

- Cluster assignments per isoform (e.g. isoform_cluster_assignments.tsv)
- Set of co-expressed isoform modules/metatranscripts used for downstream DIU/coDIU analysis

## How to apply

Initialize clustering with dynamicTreeCut's hybrid method on the percentile correlation matrix using deepSplit=4, pamStage=FALSE, and minClusterSize=20, so that fine-grained, cell-type-resolution modules are recovered while sparse isoforms are not forced into PAM-assigned clusters. Next, apply correlation-based filtering of the detected modules and merge clusters whose metatranscripts (cluster centroid expression profiles) are highly correlated, including any manually reviewed merge pairs, to remove redundant profiles. Then recursively assign remaining unclustered isoforms to the nearest matching cluster, expanding at progressively decreasing correlation thresholds (0.9, then 0.8, then 0.7). Write final assignments (e.g. to isoform_cluster_assignments.tsv) and validate the partition — in the acorde mouse neural dataset this yielded exactly 15 clusters covering all 6794 analyzed isoforms across 7 broad cell types.

## Related tools

- **dynamicTreeCut** (Hybrid tree-cut algorithm used to initialize module detection on the percentile correlation matrix (deepSplit=4, pamStage=FALSE, minClusterSize=20))
- **R** (Implementation environment for the clustering pipeline, packaged in acorde) — https://github.com/ConesaLab/acorde
- **acorde** (R package supplying the semi-automated clustering, metatranscript merging, and unclustered-isoform expansion steps on top of dynamicTreeCut) — https://github.com/ConesaLab/acorde

## Examples

```
clusters <- dynamicTreeCut::cutreeDynamic(dendro = hclust(as.dist(1 - pcor)), method = "hybrid", deepSplit = 4, pamStage = FALSE, minClusterSize = 20)
```

## Evaluation signals

- Final partition contains exactly 15 distinct clusters (expected for the mouse neural dataset)
- All 6794 analyzed isoforms are assigned to a cluster — no isoform left unclustered after the 0.9/0.8/0.7 threshold expansion
- Merged clusters are non-redundant: metatranscript correlations between final cluster centroids should not indicate remaining duplicate profiles
- Assignments are written to isoform_cluster_assignments.tsv with one label per input isoform
- Clusters span diverse expression modalities across the 7 broad cell types in the dataset

## Limitations

- Pipeline is semi-automated: two cluster pairs required manual merging decisions in the published run
- minClusterSize=20 excludes small modules; isoforms below this granularity are only recovered via the threshold-based expansion
- Percentile correlations are designed to overcome single-cell sparsity, but the correlation-based expansion is threshold-sensitive (0.9→0.8→0.7 ordering matters)

## Evidence

- [intro] Semi-automated clustering to detect modules of co-expressed isoforms across cell types: "a semi-automated clustering approach to detect modules of co-expressed isoforms across cell types"
- [abstract] Percentile correlations motivate the input matrix: "we develop and validate percentile correlations, an innovative approach that overcomes data sparsity and yields accurate co-expression estimates from single-cell data"
- [abstract] Clustering output feeds cell-type-specific interpretation: "acorde uses correlations to cluster co-expressed isoforms into a network, unraveling cell type-specific alternative isoform usage patterns"
- [readme] README positions clustering block in the pipeline: "Detection of isoform co-expression relationships using percentile correlations and semi-automated clustering."
- [readme] README confirms package provides study's expression matrix: [evidence span withheld — anchor status: external_doc]
