---
name: highly-expressed-gene-selection
description: Use when building a cell-type-specific marker gene set from a well-annotated
  scRNA-seq reference (e.g. the Tasic SMART-seq primary visual cortex dataset) to
  validate query cell identities in reference-based classification.
when_to_use_negative:
- When the reference dataset has no cell type labels, use scMAGIC_atlas with the atlas
  as reference instead of deriving reference-cell-type markers
- When you already have a curated, hand-validated marker gene list for the reference
  cell types, re-deriving markers via limma/RUVSeq filtering is unnecessary
- When query cell types are largely absent from both the reference and the atlas,
  marker-gene validation cannot assign them and the classification will be uninformative
tools:
- name: scMAGIC
  role: 'Implements first-round RBC marker gene identification: selects highly expressed
    genes and filters them for reference cell type specificity using both reference
    and atlas comparisons'
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: limma
  role: Linear-model differential expression testing of candidate highly expressed
    genes versus other reference cell types (FDR > 0.05) and atlas cell types (FDR
    > 0.001)
- name: RUVSeq
  role: Removes batch effects between the reference dataset and the atlas before the
    atlas limma comparison
- name: homologene
  role: Maps genes between the reference dataset and the atlas (MCA_ref) so candidates
    can be tested against atlas cell types
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:1c2ecbf70f1c034b6b3508d1f025bbe68d10f3c803bd400c9758320e3d04e2cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - RUVSeq
  - homologene
  - limma
  - scMAGIC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# highly-expressed-gene-selection

## Summary

In scMAGIC's first-round RBC, candidate marker genes are drawn from the pool of highly expressed genes within each reference cell type (top 20% expression after pseudo-bulk aggregation), then filtered by differential-expression tests against both other reference cell types and atlas cell types.

## When to use

Use when building a cell-type-specific marker gene set from a well-annotated scRNA-seq reference (e.g. the Tasic SMART-seq primary visual cortex dataset) to validate query cell identities in reference-based classification.

## When NOT to use

- When the reference dataset has no cell type labels, use scMAGIC_atlas with the atlas as reference instead of deriving reference-cell-type markers
- When you already have a curated, hand-validated marker gene list for the reference cell types, re-deriving markers via limma/RUVSeq filtering is unnecessary
- When query cell types are largely absent from both the reference and the atlas, marker-gene validation cannot assign them and the classification will be uninformative

## Inputs

- Reference single-cell expression count matrix with cell type labels (e.g. Tasic SMART-seq dataset, RData/RDS)
- Atlas expression matrix used as an additional comparison panel (e.g. MCA_ref from the Mouse Cell Atlas)
- Gene-to-gene homology mapping for cross-dataset gene alignment (e.g. homologene)

## Outputs

- Per-reference-cell-type lists of cell-type-specific marker genes with dual-filter statistics (FDR > 0.05 vs reference cell types, FDR > 0.001 vs atlas cell types)
- marker_genes.rds file storing the per-cell-type marker gene lists for use in first-round RBC validation

## How to apply

Aggregate the reference count matrix by cell label to form pseudo-bulk profiles per reference cell type (e.g. the Tasic SMART-seq mouse primary visual cortex reference). Within each reference cell type, select the top 20% of genes by expression as the candidate feature pool — restricting to highly expressed genes prevents sparse, lowly expressed genes from producing spurious up-regulation calls. Test each candidate gene with limma for up-regulation versus all other reference cell types, retaining genes passing an FDR > 0.05 threshold. To ensure specificity beyond the reference panel, map genes to an atlas (e.g. MCA via homologene), apply RUVSeq to remove batch effects between reference and atlas, and run a second limma test of candidates against atlas cell types, keeping only genes with FDR > 0.001. The rationale is that scMAGIC identifies markers up-regulated with respect to both the other reference cell types and the atlas cell types, so the final marker genes are specific to a reference cell type rather than to a single dataset.

## Related tools

- **scMAGIC** (Implements first-round RBC marker gene identification: selects highly expressed genes and filters them for reference cell type specificity using both reference and atlas comparisons) — https://github.com/TianLab-Bioinfo/scMAGIC
- **limma** (Linear-model differential expression testing of candidate highly expressed genes versus other reference cell types (FDR > 0.05) and atlas cell types (FDR > 0.001))
- **RUVSeq** (Removes batch effects between the reference dataset and the atlas before the atlas limma comparison)
- **homologene** (Maps genes between the reference dataset and the atlas (MCA_ref) so candidates can be tested against atlas cell types)

## Examples

```
output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)
```

## Evaluation signals

- Each reference cell type yields a non-empty marker gene set after the two-stage limma filtering
- All retained markers pass FDR > 0.05 against the other reference cell types in the first limma test
- All retained markers pass FDR > 0.001 against atlas cell types in the second limma test
- Retained markers fall within the top 20% of highly expressed genes of their reference cell type's pseudo-bulk profile
- Output marker_genes.rds contains an entry for every reference cell type, and downstream first-round RBC classification succeeds using these markers

## Limitations

- The dual FDR filtering depends on atlas coverage; query cell types not covered by the atlas may yield over- or under-specific marker sets
- Marker identification relies on pseudo-bulk aggregation of single cells, so very small reference cell type populations may produce unreliable differential tests
- Batch effects between reference and atlas are corrected with RUVSeq; residual uncorrected batch structure could cause atlas cell types to confound marker specificity
- No archived code release or version tag for scMAGIC is stated beyond the GitHub repository link, and no version numbers are given for limma/RUVSeq

## Evidence

- [discussion] scMAGIC adds an atlas expression matrix when identifying the reference cell type-specific marker genes: "scMAGIC adds an atlas expression matrix when identifying the reference cell type-specific marker genes"
- [methods] Reference dataset description: "The reference dataset includes 1727 mouse primary visual cortex cells profiled by SMART-seq."
- [methods] Marker gene verification in first-round RBC: "verify its cell identity by using the corresponding marker genes of the reference cell type"
- [methods] First-round classification scheme: "first identify the most likely cell type in the reference for a single cell based on expression profile correlation"
