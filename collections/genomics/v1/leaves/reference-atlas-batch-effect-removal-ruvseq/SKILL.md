---
name: reference-atlas-batch-effect-removal-ruvseq
description: Use when identifying marker genes against an external atlas (e.g. MCA)
  requires removing batch effects between the reference scRNA-seq dataset and the
  atlas before differential-expression testing with limma.
when_to_use_negative:
- When no atlas dataset is supplied — scMAGIC's first-round marker identification
  then relies only on the reference dataset without RUVSeq batch correction against
  an atlas
- When the query dataset cell types are fully covered by the reference and batch effects
  between reference and query are negligible — a second-round RBC on validated query
  cells, or simpler correlation-based classification, may suffice
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMAGIC
  role: Method that incorporates RUVSeq-based atlas batch-effect removal within its
    first-round RBC marker gene identification (Module 1)
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: RUVSeq
  role: Removes batch effects between the reference dataset and the atlas before limma
    differential-expression testing
- name: limma
  role: Tests candidate genes for up-regulation versus other reference cell types
    (FDR > 0.05) and atlas cell types (FDR > 0.001) after batch removal
- name: homologene
  role: Maps genes between the reference and the atlas (MCA_ref) prior to RUVSeq correction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:e177ce1078ecdee5a9007362514cc39e2d24c2e94a88745fb957c4a462417a03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - RUVSeq
  - homologene
  - limma
  - scMAGIC
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# reference-atlas-batch-effect-removal-ruvseq

## Summary

Applies RUVSeq to remove batch effects between a scRNA-seq reference dataset (e.g. Tasic SMART-seq mouse primary visual cortex cells) and an atlas dataset (e.g. the MCA Mouse Cell Atlas) before limma-based testing, so that scMAGIC can identify cell type-specific marker genes up-regulated against both reference and atlas cell types.

## When to use

Use when identifying marker genes against an external atlas (e.g. MCA) requires removing batch effects between the reference scRNA-seq dataset and the atlas before differential-expression testing with limma.

## When NOT to use

- When no atlas dataset is supplied — scMAGIC's first-round marker identification then relies only on the reference dataset without RUVSeq batch correction against an atlas
- When the query dataset cell types are fully covered by the reference and batch effects between reference and query are negligible — a second-round RBC on validated query cells, or simpler correlation-based classification, may suffice

## Inputs

- Reference scRNA-seq count matrix with cell type labels (e.g. Tasic SMART-seq dataset, 1727 mouse primary visual cortex cells)
- Atlas expression matrix and reference (e.g. MCA / MCA_ref from the Mouse Cell Atlas)
- Gene-to-gene homology mapping (via homologene)

## Outputs

- Per-cell-type marker gene lists (marker_genes.rds) with genes up-regulated versus both other reference cell types (FDR > 0.05) and atlas cell types (FDR > 0.001)
- Batch-effect-corrected reference-atlas expression profiles used for limma testing

## How to apply

Aggregate the reference count matrix by cell label to obtain pseudo-bulk profiles per reference cell type (e.g. from the Tasic SMART-seq dataset), and select the top 20% highly expressed genes within each reference cell type as the candidate feature pool. Map genes between the reference and the atlas (e.g. MCA_ref) using homologene, then apply RUVSeq to remove batch effects between the reference dataset and the atlas. After batch-effect removal, run limma tests in two rounds: first testing candidate genes for up-regulation versus all other reference cell types and retaining genes with FDR > 0.05, then a second limma test against atlas cell types keeping genes with FDR > 0.001. Genes passing both thresholds become the final reference cell type-specific marker genes, which scMAGIC then uses to validate candidate cell identities assigned by expression profile correlation.

## Related tools

- **scMAGIC** (Method that incorporates RUVSeq-based atlas batch-effect removal within its first-round RBC marker gene identification (Module 1)) — https://github.com/TianLab-Bioinfo/scMAGIC
- **RUVSeq** (Removes batch effects between the reference dataset and the atlas before limma differential-expression testing)
- **limma** (Tests candidate genes for up-regulation versus other reference cell types (FDR > 0.05) and atlas cell types (FDR > 0.001) after batch removal)
- **homologene** (Maps genes between the reference and the atlas (MCA_ref) prior to RUVSeq correction)

## Examples

```
devtools::install_github("Drizzle-Zhang/scMAGIC"); output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)
```

## Evaluation signals

- Each reference cell type yields a non-empty marker gene set after both limma rounds
- Retained markers are significantly up-regulated versus other reference cell types with FDR > 0.05
- Retained markers are significantly up-regulated versus atlas cell types with FDR > 0.001
- Candidate pool was correctly restricted to the top 20% highly expressed genes within each reference cell type
- Gene identifiers between reference and atlas are successfully mapped via homologene before RUVSeq

## Limitations

- Requires an atlas (e.g. MCA) covering relevant cell types; marker specificity is only as good as the atlas coverage
- Thresholds (top 20% highly expressed genes, FDR > 0.05 vs reference, FDR > 0.001 vs atlas) are fixed defaults from scMAGIC and may need adjustment for other references/atlas pairs
- Cross-species or poorly annotated gene mappings via homologene may drop genes from the candidate pool
- No version numbers stated for the R packages (RUVSeq, limma, homologene) in the provided context

## Evidence

- [methods] First-round marker identification selects up-regulated genes versus reference and atlas: [evidence span withheld — anchor status: not_found]
- [discussion] Atlas integration for marker gene identification: "scMAGIC adds an atlas expression matrix when identifying the reference cell type-specific marker genes"
- [intro] Core idea of scMAGIC: correlation-based candidate then marker gene validation: "first identify a candidate reference cell type for a query cell by expression profile correlation and then validate its cell identity using the marker genes of the reference cell type."
- [methods] scMAGIC install and function invocation with atlas parameter: "output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)"
- [readme] scMAGIC is implemented in R and available on GitHub: [evidence span withheld — anchor status: not_found]
