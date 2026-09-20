---
name: differential-isoform-usage-analysis
description: Use when you have a single-cell isoform-level expression matrix with
  cell type/state labels and need to detect genes whose isoform usage (not just expression)
  differs across multiple cell types, or genes with coordinated isoform co-usage (coDIU).
when_to_use_negative:
- Input is gene-level rather than isoform-level counts — DIU tests isoform proportion
  changes within genes and cannot be defined from a gene count matrix
- Only one cell group is available — DIU and coDIU are defined across multiple cell
  types/states
- Genes whose significant DE isoforms all fall in a single co-expression cluster —
  no co-usage pattern exists for them by construction
- Standard reference transcriptomes where functional annotation of isoforms is unavailable,
  if downstream functional analysis is required
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: acorde (R package)
  role: Implements the DIU and coDIU testing workflow, percentile correlation clustering,
    minor-isoform filtering, and provides the internal example dataset used in the
    manuscript
  repo: https://github.com/ConesaLab/acorde
- name: DESeq2
  role: Statistical backend for multi-group DIU testing on (optionally ZinBWaVE-weighted)
    downsampled isoform count matrices
- name: edgeR
  role: Alternative statistical backend for multi-group DIU testing, used alongside
    DESeq2 to derive consensus DE/DIU sets
- name: ZinBWaVE
  role: Computes zero-inflated weights for downsampled count matrices prior to DIU
    testing
- name: tappAS
  role: Downstream functional analysis of DIU/coDIU genes using the isoAnnotLite-annotated
    transcriptome
  repo: https://github.com/ConesaLab/tappAS
provenance:
  source_task_ids:
  - task_008
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:0a72ec273e465670277c57b65c991e1c17eecabc2a754c8c8b646410dd3f9842
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - BiocStyle
  - DESeq2
  - R
  - ZinBWaVE
  - acorde
  - acorde (R package)
  - edgeR
  - knitr
  - rmarkdown
  - tappAS
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 26
---

# differential-isoform-usage-analysis

## Summary

Differential Isoform Usage (DIU) and co-Differential Isoform Usage (coDIU) testing as implemented in the acorde R package: testing isoform proportion shifts of genes across multiple cell types in single-cell isoform count matrices, and detecting genes whose same-gene DE isoforms belong to different co-expression clusters (coDIU).

## When to use

Use when you have a single-cell isoform-level expression matrix with cell type/state labels and need to detect genes whose isoform usage (not just expression) differs across multiple cell types, or genes with coordinated isoform co-usage (coDIU).

## When NOT to use

- Input is gene-level rather than isoform-level counts — DIU tests isoform proportion changes within genes and cannot be defined from a gene count matrix
- Only one cell group is available — DIU and coDIU are defined across multiple cell types/states
- Genes whose significant DE isoforms all fall in a single co-expression cluster — no co-usage pattern exists for them by construction
- Standard reference transcriptomes where functional annotation of isoforms is unavailable, if downstream functional analysis is required

## Inputs

- Isoform-level single-cell quantification matrix (e.g. long-read transcriptome expression estimated in Tasic Smart-Seq2 cells)
- Isoform-to-gene relationships (transcript-to-gene mapping)
- Cell-level identity labels (cell type/state, e.g. Lamp5, Pvalb, Sncg, Sst, Vip)
- Functionally annotated transcriptome (optional, for downstream tappAS functional analysis)

## Outputs

- Table of DIU significance results (per-method FDR values per isoform/gene)
- CoDIU gene list with the isoforms involved in each co-usage relationship
- Summary metrics per method (number of DE isoforms, Jaccard index across replicates/downsampling rounds)
- Filtered isoform-gene tables (e.g. consensus_diu_isoforms.csv, downsampling_robustness_metrics.csv)

## How to apply

Run acorde's DIU workflow in R on an isoform-level quantification matrix with isoform-to-gene relationships and cell-level identity labels; for unbalanced groups, downsample cells (e.g. 45 cells per neural type) and optionally compute ZinBWaVE zero-inflated weights before multi-group DIU testing with DESeq2 and/or edgeR via acorde. For coDIU, first cluster co-expressed isoforms using percentile correlations (the sparsity-robust estimator) into merged expression profiles, then select same-gene isoforms that fall into different clusters to identify co-differential isoform usage genes. Apply the minor-isoform filter (isoforms with expression <10% of their gene's total are removed) and exclude genes left with only one significant DE isoform, since no isoform co-usage pattern can be defined for them. Validate against known benchmarks from the Tasic 2018 GABA-ergic analysis: 568 significant DE isoforms, 5 merged clusters, 22 DIU genes, and 16 coDIU genes involving 36 isoforms.

## Related tools

- **acorde (R package)** (Implements the DIU and coDIU testing workflow, percentile correlation clustering, minor-isoform filtering, and provides the internal example dataset used in the manuscript) — https://github.com/ConesaLab/acorde
- **DESeq2** (Statistical backend for multi-group DIU testing on (optionally ZinBWaVE-weighted) downsampled isoform count matrices)
- **edgeR** (Alternative statistical backend for multi-group DIU testing, used alongside DESeq2 to derive consensus DE/DIU sets)
- **ZinBWaVE** (Computes zero-inflated weights for downsampled count matrices prior to DIU testing)
- **tappAS** (Downstream functional analysis of DIU/coDIU genes using the isoAnnotLite-annotated transcriptome) — https://github.com/ConesaLab/tappAS

## Examples

```
devtools::install_github("ConesaLab/acorde", build_vignettes = TRUE); library(acorde)  # run vignette DIU/coDIU workflow on internal Tasic isoform matrix
```

## Evaluation signals

- Reproducing the Tasic 2018 five GABA-ergic subtype run yields exactly 22 DIU genes, 16 coDIU genes involving 36 isoforms, 5 merged clusters, and 568 significant DE isoforms; any mismatch flags the pipeline run as failed
- The minor-isoform filter removes 969 isoforms from the 9393-isoform consensus set, leaving exactly 6794 isoforms from 2696 genes in consensus_diu_isoforms.csv
- Downsampling robustness metrics fall within reported ranges: DESeq2 6908±101 DE isoforms (J=0.84±0.02), edgeR 6016±410 (J=0.74±0.02), union 9399±248 (J=0.82±0.01)
- Every coDIU call maps to a gene with same-gene DE isoforms distributed across at least two different merged isoform-expression clusters
- Consensus DE/DIU selection retains isoforms significant (FDR<0.05 by at least one method) in ≥50% of the 50 downsampling runs

## Limitations

- acorde's R package does not include the long-read transcriptome building or functional analysis steps; those require external tools (SQANTI3/IsoAnnotLite, tappAS) and a pre-annotated compatible reference (e.g. Mus_Musculus_GRCm38.p6_PacBioENCODE_RefSeq108.gff3)
- Genes with only one significant DE isoform must be excluded because no isoform co-usage pattern can be defined for them
- Mechanisms generating isoform co-expression remain speculative and the functional insights are hypothesis-generating, requiring further experimental validation
- Co-expression/co-usage analysis is sensitive to data sparsity, which is why percentile correlations rather than standard correlations are used

## Evidence

- [intro] We additionally defined and implemented Differential Isoform Usage (DIU) and co-Differential Isoform Usage (coDIU) analyses in order to leverage the multiple cell types contained in single-cell: "We additionally defined and implemented Differential Isoform Usage (DIU) and co-Differential Isoform Usage (coDIU) analyses in order to leverage the multiple cell types"
- [abstract] By selecting same-gene isoforms between these clusters, we subsequently detect and characterize genes with co-differential isoform usage (coDIU) across cell types: "By selecting same-gene isoforms between these clusters, we subsequently detect and characterize genes with co-differential isoform usage (coDIU)"
- [readme] *acorde* contains the necessary functions and documentation to obtain a set of DIU and co-DIU genes using an single-cell, isoform-level expression matrix as input.: "*acorde* contains the necessary functions and documentation to obtain a set of DIU and co-DIU genes using an single-cell, isoform-level expression"
- [abstract] acorde uses correlations to cluster co-expressed isoforms into a network, unraveling cell type-specific alternative isoform usage patterns: "acorde uses correlations to cluster co-expressed isoforms into a network, unraveling cell type-specific alternative isoform usage patterns"
- [full_text] Apply acorde's minor-isoform filter, removing isoforms whose expression is <10% of their gene's total (removing 969 isoforms).: [evidence span withheld — anchor status: not_found]
