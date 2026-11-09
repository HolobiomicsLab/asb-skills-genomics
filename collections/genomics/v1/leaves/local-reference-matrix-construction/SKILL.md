---
name: local-reference-matrix-construction
description: 'Use when running scMAGIC''s second-round RBC: after first-round candidate
  assignment and confidence scoring, you need to aggregate validated query cells into
  a local reference expression matrix and build negative controls per label.'
when_to_use_negative:
- Query cell types are not covered by any validated query cells for a label (below
  the min_cell requirement per local reference label), so no local reference group
  can be formed
- The task is a single-pass classification where batch effects between reference and
  query are negligible and a second-round RBC is unnecessary
- No first-round scMAGIC assignment or marker-gene confidence scores exist yet — this
  skill presupposes completed first-round validation
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scMAGIC
  role: Provides the first-round candidate assignments, marker-gene confidence scoring,
    and validated-cell aggregation that produce the local reference and negative control
    matrices
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: mclust
  role: Gaussian mixture modeling used to decompose confidence-score densities and
    identify the highest-confidence sub-distribution of validated cells
- name: Seurat
  role: Compatible input/output object format for the query data and downstream classification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:d242478a77ae34249e6bd9506c1010328eb691ce64826a9c345c4987337001e7
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# local-reference-matrix-construction

## Summary

Constructs a local reference expression count matrix from query cells whose identities were confidently validated in scMAGIC's first-round RBC, plus a per-label negative control matrix from low-confidence cells, enabling a second-round classification that eliminates reference-query batch effects.

## When to use

Use when running scMAGIC's second-round RBC: after first-round candidate assignment and confidence scoring, you need to aggregate validated query cells into a local reference expression matrix and build negative controls per label.

## When NOT to use

- Query cell types are not covered by any validated query cells for a label (below the min_cell requirement per local reference label), so no local reference group can be formed
- The task is a single-pass classification where batch effects between reference and query are negligible and a second-round RBC is unnecessary
- No first-round scMAGIC assignment or marker-gene confidence scores exist yet — this skill presupposes completed first-round validation

## Inputs

- SMART-seq reference matrix and labels (Tasic.Rdata: ref.mtx, ref.labels; 1727 mouse primary visual cortex cells)
- Drop-seq query expression matrix (Campbell_2k.Rdata: exp_sc_mat) with first-round candidate labels
- Reference cell type-specific marker genes identified with an atlas expression matrix (e.g. MCA)

## Outputs

- local_reference_matrix.rds: aggregated expression count matrix, one column/cell group per local reference cell label, built only from validated query cells
- negative_control_matrix.rds: per-label negative control expression matrix built from low-confidence query cells (score < 5)

## How to apply

After scMAGIC's first-round assignment (candidate reference cell type per query cell by expression profile correlation), compute per-cell confidence scores by Wilcoxon rank sum tests on the candidate label's marker genes, with P-values negatively log-transformed. For each candidate cell type, decompose the confidence-score density with mclust Gaussian mixture modeling so cells falling in the highest-confidence sub-distribution are validated, while cells with confidence score < 5 become negative controls. Aggregate only the confidently validated query cells into a local reference expression count matrix, one column per local reference cell label, requiring at least min_cell cells per label, and build a negative control expression matrix from low-confidence cells for each label. Save the matrices as local_reference_matrix.rds and negative_control_matrix.rds for the second-round RBC, which replaces the original reference with this local reference to eliminate batch effects.

## Related tools

- **scMAGIC** (Provides the first-round candidate assignments, marker-gene confidence scoring, and validated-cell aggregation that produce the local reference and negative control matrices) — https://github.com/TianLab-Bioinfo/scMAGIC
- **mclust** (Gaussian mixture modeling used to decompose confidence-score densities and identify the highest-confidence sub-distribution of validated cells)
- **Seurat** (Compatible input/output object format for the query data and downstream classification)

## Examples

```
list.Ref <- readRDS('Tasic.Rdata'); output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)
```

## Evaluation signals

- local_reference_matrix.rds and negative_control_matrix.rds exist and are loadable via readRDS
- Local reference matrix contains only labels supported by validated query cells, with at least min_cell cells aggregated per label
- Negative control matrix has one negative control group per local reference cell label, drawn from cells with confidence score < 5
- mclust decomposition places validated cells in the highest-confidence sub-distribution of the score density
- The R script runs end-to-end without errors

## Limitations

- Labels lacking enough confidently validated query cells (below min_cell) cannot contribute columns to the local reference, so query cell types not fully covered by the reference may be dropped
- The strict selection criterion trades recall for quality: only cells in the highest-confidence mclust sub-distribution enter the local reference
- Requires a reference dataset with known marker genes and a first-round scMAGIC run; it is not a standalone clustering or annotation method

## Evidence

- [discussion] Second-round RBC using a local reference expression matrix constructed from query cells with verified cell labels: "replacing the original reference expression matrix with a local reference expression matrix constructed from query cells with verified cell labels and redoing the classification"
- [intro] Validated query cells used as new reference to eliminate batch effects: "scMAGIC introduces a second-round RBC in which those query cells whose cell identifies are confidently validated in the first-round RBC are used as the new reference"
- [discussion] Strict selection criterion ensures high-quality validated cells: "our strict selection criterion has ensured that the confidently validated query cells are of very high quality"
- [methods] scMAGIC implemented in R, install via devtools from GitHub: "devtools::install_github("Drizzle-Zhang/scMAGIC")"
- [intro] First-round RBC validates identity using marker genes of the candidate reference cell type: "first identify a candidate reference cell type for a query cell by expression profile correlation and then validate its cell identity using the marker genes of the"
