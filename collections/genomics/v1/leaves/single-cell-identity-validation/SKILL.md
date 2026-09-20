---
name: single-cell-identity-validation
description: Use when a reference-based classifier (e.g. scMAGIC first round) has
  produced candidate labels per query cell and you must separate confidently validated
  cells from low-confidence negative controls before retraining or reclassification.
when_to_use_negative:
- The query cell types are already fully covered by a high-quality reference and no
  batch effects exist between reference and query datasets
- You only need a first-round annotation and do not intend to run a second-round RBC
  with validated query cells as the new reference
- Any candidate local reference cell label is supported by fewer than min_cell validated
  query cells
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMAGIC
  role: Performs the first-round RBC by expression profile correlation whose candidate
    labels this skill validates; implemented in R
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: mclust
  role: Gaussian mixture modeling used to decompose confidence-score densities and
    identify the highest-confidence sub-distribution of validated cells
- name: Seurat
  role: Preprocessing and compatibility layer for the query object in the surrounding
    workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:885f6763addb9d9fb29c37a5bd044ba2b85850aa375c3b50574c56ba877232a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - Seurat
  - mclust
  - scMAGIC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# single-cell-identity-validation

## Summary

Validates first-round candidate cell labels in scMAGIC (Module 3) by scoring each query cell with Wilcoxon rank-sum tests on the candidate reference cell type's marker genes, decomposing confidence-score densities with mclust, and using confidently validated cells to build the local reference and per-label negative control expression matrices consumed by the second-round RBC.

## When to use

Use when a reference-based classifier (e.g. scMAGIC first round) has produced candidate labels per query cell and you must separate confidently validated cells from low-confidence negative controls before retraining or reclassification.

## When NOT to use

- The query cell types are already fully covered by a high-quality reference and no batch effects exist between reference and query datasets
- You only need a first-round annotation and do not intend to run a second-round RBC with validated query cells as the new reference
- Any candidate local reference cell label is supported by fewer than min_cell validated query cells

## Inputs

- First-round scMAGIC assignment (candidate reference cell type per query cell from expression profile correlation)
- Reference dataset (Tasic.Rdata: ref.mtx, ref.labels; SMART-seq mouse primary visual cortex, 1727 cells)
- Query dataset (Campbell_2k.Rdata: exp_sc_mat, label_sc; Drop-seq mouse hypothalamic arcuate-median eminence cells)
- Reference cell type-specific marker genes

## Outputs

- local_reference_matrix.rds: local reference expression count matrix aggregating validated query cells, one column per local reference cell label
- negative_control_matrix.rds: per-label negative control expression matrix built from low-confidence query cells (score < 5)

## How to apply

After the first-round scMAGIC assignment (candidate reference cell type per query cell from expression profile correlation), run Wilcoxon rank-sum tests on the corresponding reference marker genes and negatively log-transform P-values to obtain a per-cell confidence score. For each candidate cell type, decompose the density of confidence scores with mclust (Gaussian mixture modeling, install.packages('mclust')) so that cells in the highest-confidence sub-distribution are validated. Cells with score < 5 are designated negative controls; only validated query cells are aggregated into a local reference expression count matrix, one column per local reference cell label, requiring at least min_cell cells per label. This strict selection criterion ensures the new reference is of very high quality, which lets the second-round RBC eliminate batch effects between the original reference and query data. Save the two matrices as local_reference_matrix.rds and negative_control_matrix.rds.

## Related tools

- **scMAGIC** (Performs the first-round RBC by expression profile correlation whose candidate labels this skill validates; implemented in R) — https://github.com/TianLab-Bioinfo/scMAGIC
- **mclust** (Gaussian mixture modeling used to decompose confidence-score densities and identify the highest-confidence sub-distribution of validated cells)
- **Seurat** (Preprocessing and compatibility layer for the query object in the surrounding workflow)

## Examples

```
output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)
```

## Evaluation signals

- local_reference_matrix.rds and negative_control_matrix.rds both exist and are loadable via readRDS
- The local reference matrix contains only labels supported by at least min_cell validated query cells
- Validated cells correspond to the highest-confidence mclust sub-distribution; negative controls all have confidence score < 5
- The validation script runs end-to-end without errors on Tasic.Rdata reference and Campbell_2k.Rdata query data
- Using the local reference in the second-round RBC reproduces the batch-effect-free classification reported by scMAGIC

## Limitations

- Low-confidence cells (score < 5) are excluded from the local reference, so labels lacking enough validated cells cannot be represented
- Validation depends on quality and coverage of reference cell type-specific marker genes; the article notes query cell types may not be completely covered by the reference dataset
- No version numbers or archived code release for scMAGIC beyond the GitHub repository link are stated

## Evidence

- [abstract] confidently validated cells become new reference: "those query cells whose cell identities are confidently validated in the first round are used as a new reference to again classify query cells"
- [methods] first-round candidate assignment: "first identify the most likely cell type in the reference for a single cell based on expression profile correlation"
- [discussion] strict selection criterion: "our strict selection criterion has ensured that the confidently validated query cells are of very high quality"
- [discussion] local reference matrix construction: "replacing the original reference expression matrix with a local reference expression matrix constructed from query cells with verified cell labels"
- [methods] scMAGIC invocation from methods: "output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)"
