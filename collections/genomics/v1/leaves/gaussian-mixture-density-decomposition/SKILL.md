---
name: gaussian-mixture-density-decomposition
description: Use when a population of per-cell confidence scores (e.g., Wilcoxon rank-sum
  scores on marker genes) must be split into validated vs. unvalidated sub-populations
  without an arbitrary hard cutoff.
when_to_use_negative:
- You already have a reliable biological threshold for validation and do not need
  mixture-model decomposition of score densities
- There are too few scored cells per candidate label for mclust to fit a meaningful
  mixture density
- The goal is initial candidate-label assignment — that comes from expression profile
  correlation, not from score-density decomposition
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: scMAGIC
  role: Hosts Module 3, which computes marker-gene-based per-cell confidence scores
    and applies mclust-based density decomposition to validate candidate labels
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: mclust
  role: Gaussian mixture modeling package used to decompose the density of per-cell
    confidence scores and identify the highest-confidence sub-distribution
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:5b5673acb16f649ab68926d70c1ebbba83486445f67ee247be66a0b8f584cde7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - Seurat
  - mclust
  - scMAGIC
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# gaussian-mixture-density-decomposition

## Summary

Decomposes per-cell confidence-score densities with mclust Gaussian mixture modeling to separate confidently validated query cells (highest-confidence sub-distribution) from low-confidence cells during scMAGIC's first-round Module 3 validation.

## When to use

Use when a population of per-cell confidence scores (e.g., Wilcoxon rank-sum scores on marker genes) must be split into validated vs. unvalidated sub-populations without an arbitrary hard cutoff.

## When NOT to use

- You already have a reliable biological threshold for validation and do not need mixture-model decomposition of score densities
- There are too few scored cells per candidate label for mclust to fit a meaningful mixture density
- The goal is initial candidate-label assignment — that comes from expression profile correlation, not from score-density decomposition

## Inputs

- Per-cell confidence score vector (negatively log-transformed Wilcoxon rank-sum P-values on candidate label marker genes)
- Candidate cell-label assignments for query cells from the first-round scMAGIC RBC (expression profile correlation)

## Outputs

- Set of confidently validated query cells (highest-confidence sub-distribution per candidate label)
- Low-confidence negative-control query cells per local reference cell label (score < 5)
- Local reference expression count matrix (one cell group per validated label) and negative control expression matrix

## How to apply

First compute a per-cell confidence score for each query cell's candidate reference label — in scMAGIC's Module 3 this is done via Wilcoxon rank-sum tests on the candidate label's marker genes, with P-values negatively log-transformed. Then fit a Gaussian mixture model to the density of these scores using mclust (installed via install.packages('mclust')), letting the mixture decomposition identify the sub-distribution of highest-confidence cells; cells falling in that sub-distribution are treated as confidently validated. Cells with very low scores (score < 5) are routed as negative controls for the corresponding local reference cell label. The rationale, per the article, is that a strict selection criterion ensures the confidently validated query cells are of very high quality, so they can serve as the new (local) reference for the second-round RBC and eliminate batch effects between reference and query.

## Related tools

- **scMAGIC** (Hosts Module 3, which computes marker-gene-based per-cell confidence scores and applies mclust-based density decomposition to validate candidate labels) — https://github.com/TianLab-Bioinfo/scMAGIC
- **mclust** (Gaussian mixture modeling package used to decompose the density of per-cell confidence scores and identify the highest-confidence sub-distribution)

## Examples

```
install.packages('mclust'); fit <- mclust::densityMclust(confidence_scores); validated <- names(confidence_scores)[fit$classification %in% which.max(fit$parameters$mean)]
```

## Evaluation signals

- Validated cells are restricted to cells falling in the highest-confidence sub-distribution identified by the mclust mixture fit, not an arbitrary percentile
- Cells with confidence score < 5 appear in the negative control matrix, not the local reference matrix
- The local reference expression count matrix contains only labels supported by at least min_cell validated query cells
- Saved local_reference_matrix.rds and negative_control_matrix.rds exist, load via readRDS, and the script runs end-to-end without errors

## Limitations

- Cells with confidence scores below 5 are discarded from the validated set and become negative controls, so ambiguous cells carry no vote in the second-round reference
- Labels lacking sufficient validated cells (below min_cell) cannot contribute to the local reference expression matrix
- The strict selection criterion trades recall for precision: only a subset of query cells is validated per candidate label

## Evidence

- [methods] score densities are decomposed with mclust so cells in the highest-confidence sub-distribution are validated: [evidence span withheld — anchor status: not_found]
- [discussion] our strict selection criterion has ensured that the confidently validated query cells are of very high quality: "our strict selection criterion has ensured that the confidently validated query cells are of very high quality"
- [methods] verify its cell identity by using the corresponding marker genes of the reference cell type: "verify its cell identity by using the corresponding marker genes of the reference cell type"
- [other] mclust installed via install.packages: [evidence span withheld — anchor status: not_found]
