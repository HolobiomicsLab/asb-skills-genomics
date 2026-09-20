---
name: marker-gene-based-cell-identity-validation
description: Use when annotating query scRNA-seq cells against a well-annotated reference
  and you need to verify predicted identities via reference cell type marker genes,
  or when ablating the atlas-assisted marker identification step (scMAGIC_no-Atlas)
  to quantify its contribution.
when_to_use_negative:
- No reference dataset exists at all and the query must be annotated de novo — then
  the reference-free scMAGIC_atlas route with an atlas as reference is required instead
  of reference-based marker-gene validation
- The goal is benchmarking against statistical-metric, tree-based, or machine-learning
  RBC competitors (SingleR, scmap, SciBet, CHETAH, scClassify, scPred, SingleCellNet,
  scID, CaSTLe) rather than validating identities by reference marker genes
- The query cell types are fully covered by the reference and batch effects are minimal
  — the atlas-informed marker identification step provides no measurable advantage
  (scMAGIC_no-Atlas matches full scMAGIC accuracy in that setting)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scMAGIC
  role: Implements the two-round RBC in which first-round identity is validated with
    reference cell type marker genes; atlas argument controls atlas-informed marker
    gene identification, num_threads = 4-10 for parallelization
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: Seurat
  role: Preprocessing (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, RunUMAP)
    and visualization of true versus predicted labels; Seurat objects accepted as
    input/output via scMAGIC_Seurat
- name: RUVSeq
  role: Optional removal of unwanted variation (use_RUVseq argument) during classification
- name: parallel
  role: R parallelization backend supporting the num_threads parameter used for marker-gene-based
    classification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:6b86186f4825039c92d43cf7bbcb5aaef17949d76c105745eb956bea051e5447
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - RUVSeq
  - Seurat
  - parallel
  - scMAGIC
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# marker-gene-based-cell-identity-validation

## Summary

Validates single-cell type assignments by first identifying a candidate reference cell type via expression profile correlation and then confirming identity using that cell type's marker genes, as implemented in scMAGIC's first round of reference-based classification (RBC). Including an atlas expression matrix (e.g. MCA) when identifying reference cell type-specific marker genes improves accuracy, especially when query cell types are not fully covered by the reference.

## When to use

Use when annotating query scRNA-seq cells against a well-annotated reference and you need to verify predicted identities via reference cell type marker genes, or when ablating the atlas-assisted marker identification step (scMAGIC_no-Atlas) to quantify its contribution.

## When NOT to use

- No reference dataset exists at all and the query must be annotated de novo — then the reference-free scMAGIC_atlas route with an atlas as reference is required instead of reference-based marker-gene validation
- The goal is benchmarking against statistical-metric, tree-based, or machine-learning RBC competitors (SingleR, scmap, SciBet, CHETAH, scClassify, scPred, SingleCellNet, scID, CaSTLe) rather than validating identities by reference marker genes
- The query cell types are fully covered by the reference and batch effects are minimal — the atlas-informed marker identification step provides no measurable advantage (scMAGIC_no-Atlas matches full scMAGIC accuracy in that setting)

## Inputs

- Reference scRNA-seq dataset: expression matrix plus cell-type labels in .Rdata format (e.g. Tasic.Rdata, 1727 mouse primary visual cortex SMART-seq cells)
- Query/target scRNA-seq dataset: expression matrix plus true labels in .Rdata format (e.g. Campbell_2k.Rdata, Drop-seq mouse hypothalamic arcuate-median eminence cells)
- Atlas expression matrix for marker gene identification (e.g. MCA_ref loaded via data('MCA_ref'), type_ref = 'sum-counts')
- Optionally a Seurat object for query and reference (scMAGIC_Seurat interface)

## Outputs

- Per-cell predicted labels (output.scMAGIC$scMAGIC.tag)
- Confusion matrix of predicted versus true labels via table(label_sc, pred.scMAGIC)
- Per-benchmark-test annotation accuracy values, compilable into scmagic_noatlas_accuracy.csv
- Seurat object with predicted tags in metadata (pred.tag) for UMAP visualization

## How to apply

Load the reference and query expression matrices and true labels from .Rdata files (e.g. Tasic.Rdata reference, Campbell_2k.Rdata target) with readRDS. In the first round of RBC, identify the most likely reference cell type for each query cell based on expression profile correlation, then verify the candidate identity using the marker genes of that reference cell type; scMAGIC adds an atlas expression matrix (atlas = 'MCA', type_ref = 'sum-counts') when identifying these marker genes, running with num_threads = 4-10. Extract predictions via output.scMAGIC$scMAGIC.tag. For the ablation (scMAGIC_no-Atlas), invoke scMAGIC with the atlas argument omitted and repeat across the four benchmark reference-query pairs. Tabulate predicted versus true labels with table(label_sc, pred.scMAGIC) and compute per-test accuracy as the fraction of correctly assigned cells; when the query contains cell types not covered by the reference, the no-Atlas accuracies should fall below the atlas-assisted accuracies (Figure 6C).

## Related tools

- **scMAGIC** (Implements the two-round RBC in which first-round identity is validated with reference cell type marker genes; atlas argument controls atlas-informed marker gene identification, num_threads = 4-10 for parallelization) — https://github.com/TianLab-Bioinfo/scMAGIC
- **Seurat** (Preprocessing (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, RunUMAP) and visualization of true versus predicted labels; Seurat objects accepted as input/output via scMAGIC_Seurat)
- **RUVSeq** (Optional removal of unwanted variation (use_RUVseq argument) during classification)
- **parallel** (R parallelization backend supporting the num_threads parameter used for marker-gene-based classification)

## Examples

```
output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4); pred.scMAGIC <- output.scMAGIC$scMAGIC.tag
```

## Evaluation signals

- When query cell types are fully covered by the reference, scMAGIC_no-Atlas accuracy should match full scMAGIC accuracy
- When the query includes cell types not covered by the reference, no-Atlas accuracy should be evidently lower than atlas-assisted accuracy, consistent with Figure 6C
- Predictions extracted as output.scMAGIC$scMAGIC.tag should have one label per query cell, matching the length of label_sc
- table(label_sc, pred.scMAGIC) should be well-formed (rows = true types, columns = predicted types) and per-test accuracy = correctly assigned cells / total cells
- The four compiled accuracies should be written successfully to scmagic_noatlas_accuracy.csv

## Limitations

- Without the atlas expression matrix, marker gene identification degrades and annotation accuracy drops when query cell types are not completely covered by the reference dataset
- No version numbers or archived code release are stated for the toolchain beyond the GitHub repository links, so exact reproducibility of dependency versions is not guaranteed
- Dataset accession codes and detailed provenance are deferred to Materials and Methods rather than provided in the main text

## Evidence

- [intro] first-round RBC validates candidate identities with marker genes: "first identify a candidate reference cell type for a query cell by expression profile correlation and then validate its cell identity using the marker genes of the"
- [discussion] atlas matrix is added for marker gene identification: "scMAGIC adds an atlas expression matrix when identifying the reference cell type-specific marker genes"
- [abstract] no-Atlas ablation finding: "especially when the cell types in the query dataset are not completely covered by the reference dataset and when there exist significant batch effects"
- [readme] README classification call with atlas and threads: "output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)"
- [readme] README result tabulation: "# classification results
table(label_sc, pred.scMAGIC)"
