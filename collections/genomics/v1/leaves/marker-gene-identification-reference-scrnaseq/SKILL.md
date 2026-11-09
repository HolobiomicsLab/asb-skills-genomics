---
name: marker-gene-identification-reference-scrnaseq
description: Use when building a reference-based single-cell classifier that needs
  cell type-specific marker genes validated not only within the reference dataset
  but also against an independent atlas (e.g. MCA), so markers are specific across
  a broader cell-type space.
when_to_use_negative:
- No independent atlas dataset is available and marker specificity needs only be assessed
  within the reference itself
- The goal is annotating query cells directly from an atlas without a curated reference
  (use scMAGIC_atlas instead of first-round marker identification)
- Candidate genes have not been filtered to highly expressed genes in the reference
  cell type — the limma tests are defined on the top 20% expression pool
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scMAGIC
  role: Encapsulates the first-round marker-gene identification module that selects
    reference cell type-specific markers using both reference and atlas datasets
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: limma
  role: Differential-expression testing of candidate genes up-regulated in each reference
    cell type versus other reference types and atlas cell types
- name: RUVSeq
  role: Removes batch effects between the reference dataset and the atlas before the
    second limma test against atlas cell types
- name: homologene
  role: Maps genes between the reference dataset and the atlas reference (MCA_ref)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:aaa3c66ab2345c270797a23aa1c3148d3556de2cddcdffca067ca58dc96e5b59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - RUVSeq
  - homologene
  - limma
  - scMAGIC
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# marker-gene-identification-reference-scRNAseq

## Summary

Identifies reference cell type-specific marker genes in scMAGIC's first-round RBC by testing highly expressed reference genes with limma for up-regulation versus both other reference cell types (FDR > 0.05) and atlas cell types (FDR > 0.001), after removing reference–atlas batch effects with RUVSeq.

## When to use

Use when building a reference-based single-cell classifier that needs cell type-specific marker genes validated not only within the reference dataset but also against an independent atlas (e.g. MCA), so markers are specific across a broader cell-type space.

## When NOT to use

- No independent atlas dataset is available and marker specificity needs only be assessed within the reference itself
- The goal is annotating query cells directly from an atlas without a curated reference (use scMAGIC_atlas instead of first-round marker identification)
- Candidate genes have not been filtered to highly expressed genes in the reference cell type — the limma tests are defined on the top 20% expression pool

## Inputs

- Reference scRNA-seq expression matrix with cell type labels (e.g. Tasic SMART-seq dataset, Tasic.Rdata)
- Reference cell type label vector per cell
- Atlas reference expression matrix (e.g. MCA_ref from the Mouse Cell Atlas)

## Outputs

- Per-reference-cell-type marker gene lists with significant up-regulation vs other reference types (FDR > 0.05) and atlas cell types (FDR > 0.001)
- marker_genes.rds file of marker gene lists

## How to apply

Aggregate the reference count matrix (e.g. Tasic SMART-seq mouse primary visual cortex, 1727 cells) by cell label to pseudo-bulk profiles per reference cell type, then select the top 20% highly expressed genes in each type as the candidate pool. Run limma tests of each candidate gene against all other reference cell types and retain genes passing FDR > 0.05. Map genes between reference and atlas (e.g. MCA) using homologene and apply RUVSeq to remove batch effects between the reference dataset and the atlas. Perform a second limma test of the candidates against atlas cell types, keeping genes with FDR > 0.001 as the final cell-type-specific marker genes. Rationale: requiring up-regulation versus both the reference and the atlas prevents markers that are shared with unrepresented cell types, which is why scMAGIC adds an atlas expression matrix when identifying reference cell type-specific marker genes.

## Related tools

- **scMAGIC** (Encapsulates the first-round marker-gene identification module that selects reference cell type-specific markers using both reference and atlas datasets) — https://github.com/TianLab-Bioinfo/scMAGIC
- **limma** (Differential-expression testing of candidate genes up-regulated in each reference cell type versus other reference types and atlas cell types)
- **RUVSeq** (Removes batch effects between the reference dataset and the atlas before the second limma test against atlas cell types)
- **homologene** (Maps genes between the reference dataset and the atlas reference (MCA_ref))

## Examples

```
output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)
```

## Evaluation signals

- Each reference cell type yields a non-empty marker gene set
- Every retained marker passes FDR > 0.05 in the limma test versus other reference cell types
- Every retained marker passes FDR > 0.001 in the second limma test versus atlas cell types
- Candidates were drawn from the top 20% highly expressed genes of each reference cell type
- The per-cell-type marker lists are saved to marker_genes.rds and loadable as an RDS object

## Limitations

- Marker specificity depends on atlas coverage: the mouse duodenum case shows a query dataset may include cell types not covered by the atlas
- Requires gene mapping (homologene) between reference and atlas; unmapped genes are dropped from cross-dataset testing
- Batch effects between reference and atlas must be handled by RUVSeq; residual batch effects could affect the FDR > 0.001 atlas test

## Evidence

- [methods] two-stage limma selection with atlas comparison: [evidence span withheld — anchor status: not_found]
- [methods] RUVSeq batch removal and atlas FDR threshold: [evidence span withheld — anchor status: not_found]
- [results] finding: dual-context up-regulation defines markers: [evidence span withheld — anchor status: not_found]
- [discussion] article discussion: atlas-augmented marker identification: "scMAGIC adds an atlas expression matrix when identifying the reference cell type-specific marker genes"
