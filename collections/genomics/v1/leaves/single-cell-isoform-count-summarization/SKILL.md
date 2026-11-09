---
name: single-cell-isoform-count-summarization
description: Use when the input is a filtered isoform-level single-cell count matrix
  (e.g. 6794 isoforms from Tasic et al. Smart-Seq2 data) with cell type labels, and
  sparse cell-level counts make direct cell-to-cell correlations too noisy for co-expression
  estimation.
when_to_use_negative:
- Input is bulk RNA-seq or already aggregated to cell-type/group-level expression,
  where direct correlations suffice and percentile summarization of single-cell sparsity
  is unnecessary
- The goal is functional (tappAS/isoAnnotLite) annotation analysis or DIU/coDIU testing
  rather than co-expression estimation, since acorde handles those as separate analysis
  blocks
- The count matrix has not been DE-filtered, so the isoform set has not been reduced
  to isoforms with detectable expression signal
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: acorde
  role: R package providing the percentile correlation implementation that computes
    decile-summarized correlations across cell types and feeds the semi-automated
    clustering step
  repo: https://github.com/ConesaLab/acorde
- name: R
  role: Runtime environment for loading the filtered isoform count matrix and computing/saving
    the percentile correlation matrix (as .rds)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:4179a791ecf63bb7a2554935784603735b4ce2e7a902903012651a3ff4340e48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - R
  - acorde
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-isoform-count-summarization

## Summary

Summarizes sparse single-cell isoform count values into cell-type-level percentile profiles (10 deciles) before computing correlations, as implemented in acorde's percentile correlation method for isoform co-expression estimation.

## When to use

Use when the input is a filtered isoform-level single-cell count matrix (e.g. 6794 isoforms from Tasic et al. Smart-Seq2 data) with cell type labels, and sparse cell-level counts make direct cell-to-cell correlations too noisy for co-expression estimation.

## When NOT to use

- Input is bulk RNA-seq or already aggregated to cell-type/group-level expression, where direct correlations suffice and percentile summarization of single-cell sparsity is unnecessary
- The goal is functional (tappAS/isoAnnotLite) annotation analysis or DIU/coDIU testing rather than co-expression estimation, since acorde handles those as separate analysis blocks
- The count matrix has not been DE-filtered, so the isoform set has not been reduced to isoforms with detectable expression signal

## Inputs

- Filtered isoform-level expression count matrix from single-cell RNA-seq (6794 isoforms, Tasic et al. mouse primary visual cortex Smart-Seq2 data)
- Cell-level identity labels assigning each cell to one of 7 cell types
- Isoform-to-gene relationships

## Outputs

- Percentile-summarized isoform x cell-type expression matrix (decile values per cell type)
- 6794 x 6794 symmetric isoform-to-isoform Pearson correlation matrix with unit diagonal (e.g. percentile_correlation_matrix.rds), suitable as input to acorde's semi-automated clustering

## How to apply

Load the filtered isoform expression count matrix (6794 isoforms) in R and the cell-level identity labels for the 7 cell types. For each isoform, compute the p=10 percentile (decile) summary of single-cell counts within each cell type, replacing sparse cell-level values with a cell-type-level expression distribution signature. Compute pairwise Pearson correlations between isoforms across the 7 percentile-summarized cell-type profiles using acorde's percentile correlation implementation. Assemble the full 6794 x 6794 symmetric isoform-to-isoform correlation matrix and save it (e.g. percentile_correlation_matrix.rds) as input for acorde's semi-automated isoform clustering. The rationale is that percentile correlations overcome data sparsity and yield accurate co-expression estimates that better reflect isoform co-usage than traditional cell-level correlation metrics.

## Related tools

- **acorde** (R package providing the percentile correlation implementation that computes decile-summarized correlations across cell types and feeds the semi-automated clustering step) — https://github.com/ConesaLab/acorde
- **R** (Runtime environment for loading the filtered isoform count matrix and computing/saving the percentile correlation matrix (as .rds))

## Examples

```
devtools::install_github("ConesaLab/acorde", build_vignettes = TRUE)
```

## Evaluation signals

- Resulting correlation matrix is symmetric with dimensions 6794 x 6794
- Correlation matrix diagonal equals 1 (unit diagonal)
- Each isoform contributes exactly 10 percentile values (deciles) per cell type, one per cell type, replacing cell-level counts
- Matrix loads in R as a valid object usable as input to acorde's semi-automated isoform clustering step
- Isoform count is consistent with the DE-filtered acorde vignette matrix (6794 isoforms)

## Limitations

- Percentile correlation is designed for sparse single-cell data; it summarizes counts to the cell-type level, so within-cell-type (cell-to-cell) co-expression variation is lost
- acorde's R package excludes the long-read transcriptome building and functional analysis modules; those depend on external tools (SQANTI3/IsoAnnotLite/tappAS)
- Results depend on the input being a DE-filtered isoform quantification matrix and on reliable cell type labels; acorde requires an isoform-level quantification matrix plus isoform-to-gene relationships and cell-level identity labels

## Evidence

- [abstract] percentile correlations overcome data sparsity and yield accurate co-expression estimates from single-cell data: "we develop and validate percentile correlations, an innovative approach that overcomes data sparsity and yields accurate co-expression estimates from single-cell data"
- [readme] Detection of isoform co-expression relationships using percentile correlations and semi-automated clustering.: "Detection of isoform co-expression relationships using percentile correlations
and semi-automated clustering."
- [readme] In order to use acorde, you will need the following items: an isoform-level quantification matrix and cell-level identity labels.: "In order to use acorde, you will need the following items:

- An isoform-level quantification matrix.
- Isoform-to-gene relationships and cell-level identity labels (i.e. cell type
or state)."
