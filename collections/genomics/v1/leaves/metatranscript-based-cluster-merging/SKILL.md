---
name: metatranscript-based-cluster-merging
description: Use when a dynamicTreeCut-initialized partition of isoform co-expression
  clusters contains redundant or highly correlated modules that must be consolidated
  before downstream coDIU or functional analysis.
when_to_use_negative:
- Input is a gene-level expression matrix or a bulk RNA-seq dataset rather than a
  single-cell, isoform-level percentile correlation matrix
- Modules from dynamicTreeCut are already distinct with low inter-cluster metatranscript
  correlation — merging would incorrectly collapse genuinely different expression
  modalities
- The goal is transcriptome reconstruction or isoform functional annotation (SQANTI3/IsoAnnotLite/tappAS
  territory), not co-expression module definition
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: acorde
  role: R package implementing the semi-automated clustering, metatranscript-based
    cluster merging, and unclustered-isoform expansion pipeline
  repo: https://github.com/ConesaLab/acorde
- name: dynamicTreeCut
  role: initial hierarchical module detection on the percentile correlation matrix
    prior to filtering and merging
- name: dismay
  role: provides co-expression metric functions underlying some correlations available
    in acorde (must be installed from GitHub)
  repo: https://github.com/skinnider/dismay
- name: R
  role: runtime environment for the acorde clustering and merging workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:8dcdb67ccd2e71495bd082cd5745388f59aa7337cd710abcccd50cbe5c677537
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - R
  - acorde
  - dismay
  - dynamicTreeCut
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metatranscript-based cluster merging

## Summary

A module-refinement step in the acorde isoform co-expression clustering pipeline in which candidate isoform clusters are merged based on the correlation between their cluster metatranscripts (aggregate expression profiles), yielding non-redundant co-expression modules across cell types.

## When to use

Use when a dynamicTreeCut-initialized partition of isoform co-expression clusters contains redundant or highly correlated modules that must be consolidated before downstream coDIU or functional analysis.

## When NOT to use

- Input is a gene-level expression matrix or a bulk RNA-seq dataset rather than a single-cell, isoform-level percentile correlation matrix
- Modules from dynamicTreeCut are already distinct with low inter-cluster metatranscript correlation — merging would incorrectly collapse genuinely different expression modalities
- The goal is transcriptome reconstruction or isoform functional annotation (SQANTI3/IsoAnnotLite/tappAS territory), not co-expression module definition

## Inputs

- percentile correlation matrix computed from single-cell isoform expression data (mouse neural dataset, 6794 analyzed isoforms across 7 broad cell types)
- dynamicTreeCut-derived module assignments (deepSplit=4, pamStage=FALSE, minClusterSize=20)
- correlation-based module filtering results

## Outputs

- non-redundant set of merged isoform co-expression clusters (15 clusters covering all 6794 isoforms in the reference run)
- final cluster assignment table (isoform_cluster_assignments.tsv)

## How to apply

After initializing modules with dynamicTreeCut on the percentile correlation matrix (deepSplit=4, pamStage=FALSE, minClusterSize=20), apply correlation-based filtering of detected modules, then compute a representative metatranscript per cluster and merge clusters whose metatranscripts are highly correlated; in the mouse neural dataset this included two manually guided cluster merges. After merging, isoforms left unclustered can be recursively assigned by expansion at decreasing correlation thresholds (0.9, then 0.8, then 0.7) to the nearest matching cluster. The rationale is that over-fragmented modules from aggressive dynamicTreeCut settings represent the same underlying isoform co-usage pattern and merging restores functionally interpretable, cell type-specific modules.

## Related tools

- **acorde** (R package implementing the semi-automated clustering, metatranscript-based cluster merging, and unclustered-isoform expansion pipeline) — https://github.com/ConesaLab/acorde
- **dynamicTreeCut** (initial hierarchical module detection on the percentile correlation matrix prior to filtering and merging)
- **dismay** (provides co-expression metric functions underlying some correlations available in acorde (must be installed from GitHub)) — https://github.com/skinnider/dismay
- **R** (runtime environment for the acorde clustering and merging workflow)

## Examples

```
devtools::install_github("ConesaLab/acorde"); devtools::install_github("skinnider/dismay")
```

## Evaluation signals

- Final partition yields exactly 15 distinct clusters in the mouse neural dataset reference run
- All 6794 initially analyzed isoforms are assigned to a cluster (no isoform left unclustered after recursive expansion at 0.9/0.8/0.7 thresholds)
- No remaining pair of clusters has highly correlated metatranscripts (merging has removed redundant profiles, including the two manual merge pairs)
- Every cluster meets the minClusterSize=20 constraint established at dynamicTreeCut initialization
- Cluster assignments reproduce cell type-specific alternative isoform usage patterns across the 7 broad cell types

## Limitations

- Merging is semi-automated: two cluster pairs in the reference pipeline were merged manually, so results depend on operator judgment
- Percentile correlations are needed to overcome single-cell data sparsity; standard correlations on sparse isoform matrices will not support reliable metatranscript comparison
- The acorde package excludes upstream long-read transcriptome building and functional analyses (tappAS/IsoAnnotLite), so merging must be embedded in a hybrid workflow

## Evidence

- [intro] semi-automated clustering rationale: "a semi-automated clustering approach to detect modules of co-expressed isoforms across cell types"
- [abstract] correlation-based cluster network: "Next, acorde uses correlations to cluster co-expressed isoforms into a network, unraveling cell type-specific alternative isoform usage patterns"
- [abstract] percentile correlation necessity: "we develop and validate percentile correlations, an innovative approach that overcomes data sparsity and yields accurate co-expression estimates from single-cell data"
- [readme] README pipeline block: "Detection of isoform co-expression relationships using percentile correlations and semi-automated clustering."
