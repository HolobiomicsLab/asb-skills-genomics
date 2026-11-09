---
name: marker-gene-wilcoxon-confidence-scoring
description: Use when query cells have candidate reference cell-type assignments from
  expression profile correlation and you need to verify identity using the reference
  cell type's marker genes before constructing a local reference for reclassification.
when_to_use_negative:
- Query cell types are not represented in the reference at all — no candidate marker
  genes exist to score against
- Only raw counts are available without any first-round candidate labels from expression-profile-correlation
  classification
- The goal is a full scMAGIC annotation run including Seurat integration (scMAGIC_Seurat)
  rather than extracting the verified local reference for a second-round RBC
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMAGIC
  role: Implements the two-round reference-based classification whose first-round
    candidate labels are validated by marker-gene Wilcoxon confidence scoring before
    the second-round RBC
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: mclust
  role: Gaussian mixture modeling package used to decompose confidence score densities
    and select cells in the highest-confidence sub-distribution
- name: Seurat
  role: Compatible input/output format for query objects preprocessed with NormalizeData,
    FindVariableFeatures, ScaleData, RunPCA, RunUMAP
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:67fc56e0020b820d4cdc1131052fd487e08d62fdc321f2d2d8bd062b06d6e47d
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# marker-gene-wilcoxon-confidence-scoring

## Summary

Validates candidate cell labels from a first-round reference-based classification by scoring each query cell with Wilcoxon rank-sum tests on the candidate reference cell type's marker genes, then uses mixture-model decomposition of score densities (mclust) to identify confidently validated cells that feed scMAGIC's second-round RBC.

## When to use

Use when query cells have candidate reference cell-type assignments from expression profile correlation and you need to verify identity using the reference cell type's marker genes before constructing a local reference for reclassification.

## When NOT to use

- Query cell types are not represented in the reference at all — no candidate marker genes exist to score against
- Only raw counts are available without any first-round candidate labels from expression-profile-correlation classification
- The goal is a full scMAGIC annotation run including Seurat integration (scMAGIC_Seurat) rather than extracting the verified local reference for a second-round RBC

## Inputs

- Query expression count matrix (e.g. exp_sc_mat from Drop-seq Campbell_2k.Rdata, loaded via readRDS)
- Reference expression matrix and labels (ref.mtx, ref.labels from SMART-seq Tasic.Rdata, loaded via readRDS)
- First-round candidate reference cell type assignments per query cell (from expression profile correlation)
- Marker genes of the reference cell types (identified with an atlas expression matrix such as MCA)

## Outputs

- Per-cell confidence scores (negative log-transformed Wilcoxon rank-sum test P-values)
- Set of confidently validated query cells per label (mclust highest-confidence sub-distribution)
- Local reference expression count matrix (one column per local reference cell label, min_cell cells per label)
- Per-label negative control expression matrix from low-confidence query cells (score < 5)
- Saved RDS outputs: local_reference_matrix.rds and negative_control_matrix.rds

## How to apply

After the first round of reference-based classification assigns each query cell a candidate reference cell type, verify its identity by running Wilcoxon rank-sum tests on the corresponding marker genes of that reference cell type, negatively log-transforming the P-values into a per-cell confidence score. For each candidate cell type, decompose the density of confidence scores with mclust (Gaussian mixture modeling) so that cells falling in the highest-confidence sub-distribution are validated. Low-confidence cells (score < 5) are diverted as negative controls, and only validated query cells are aggregated — requiring at least min_cell cells per label — into the local reference expression count matrix used by the second-round RBC, which eliminates batch effects between the original reference and the query data. The strictness of this selection criterion is what guarantees the verified target cells used as the new reference are of very high quality.

## Related tools

- **scMAGIC** (Implements the two-round reference-based classification whose first-round candidate labels are validated by marker-gene Wilcoxon confidence scoring before the second-round RBC) — https://github.com/TianLab-Bioinfo/scMAGIC
- **mclust** (Gaussian mixture modeling package used to decompose confidence score densities and select cells in the highest-confidence sub-distribution)
- **Seurat** (Compatible input/output format for query objects preprocessed with NormalizeData, FindVariableFeatures, ScaleData, RunPCA, RunUMAP)

## Examples

```
output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)
```

## Evaluation signals

- Saved RDS files (local_reference_matrix.rds, negative_control_matrix.rds) exist and load via readRDS without errors
- Local reference matrix contains only labels supported by validated cells, with at least min_cell cells aggregated per label
- Validated cells fall in the highest-confidence mclust sub-distribution while negative controls have confidence scores below 5
- Script runs end-to-end without errors on the Tasic.Rdata reference and Campbell_2k.Rdata query inputs
- Second-round RBC on the local reference removes batch effects versus the original reference (e.g. improved annotation of Drop-seq query cells)

## Limitations

- Unclassifiable query cells can be misassigned; scMAGIC's authors note competing methods like sciBet and scSemiCluster wrongly assign all unclassifiable query cells, motivating the strict validation criterion
- Validation depends on marker genes being specific to reference cell types, which requires adding an atlas expression matrix (e.g. MCA) during marker gene identification
- Labels lacking at least min_cell validated query cells cannot contribute to the local reference matrix

## Evidence

- [methods] first-round candidate identification: "first identify the most likely cell type in the reference for a single cell based on expression profile correlation"
- [methods] identity verification by marker genes: "verify its cell identity by using the corresponding marker genes of the reference cell type"
- [discussion] strict selection rationale: "our strict selection criterion has ensured that the confidently validated query cells are of very high quality"
- [methods] second-round local reference: "conducts a second-round of RBC by using target single cells with verified cell identity as the new reference"
- [methods] package install: "devtools::install_github("Drizzle-Zhang/scMAGIC")"
