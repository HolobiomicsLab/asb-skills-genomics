---
name: percentile-correlation-co-expression-estimation
description: Use when estimating isoform co-expression from sparse scRNA-seq data
  (e.g. Smart-Seq2 counts across multiple cell types/subtypes) where standard correlation
  is unreliable due to dropout, and when clustering co-expressed isoforms into networks
  or detecting co-differential isoform usage (coDIU) genes.
when_to_use_negative:
- When the input is bulk RNA-seq or another non-sparse expression matrix — percentile
  correlations were designed to overcome scRNA-seq data sparsity, and standard correlation
  suffices there
- When the goal is differential gene or transcript expression (DE/DTE) rather than
  co-expression / isoform co-usage relationships — acorde's DE filtering is a prerequisite
  step, not the output of this skill
- When no isoform-level resolution is available (e.g. only gene-level counts) — the
  method operates on isoform expression matrices and same-gene isoform pairs
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: acorde (R package)
  role: Implements percentile correlation computation, semi-automated clustering of
    co-expressed isoforms, and DIU/coDIU analysis; provides the isoform expression
    matrix used in the study as internal package data
  repo: https://github.com/ConesaLab/acorde
- name: dismay
  role: Provides the underlying co-expression metrics that some acorde percentile-correlation
    functions are based on; must be installed from GitHub before acorde
  repo: https://github.com/skinnider/dismay
- name: tappAS
  role: Downstream functional analysis of the coDIU/DIU genes and isoforms surfaced
    by the percentile-correlation clustering step
  repo: https://github.com/ConesaLab/tappAS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:13fafca82ecf60f14c6be57f31b8acc2838cf1fdf2b19acb865f183715a2308b
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
  - knitr
  - rmarkdown
  - tappAS
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# percentile-correlation-co-expression-estimation

## Summary

Estimate co-expression between isoforms from sparse single-cell RNA-seq data using percentile correlations, the noise- and sparsity-robust correlation estimator developed and validated in the acorde pipeline, then cluster co-expressed isoforms into co-usage networks and flag coDIU genes.

## When to use

Use when estimating isoform co-expression from sparse scRNA-seq data (e.g. Smart-Seq2 counts across multiple cell types/subtypes) where standard correlation is unreliable due to dropout, and when clustering co-expressed isoforms into networks or detecting co-differential isoform usage (coDIU) genes.

## When NOT to use

- When the input is bulk RNA-seq or another non-sparse expression matrix — percentile correlations were designed to overcome scRNA-seq data sparsity, and standard correlation suffices there
- When the goal is differential gene or transcript expression (DE/DTE) rather than co-expression / isoform co-usage relationships — acorde's DE filtering is a prerequisite step, not the output of this skill
- When no isoform-level resolution is available (e.g. only gene-level counts) — the method operates on isoform expression matrices and same-gene isoform pairs

## Inputs

- Isoform-level single-cell expression/quantification matrix (e.g. from Smart-Seq2 scRNA-seq with long-read-defined isoforms)
- Isoform-to-gene mapping and cell-level identity labels (cell type or subtype, e.g. Lamp5, Pvalb, Sncg, Sst, Vip)
- Bulk long-read (PacBio) transcriptome used to define the isoform repertoire
- Optional functionally annotated transcriptome (e.g. IsoAnnotLite GFF3) for downstream tappAS functional analysis

## Outputs

- Percentile correlation matrix of isoform pairs
- Clusters/modules of co-expressed isoforms (merged expression profiles, e.g. 5 final clusters)
- List of DIU genes across cell types
- List of coDIU genes with their participating isoforms (e.g. 16 genes / 36 isoforms)

## How to apply

Integrate bulk long-read transcriptome definitions (e.g. ENCODE PacBio mouse cortex/hippocampus) with single-cell short-read counts to estimate isoform-level expression per cell, then compute percentile correlations between isoforms instead of standard Pearson/Spearman coefficients, because the sparse, dropout-prone single-cell expression matrix defeats conventional estimators. Feed the percentile correlation estimates into acorde's semi-automated clustering to merge isoforms with similar expression profiles into co-expression clusters/modules across cell types. Once clusters are defined (the Tasic 2018 GABA-ergic run produced 5 final merged profile clusters), select same-gene isoforms located in different clusters to identify genes with co-differential isoform usage (coDIU); validate the run by requiring exact reproduction of the reported counts (568 DE isoforms after filtering, 5 merged profiles, 22 DIU genes, 16 coDIU genes involving 36 isoforms). Rationale: the acorde authors validated percentile correlations specifically as the method that overcomes data sparsity in scRNA-seq, and coupling the correlation step to DIU/coDIU analysis turns co-expression into a biologically interpretable isoform co-usage readout.

## Related tools

- **acorde (R package)** (Implements percentile correlation computation, semi-automated clustering of co-expressed isoforms, and DIU/coDIU analysis; provides the isoform expression matrix used in the study as internal package data) — https://github.com/ConesaLab/acorde
- **dismay** (Provides the underlying co-expression metrics that some acorde percentile-correlation functions are based on; must be installed from GitHub before acorde) — https://github.com/skinnider/dismay
- **tappAS** (Downstream functional analysis of the coDIU/DIU genes and isoforms surfaced by the percentile-correlation clustering step) — https://github.com/ConesaLab/tappAS

## Examples

```
install.packages("devtools"); devtools::install_github("skinnider/dismay"); devtools::install_github("ConesaLab/acorde", build_vignettes = TRUE)
```

## Evaluation signals

- Merged co-expression profile count matches the reference run exactly (5 clusters for the Tasic 2018 GABA-ergic subtypes)
- DIU gene count matches the reference (22 genes among the 5 GABA-ergic subtypes)
- coDIU gene count and participating isoform count match the reference (16 coDIU genes / 36 isoforms)
- Upstream DE filtering yields 568 significant DE isoforms before co-expression clustering
- Number of cells passing QC matches the reference dataset (4921 cells for the five GABA-ergic subtypes)

## Limitations

- Percentile correlations were developed and validated specifically for sparse single-cell data; behavior on non-scRNA-seq inputs is not established
- Functional elements predicted from long-read isoforms are hypothesis-generating and require experimental validation
- Mechanisms generating isoform co-expression remain speculative / unresolved
- Functional annotation of long-read-defined isoforms depends on pre-annotated references via IsoAnnotLite; de novo annotation tooling is a work in progress
- acorde package does not include the long-read transcriptome building step or the functional analysis; both require external tools (Supplementary Note / tappAS)

## Evidence

- [abstract] Percentile correlations overcome data sparsity: "we develop and validate percentile correlations, an innovative approach that overcomes data sparsity and yields accurate co-expression estimates from single-cell data"
- [abstract] Acorde clusters co-expressed isoforms into a network: "Next, acorde uses correlations to cluster co-expressed isoforms into a network, unraveling cell type-specific alternative isoform usage patterns"
- [abstract] coDIU detection by same-gene isoform selection between clusters: "By selecting same-gene isoforms between these clusters, we subsequently detect and characterize genes with co-differential isoform usage (coDIU) across cell types"
- [readme] dismay dependency for co-expression metrics: [evidence span withheld — anchor status: external_doc]
- [readme] Example dataset and isoform expression matrix provided: [evidence span withheld — anchor status: external_doc]
