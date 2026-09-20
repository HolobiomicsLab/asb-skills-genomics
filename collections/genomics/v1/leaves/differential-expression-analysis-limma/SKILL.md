---
name: differential-expression-analysis-limma
description: Use when a reference-based cell classifier needs marker genes that are
  specific to a reference cell type not only against sibling reference types but also
  against cell types present in an external atlas (e.g. MCA).
when_to_use_negative:
- No atlas dataset is available — scMAGIC then falls back to annotating with the atlas
  alone via scMAGIC_atlas instead of marker identification against a reference
- The goal is classification of query cells itself rather than marker gene discovery
  for the reference
- Reference cell types are already fully covered and validated; the two-stage limma-plus-RUVSeq
  specificity filter is designed for the batch-effect and incomplete-coverage settings
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: limma
  role: 'Moderated differential expression testing, run twice: candidates up-regulated
    versus other reference cell types (FDR > 0.05) and then versus atlas cell types
    (FDR > 0.001)'
- name: RUVSeq
  role: Removes batch effects between the reference dataset and the MCA atlas before
    the second limma test
- name: homologene
  role: Maps genes between the reference dataset and the atlas prior to batch correction
- name: scMAGIC
  role: Hosts the marker-identification module that consumes the per-cell-type marker
    lists for first-round RBC identity validation
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:982a7271ef701745931f65034dd7f29099df8fe935e4321d34d28edaec5f5ba0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - RUVSeq
  - homologene
  - limma
  - scMAGIC
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# differential-expression-analysis-limma

## Summary

Uses limma moderated two-group testing, run twice with RUVSeq batch correction in between, to identify reference cell type-specific marker genes for scMAGIC's first-round RBC: genes must be up-regulated versus both other reference cell types (FDR > 0.05) and MCA atlas cell types (FDR > 0.001).

## When to use

Use when a reference-based cell classifier needs marker genes that are specific to a reference cell type not only against sibling reference types but also against cell types present in an external atlas (e.g. MCA).

## When NOT to use

- No atlas dataset is available — scMAGIC then falls back to annotating with the atlas alone via scMAGIC_atlas instead of marker identification against a reference
- The goal is classification of query cells itself rather than marker gene discovery for the reference
- Reference cell types are already fully covered and validated; the two-stage limma-plus-RUVSeq specificity filter is designed for the batch-effect and incomplete-coverage settings

## Inputs

- Reference scRNA-seq expression matrix with cell type labels (e.g. Tasic SMART-seq dataset, Tasic.Rdata)
- Reference cell type label vector/metadata for aggregating cells into pseudo-bulk profiles
- Atlas expression matrix reference (MCA_ref, Mouse Cell Atlas)

## Outputs

- Per-cell-type marker gene lists saved to marker_genes.rds
- limma test statistics with FDR values for reference-vs-reference and reference-vs-atlas contrasts

## How to apply

Aggregate the reference count matrix (Tasic SMART-seq, 1727 mouse primary visual cortex cells) by cell label to pseudo-bulk profiles per reference cell type, and select the top 20% highly expressed genes in each type as the candidate pool. Run limma to test each candidate for up-regulation versus all other reference cell types, retaining genes with FDR > 0.05. Then load the MCA atlas reference (MCA_ref), map genes between reference and atlas via homologene, and apply RUVSeq to remove batch effects between the reference dataset and the atlas. Run a second limma test of the surviving candidates against atlas cell types and keep genes with FDR > 0.001 as the final cell-type-specific markers. The rationale is that requiring up-regulation against both reference siblings and atlas cell types guards against markers that are actually shared with unannotated atlas cell types, improving first-round RBC identity validation.

## Related tools

- **limma** (Moderated differential expression testing, run twice: candidates up-regulated versus other reference cell types (FDR > 0.05) and then versus atlas cell types (FDR > 0.001))
- **RUVSeq** (Removes batch effects between the reference dataset and the MCA atlas before the second limma test)
- **homologene** (Maps genes between the reference dataset and the atlas prior to batch correction)
- **scMAGIC** (Hosts the marker-identification module that consumes the per-cell-type marker lists for first-round RBC identity validation) — https://github.com/TianLab-Bioinfo/scMAGIC

## Examples

```
output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)
```

## Evaluation signals

- Each reference cell type yields a non-empty marker gene set after the two limma stages
- Retained markers pass FDR > 0.05 against other reference cell types and FDR > 0.001 against atlas cell types
- Candidate pool restricted to the top 20% highly expressed genes within each reference cell type before testing
- Gene identifiers are consistently mapped between reference and atlas after homologene mapping and RUVSeq correction
- Output file marker_genes.rds contains per-cell-type lists loadable by the scMAGIC classification module

## Limitations

- Marker specificity depends on atlas coverage; cell types absent from the atlas may evade the specificity filter
- Thresholds (top 20% expression, FDR > 0.05, FDR > 0.001) are tuned for the Tasic/MCA benchmark setting and may need adjustment for other references
- Batch effect removal by RUVSeq between reference and atlas relies on adequate gene mapping via homologene

## Evidence

- [methods] two-stage limma selection of markers: [evidence span withheld — anchor status: not_found]
- [discussion] atlas-based specificity filter: "scMAGIC adds an atlas expression matrix when identifying the reference cell type-specific marker genes"
- [methods] candidate pool definition: [evidence span withheld — anchor status: not_found]
- [abstract] tool scope of the method: "scMAGIC (Single Cell annotation using MArker Genes Identification and two rounds of reference-based Classification [RBC]), a novel method"
- [methods] reference dataset composition: "The reference dataset includes 1727 mouse primary visual cortex cells profiled by SMART-seq."
