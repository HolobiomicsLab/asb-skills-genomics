---
name: pseudobulk-aggregation-by-cell-label
description: Use when a labeled reference scRNA-seq dataset (e.g. Tasic SMART-seq,
  1727 mouse primary visual cortex cells) must be collapsed into per-cell-type expression
  profiles before testing which genes are up-regulated in each reference cell type.
when_to_use_negative:
- The reference dataset has no cell type labels, so profiles cannot be aggregated
  per type (use the atlas-only scMAGIC_atlas mode instead)
- The analysis requires single-cell resolution expression (e.g. direct expression
  profile correlation of individual query cells), where aggregation would erase per-cell
  information
- Input is already bulk RNA-seq or already pseudo-bulk reference data — no aggregation
  step is needed before limma
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3332
tools:
- name: scMAGIC
  role: 'Implements first-round Module 1: aggregates the reference count matrix by
    cell label to obtain pseudo-bulk profiles and derives marker genes from them'
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: limma
  role: Differential expression testing on the aggregated pseudo-bulk profiles to
    identify genes up-regulated in each cell type (FDR > 0.05 vs reference, FDR >
    0.001 vs atlas)
- name: RUVSeq
  role: Removes batch effects between the reference pseudo-bulk data and the atlas
    before the second limma test
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:65a30278231b6c71e1f5093a6d9432d61863be88fd44e1f984c5dff13c271b12
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
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pseudobulk-aggregation-by-cell-label

## Summary

Aggregates a single-cell count matrix by known cell type labels into pseudo-bulk profiles per reference cell type, reducing sparsity so downstream differential-expression testing (limma) can identify cell type-specific marker genes, as done in scMAGIC's first-round marker-gene module.

## When to use

Use when a labeled reference scRNA-seq dataset (e.g. Tasic SMART-seq, 1727 mouse primary visual cortex cells) must be collapsed into per-cell-type expression profiles before testing which genes are up-regulated in each reference cell type.

## When NOT to use

- The reference dataset has no cell type labels, so profiles cannot be aggregated per type (use the atlas-only scMAGIC_atlas mode instead)
- The analysis requires single-cell resolution expression (e.g. direct expression profile correlation of individual query cells), where aggregation would erase per-cell information
- Input is already bulk RNA-seq or already pseudo-bulk reference data — no aggregation step is needed before limma

## Inputs

- Reference single-cell count/expression matrix (genes x cells), e.g. Tasic SMART-seq matrix loaded via readRDS('Tasic.Rdata')
- Cell type label assignments for the reference cells (per-cell categorical labels)
- Optionally, a homologene-mapped atlas reference (MCA_ref) for the downstream second limma test

## Outputs

- Pseudo-bulk expression matrix with one aggregated profile (column) per reference cell type
- Per-cell-type highly expressed candidate gene pools (top 20% within each cell type)
- Per-cell-type marker gene lists (final output saved to marker_genes.rds)

## How to apply

Load the reference expression matrix (genes x cells) together with the cell type label vector (e.g. from Tasic.Rdata). Sum (aggregate) the raw count matrix over cells sharing the same label to produce one pseudo-bulk expression profile per reference cell type. From these aggregated profiles, select the top 20% highly expressed genes per cell type as the candidate feature pool, then run limma to test each candidate gene for up-regulation versus all other reference cell types (retaining genes passing FDR > 0.05). The rationale is that pseudo-bulk aggregation mitigates single-cell sparsity and gives the count-level expression profiles that limma requires; scMAGIC subsequently repeats an analogous test against an atlas (MCA) after batch-effect removal with RUVSeq, keeping genes with FDR > 0.001 as the final cell-type-specific markers.

## Related tools

- **scMAGIC** (Implements first-round Module 1: aggregates the reference count matrix by cell label to obtain pseudo-bulk profiles and derives marker genes from them) — https://github.com/TianLab-Bioinfo/scMAGIC
- **limma** (Differential expression testing on the aggregated pseudo-bulk profiles to identify genes up-regulated in each cell type (FDR > 0.05 vs reference, FDR > 0.001 vs atlas))
- **RUVSeq** (Removes batch effects between the reference pseudo-bulk data and the atlas before the second limma test)

## Examples

```
list.Ref <- readRDS('Tasic.Rdata'); output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)
```

## Evaluation signals

- The aggregated matrix has exactly one profile per unique reference cell type label (no empty or duplicate groups)
- Each aggregated profile equals the sum of raw counts over cells bearing that label (mass-balance invariant against the original matrix)
- Each reference cell type yields a non-empty marker gene set after both limma tests (FDR > 0.05 vs other reference types; FDR > 0.001 vs atlas cell types)
- The candidate feature pool corresponds to the top 20% highly expressed genes per cell type in the aggregated profiles
- Final per-cell-type marker lists are serializable to marker_genes.rds without missing cell types

## Limitations

- Requires a well-annotated reference with trusted per-cell labels; label errors propagate into the pseudo-bulk profiles
- Batch effects between the reference and an external atlas must be removed (via homologene mapping and RUVSeq) before cross-dataset differential tests
- Aggregation assumes cell types are discrete and pre-assigned; it does not address query cell types not fully covered by the reference (a scenario scMAGIC handles in later rounds)

## Evidence

- [other] Load the reference expression matrix and cell type labels (Tasic SMART-seq dataset) and aggregate the count matrix by cell label to obtain pseudo-bulk profiles per reference cell type.: [evidence span withheld — anchor status: not_found]
- [methods] Tasic SMART-seq reference with 1727 cells: "The reference dataset includes 1727 mouse primary visual cortex cells profiled by SMART-seq."
- [discussion] atlas used alongside reference for marker gene identification: "scMAGIC adds an atlas expression matrix when identifying the reference cell type-specific marker genes"
