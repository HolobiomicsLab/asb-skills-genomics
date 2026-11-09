---
name: recursive-isoform-cluster-expansion
description: Use when after module detection on a percentile correlation matrix of
  single-cell isoform expression, some isoforms remain unassigned and a complete isoform
  co-usage partition is required.
when_to_use_negative:
- Input is bulk RNA-seq or full-length isoform quantification without single-cell
  sparsity — the graduated-threshold expansion is designed to rescue isoforms lost
  to sparse single-cell percentile correlation estimates
- The clustering goal is a strict, resolution-oriented partition where unclustered
  isoforms should remain unassigned (e.g. outlier/noise isoforms) rather than a complete
  assignment of every isoform
- No module initialization exists yet — expansion operates on existing dynamicTreeCut-derived
  clusters, not on a raw expression matrix
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: acorde
  role: R package implementing the semi-automated clustering pipeline, including cluster
    filtering, metatranscript-based merging, and recursive assignment of unclustered
    isoforms
  repo: https://github.com/ConesaLab/acorde
- name: dynamicTreeCut
  role: initial module detection algorithm run on the percentile correlation matrix
    (deepSplit=4, pamStage=FALSE, minClusterSize=20) before expansion
- name: dismay
  role: provides the co-expression metric functions underlying some of acorde's percentile
    correlation estimates
  repo: https://github.com/skinnider/dismay
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:1b75e44e12ce5725e961ffc93d720c25b368250e9ebbec59357ef1273d04b804
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
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# recursive-isoform-cluster-expansion

## Summary

A semi-automated post-clustering rescue step in the acorde pipeline that iteratively assigns initially unclustered isoforms to existing modules by expanding at decreasing percentile-correlation thresholds (0.9, then 0.8, then 0.7) until all isoforms are placed, yielding a complete partition of co-expressed isoform clusters.

## When to use

Use when, after module detection on a percentile correlation matrix of single-cell isoform expression, some isoforms remain unassigned and a complete isoform co-usage partition is required.

## When NOT to use

- Input is bulk RNA-seq or full-length isoform quantification without single-cell sparsity — the graduated-threshold expansion is designed to rescue isoforms lost to sparse single-cell percentile correlation estimates
- The clustering goal is a strict, resolution-oriented partition where unclustered isoforms should remain unassigned (e.g. outlier/noise isoforms) rather than a complete assignment of every isoform
- No module initialization exists yet — expansion operates on existing dynamicTreeCut-derived clusters, not on a raw expression matrix

## Inputs

- percentile correlation matrix computed from single-cell isoform expression data
- dynamicTreeCut-initialized cluster assignments on the correlation matrix
- cluster metatranscripts (representative expression profiles used for merging and matching)
- cell type/identity labels for the single-cell dataset (7 broad cell types in the mouse neural dataset)

## Outputs

- final isoform-to-cluster assignments covering all 6794 analyzed isoforms (15 clusters), e.g. written to isoform_cluster_assignments.tsv
- complete isoform co-expression network partition suitable for downstream coDIU detection and metatranscript interpretation

## How to apply

First initialize clustering with dynamicTreeCut on the percentile correlation matrix (deepSplit=4, pamStage=FALSE, minClusterSize=20), then filter detected modules by correlation and merge redundant clusters via metatranscript-based correlation merging (including manual merges of redundant profiles). Because dynamicTreeCut leaves isoforms unassigned, recursively expand each cluster: at each round, lower the correlation threshold (0.9 -> 0.8 -> 0.7) and assign each unclustered isoform to the nearest matching cluster whose members it correlates with at that threshold. Rationale: single-cell sparsity makes strict cutoffs too conservative, so graduated thresholds recover isoforms whose co-expression signal is weaker while still anchored to a module's metatranscript. Terminate when every one of the analyzed isoforms (6794 in the mouse neural dataset) is assigned, and validate that the final partition contains the expected number of clusters (15) covering all cell types.

## Related tools

- **acorde** (R package implementing the semi-automated clustering pipeline, including cluster filtering, metatranscript-based merging, and recursive assignment of unclustered isoforms) — https://github.com/ConesaLab/acorde
- **dynamicTreeCut** (initial module detection algorithm run on the percentile correlation matrix (deepSplit=4, pamStage=FALSE, minClusterSize=20) before expansion)
- **dismay** (provides the co-expression metric functions underlying some of acorde's percentile correlation estimates) — https://github.com/skinnider/dismay

## Examples

```
Rscript -e 'library(acorde); clusters <- cutreeDynamic(dendro, deepSplit=4, pamStage=FALSE, minClusterSize=20); expanded <- acorde::expand_unclustered(clusters, corr_matrix, thresholds=c(0.9,0.8,0.7)); write.table(expanded, "isoform_cluster_assignments.tsv", sep="\t")'
```

## Evaluation signals

- Final partition is complete: the number of assigned isoforms equals the number analyzed (all 6794 isoforms in the mouse neural dataset are assigned)
- The final partition contains the expected number of clusters (exactly 15 distinct clusters after merging)
- Cluster count is stable across the decreasing threshold rounds — expansion at 0.9 -> 0.8 -> 0.7 should not fragment existing modules, only absorb unclustered isoforms
- Every isoform's assigned cluster matches its nearest cluster metatranscript at the round's correlation threshold (0.7 minimum at final round)
- Clusters span diverse expression modalities across the 7 broad cell types of the mouse neural dataset

## Limitations

- Percentile correlations and the clustering pipeline were validated on the Smart-Seq2 mouse visual cortex dataset (Tasic et al.) combined with ENCODE PacBio long-read-defined transcriptomes; behavior on other scRNA-seq chemistries or species is not established in this study
- Expansion is greedy/nearest-matching — an isoform rescued at 0.7 is anchored to the best-correlated cluster rather than re-evaluated globally, so borderline isoforms may be misassigned
- The pipeline is semi-automated: redundant cluster profiles may require manual merges, so results are not fully reproducible without recording manual merge decisions
- Functional interpretation of resulting clusters is hypothesis-generating and requires further experimental validation

## Evidence

- [intro] semi-automated clustering to detect modules of co-expressed isoforms: "a semi-automated clustering approach to detect modules of co-expressed isoforms across cell types"
- [abstract] percentile correlations motivation: "we develop and validate percentile correlations, an innovative approach that overcomes data sparsity and yields accurate co-expression estimates from single-cell data"
- [abstract] clustering into a network: "Next, acorde uses correlations to cluster co-expressed isoforms into a network, unraveling cell type-specific alternative isoform usage patterns"
- [readme] README pipeline block: "Detection of isoform co-expression relationships using percentile correlations
and semi-automated clustering."
- [discussion] workflow chain in discussion: "acorde can effectively be used to leverage single-cell RNA-seq data to build isoform co-expression networks"
