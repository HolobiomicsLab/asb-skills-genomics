---
name: pearson-correlation-matrix-construction
description: Use when a single-cell isoform-level count matrix is too sparse for reliable
  cell-level Pearson correlation, and a noise-robust, cell-type-summarized co-expression
  matrix is needed as input to isoform clustering (e.g. acorde semi-automated clustering).
when_to_use_negative:
- 'Bulk RNA-seq or already cell-type-aggregated data: percentile summarization exists
  specifically to overcome single-cell sparsity, and standard correlation on dense
  bulk profiles is sufficient'
- When per-cell (not per-cell-type) co-expression relationships are the target — the
  decile summarization deliberately discards cell-level resolution by collapsing counts
  to 10 percentiles within each of the 7 cell types
- 'Before the isoform count matrix has been DE-filtered by acorde''s pipeline: clustering
  the unfiltered matrix is out of the documented workflow order'
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: acorde
  role: Implements the percentile correlation approach (decile summarization within
    cell types) whose output Pearson correlation matrix feeds the semi-automated isoform
    clustering; provides the example isoform matrix as internal data
  repo: https://github.com/ConesaLab/acorde
- name: R
  role: Environment in which the acorde pipeline (including vignette-based percentile
    correlation computation via quantile and cor) is executed
  repo: https://github.com/ConesaLab/acorde
- name: dismay
  role: Backend for some of the alternative co-expression metrics available in acorde
    (installed from GitHub since not on CRAN); the percentile correlation matrix is
    one of the co-expression inputs
  repo: https://github.com/skinnider/dismay
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:c4380bc032a19b11146ba184af2e1bbe51ccc37cf6c972e630b6a31678b6a177
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - R
  - acorde
  - dismay
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# pearson-correlation-matrix-construction

## Summary

Constructs an isoform-to-isoform Pearson correlation matrix (6794 x 6794) from sparse single-cell isoform counts by first replacing raw Smart-Seq2 cell-level counts with cell-type-level decile (p=10 percentile) profiles, as implemented in acorde's percentile correlation approach to defeat single-cell noise.

## When to use

Use when a single-cell isoform-level count matrix is too sparse for reliable cell-level Pearson correlation, and a noise-robust, cell-type-summarized co-expression matrix is needed as input to isoform clustering (e.g. acorde semi-automated clustering).

## When NOT to use

- Bulk RNA-seq or already cell-type-aggregated data: percentile summarization exists specifically to overcome single-cell sparsity, and standard correlation on dense bulk profiles is sufficient
- When per-cell (not per-cell-type) co-expression relationships are the target — the decile summarization deliberately discards cell-level resolution by collapsing counts to 10 percentiles within each of the 7 cell types
- Before the isoform count matrix has been DE-filtered by acorde's pipeline: clustering the unfiltered matrix is out of the documented workflow order

## Inputs

- Isoform-level single-cell count matrix (6794 isoforms x cells, filtered by acorde DE filtering; e.g. the Tasic et al. mouse V1 Smart-Seq2 dataset bundled as acorde internal data)
- Cell-level identity labels assigning each cell to one of 7 cell types (isoform-to-gene relationships may also accompany the matrix per the README)

## Outputs

- Symmetric 6794 x 6794 isoform-to-isoform Pearson correlation matrix with unit diagonal (saved as percentile_correlation_matrix.rds), suitable as input to acorde's semi-automated clustering

## How to apply

Load the filtered isoform expression count matrix (6794 isoforms from the Tasic et al. mouse primary visual cortex Smart-Seq2 dataset, provided as internal data in the acorde package) together with cell-type identity labels for the 7 cell types. For each isoform, group its single-cell counts by cell type and replace the raw counts with the p=10 percentile values (deciles) computed within each cell type, yielding a percentile-summarized expression signature per isoform per cell type. Compute pairwise Pearson correlations between all isoforms across these decile-summarized profiles, assembling the full symmetric 6794 x 6794 correlation matrix. The rationale, per the acorde paper, is that decile summarization overcomes single-cell data sparsity and produces co-expression estimates that better reflect isoform co-usage than traditional cell-level correlation. Save the matrix (e.g. percentile_correlation_matrix.rds) as the direct input to acorde's semi-automated isoform clustering step.

## Related tools

- **acorde** (Implements the percentile correlation approach (decile summarization within cell types) whose output Pearson correlation matrix feeds the semi-automated isoform clustering; provides the example isoform matrix as internal data) — https://github.com/ConesaLab/acorde
- **R** (Environment in which the acorde pipeline (including vignette-based percentile correlation computation via quantile and cor) is executed) — https://github.com/ConesaLab/acorde
- **dismay** (Backend for some of the alternative co-expression metrics available in acorde (installed from GitHub since not on CRAN); the percentile correlation matrix is one of the co-expression inputs) — https://github.com/skinnider/dismay

## Examples

```
cell_deciles <- sapply(levels(factor(cell_types)), function(ct) apply(iso_counts[, cell_types == ct], 1, quantile, probs = seq(0.1, 1, 0.1))); cor_mat <- cor(t(cell_deciles), method = "pearson"); saveRDS(cor_mat, "percentile_correlation_matrix.rds")
```

## Evaluation signals

- Correlation matrix is symmetric with dimensions 6794 x 6794 and unit diagonal before use as clustering input
- Each isoform's summarized profile contains exactly 10 percentile (decile) values per cell type (p=10), i.e. 7 cell types x 10 percentiles summarize the profile space
- All correlation values fall within [-1, 1]; Pearson correlations computed across percentile-summarized profiles, not raw cell-level counts
- Saved matrix loads as an R object (percentile_correlation_matrix.rds) and is accepted by acorde's semi-automated clustering without further reshaping

## Limitations

- Percentile summarization collapses cells to decile values within each cell type, so per-cell isoform co-expression variation is intentionally sacrificed to gain noise robustness
- Correlation structure depends on the quality of the cell-type labels; acorde requires an isoform-level quantification matrix plus cell-level identity labels as inputs
- Downstream interpretation of the co-expression networks is hypothesis-generating: the paper notes functional insights 'need to be subject to further experimental validation' and mechanisms generating isoform co-expression remain unresolved

## Evidence

- [abstract] Percentile correlations overcome single-cell sparsity: "we develop and validate percentile correlations, an innovative approach that overcomes data sparsity and yields accurate co-expression estimates from single-cell data"
- [intro] acorde provides noise-robust correlation strategy: "we developed a strategy to obtain noise-robust correlation estimates in scRNA-seq data"
- [abstract] Correlation matrix feeds clustering: "acorde uses correlations to cluster co-expressed isoforms into a network, unraveling cell type-specific alternative isoform usage patterns"
- [readme] README confirms pipeline block and clustering: "Detection of isoform co-expression relationships using percentile correlations"
- [readme] Example matrix available as package data: [evidence span withheld — anchor status: external_doc]
