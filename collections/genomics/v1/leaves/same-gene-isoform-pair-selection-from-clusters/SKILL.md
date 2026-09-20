---
name: same-gene-isoform-pair-selection-from-clusters
description: Use when isoform co-expression clusters (from percentile-correlation
  clustering) are available and the goal is to detect genes whose isoforms co-vary
  across cell types (coDIU) beyond gene-level differential usage.
when_to_use_negative:
- No isoform cluster assignments exist yet — run acorde's percentile-correlation clustering
  first, since candidate pairs are selected from clusters
- The input matrix is gene-level rather than isoform-level, so same-gene isoform pairs
  cannot be formed
- The aim is plain gene-level differential expression/DIU without testing isoform
  co-usage coordination
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3518
tools:
- name: acorde
  role: Provides clustering of co-expressed isoforms and the DIU/coDIU analysis functions
    that operate on the resulting isoform clusters
  repo: https://github.com/ConesaLab/acorde
- name: acorde (R package)
  role: R implementation installed via devtools from GitHub; contains the functions
    to obtain DIU and co-DIU genes from an isoform-level expression matrix
  repo: https://github.com/ConesaLab/acorde
- name: R
  role: Runtime environment in which the acorde package GLM-based coDIU tests are
    executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:740b8762c77b54fbfdaf25eab9cbcfa6e97f219407c8a5cc035e692ed64c1988
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - R
  - acorde
  - acorde (R package)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Same-gene isoform pair selection from clusters

## Summary

Screens isoform co-expression clusters for candidate pairs containing at least two same-gene isoforms, then tests them with negative-binomial GLMs (cluster*cell-type vs. gene*cell-type terms) to identify genes with co-differential isoform usage (coDIU) that is not visible at the gene level.

## When to use

Use when isoform co-expression clusters (from percentile-correlation clustering) are available and the goal is to detect genes whose isoforms co-vary across cell types (coDIU) beyond gene-level differential usage.

## When NOT to use

- No isoform cluster assignments exist yet — run acorde's percentile-correlation clustering first, since candidate pairs are selected from clusters
- The input matrix is gene-level rather than isoform-level, so same-gene isoform pairs cannot be formed
- The aim is plain gene-level differential expression/DIU without testing isoform co-usage coordination

## Inputs

- Isoform-level single-cell expression matrix (e.g. from the Tasic et al. Smart-Seq2 mouse visual cortex dataset)
- Isoform cluster assignments from acorde percentile-correlation co-expression clustering
- Isoform-to-gene relationships (transcript-to-gene mapping)
- Cell-level identity labels (cell type or state)

## Outputs

- Table of coDIU genes with significant cluster*cell-type interaction (FDR < 0.05) and non-significant gene*cell-type interaction (FDR > 0.05), written to codiu_results.csv
- Set of co-expressed isoforms belonging to the detected coDIU genes

## How to apply

Scan the isoform cluster assignments produced by the acorde percentile-correlation/semi-automated clustering step to find candidate gene pairs with at least two isoforms from the same gene assigned to the same clusters. For each candidate, fit negative-binomial GLMs modeling isoform usage as a function of cluster, cell type, and their interaction (cluster*cell-type and gene*cell-type terms). Apply FDR multiple-testing correction to the interaction terms and retain pairs with significant cluster*cell-type interaction (FDR < 0.05) and non-significant gene*cell-type interaction (FDR > 0.05), which defines coDIU. Summarize retained coDIU genes and their co-expressed isoforms into codiu_results.csv. Rationale: this couples the co-expression network structure to a differential-usage test that explicitly separates isoform-level coordination (cluster*cell-type) from gene-level DIU (gene*cell-type).

## Related tools

- **acorde** (Provides clustering of co-expressed isoforms and the DIU/coDIU analysis functions that operate on the resulting isoform clusters) — https://github.com/ConesaLab/acorde
- **acorde (R package)** (R implementation installed via devtools from GitHub; contains the functions to obtain DIU and co-DIU genes from an isoform-level expression matrix) — https://github.com/ConesaLab/acorde
- **R** (Runtime environment in which the acorde package GLM-based coDIU tests are executed)

## Examples

```
devtools::install_github("ConesaLab/acorde", build_vignettes = TRUE); library(acorde)
```

## Evaluation signals

- Significance filter logic: retained pairs must have cluster*cell-type FDR < 0.05 AND gene*cell-type FDR > 0.05
- Candidate invariant: every tested pair contains at least two isoforms from the same gene assigned to the same cluster
- On the Tasic et al. mouse visual cortex Smart-Seq2 dataset, results should reproduce 1784 coDIU genes involving 5274 co-expressed isoforms
- Output schema check: codiu_results.csv exists and summarizes coDIU genes together with their co-expressed isoforms

## Limitations

- Detection depends on prior percentile-correlation clustering quality; sparse scRNA-seq data motivated the percentile-correlation approach to obtain noise-robust estimates
- Functional interpretation of coDIU results requires a functionally-annotated transcriptome (e.g. IsoAnnotLite with a compatible pre-annotated reference); acorde's long-read transcriptome building is not included in the R package
- Example dataset results are specific to the mouse neural transcriptome reference used in the manuscript (e.g. Mus_Musculus_GRCm38.p6_PacBioENCODE_RefSeq108.gff3)

## Evidence

- [abstract] By selecting same-gene isoforms between these clusters, we subsequently detect and characterize genes with co-differential isoform usage (coDIU) across cell types: "By selecting same-gene isoforms between these clusters, we subsequently detect and characterize genes with co-differential isoform usage (coDIU) across cell types"
- [intro] We additionally defined and implemented Differential Isoform Usage (DIU) and co-Differential Isoform Usage (coDIU) analyses in order to leverage the multiple cell types contained in single-cell: "We additionally defined and implemented Differential Isoform Usage (DIU) and co-Differential Isoform Usage (coDIU) analyses in order to leverage the multiple cell types contained in single-cell"
- [discussion] acorde can effectively be used to leverage single-cell RNA-seq data to build isoform co-expression networks: "acorde can effectively be used to leverage single-cell RNA-seq data to build isoform co-expression networks"
- [readme] acorde contains the necessary functions and documentation to obtain a set of DIU and co-DIU genes using an single-cell, isoform-level expression matrix as input: "*acorde* contains the necessary functions and documentation to obtain 
a set of DIU and co-DIU genes using an single-cell, isoform-level expression 
matrix as input."
