---
name: single-cell-rna-seq-cell-type-annotation
description: Use when a query scRNA-seq dataset needs automated cell-type labels against
  a labeled scRNA-seq/bulk reference, especially when query cell types may not be
  fully covered by the reference or significant batch effects separate reference and
  query (e.g. SMART-seq reference vs Drop-seq query).
when_to_use_negative:
- The query dataset is bulk RNA-seq or non-expression data (e.g. ATAC, CITE-seq ADT)
  where expression-profile correlation and marker gene scoring are not applicable
  as designed
- No labeled reference expression matrix exists and no atlas (MCA/HCL) matches the
  organism — scMAGIC_atlas cannot produce reliable annotations without an atlas reference
- Cell labels already exist and are trusted; re-classification with RBC would be redundant
  rather than informative
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMAGIC
  role: Core two-round RBC classifier (scMAGIC and scMAGIC_atlas entry points)
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: Seurat
  role: Preprocessing (NormalizeData, FindVariableFeatures, ScaleData) and UMAP visualization
    of true vs predicted labels; Seurat-object interface via scMAGIC_Seurat
- name: SciBet
  role: Baseline statistical-metric-based classifier used in benchmark comparisons;
    also provides the SciBet_R HVGene method for Windows
  repo: https://github.com/PaulingLiu/scibet
- name: SingleR
  role: Competing statistical-metric-based RBC method run at optimal parameters in
    the 86-test benchmark
  repo: https://github.com/dviraran/SingleR
- name: scmap
  role: Competing statistical-metric-based RBC method in the benchmark
- name: CHETAH
  role: Competing tree-based RBC method in the benchmark
  repo: https://github.com/jdekanter/CHETAH
- name: COSG
  role: Marker gene tool installed from genecell/COSGR as a scMAGIC dependency
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:aecb93c15e7f830b586958a8cd8f75655a9d0e114cff1a2efc9e7c8c6a0acc46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - AUCell
  - CHETAH
  - COSG
  - CaSTLe
  - SVMrejection
  - SciBet
  - Seurat
  - SingleCellNet
  - SingleR
  - homologene
  - mclust
  - parallel
  - pcaPP
  - scClassify
  - scID
  - scMAGIC
  - scPred
  - scmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6086956521739131
n_claims: 23
---

# single-cell-rna-seq-cell-type-annotation

## Summary

Annotate query scRNA-seq cells against a well-annotated reference using scMAGIC, which identifies a candidate reference cell type by expression-profile correlation, validates identity via reference cell-type marker genes, and runs a second-round RBC using confidently validated query cells as a new reference to eliminate reference-query batch effects.

## When to use

Use when a query scRNA-seq dataset needs automated cell-type labels against a labeled scRNA-seq/bulk reference, especially when query cell types may not be fully covered by the reference or significant batch effects separate reference and query (e.g. SMART-seq reference vs Drop-seq query).

## When NOT to use

- The query dataset is bulk RNA-seq or non-expression data (e.g. ATAC, CITE-seq ADT) where expression-profile correlation and marker gene scoring are not applicable as designed
- No labeled reference expression matrix exists and no atlas (MCA/HCL) matches the organism — scMAGIC_atlas cannot produce reliable annotations without an atlas reference
- Cell labels already exist and are trusted; re-classification with RBC would be redundant rather than informative

## Inputs

- Query scRNA-seq expression matrix (gene x cell; e.g. list.demo$mat_exp from Campbell_2k.Rdata)
- Well-annotated reference expression matrix and cell-type labels (e.g. Tasic.Rdata SMART-seq mouse primary visual cortex, 1727 cells)
- Optional atlas expression matrix or built-in atlas name ('MCA' or 'HCL') for marker gene identification
- Seurat object (counts) for the Seurat-compatible interface

## Outputs

- Per-cell predicted cell-type labels: output.scMAGIC$scMAGIC.tag
- Annotated Seurat object (seurat.query$pred.tag) from scMAGIC_Seurat
- Confusion matrix table(label_sc, pred.scMAGIC) from which accuracy/balanced accuracy/cluster accuracy are computed

## How to apply

Load the reference (.Rdata list.Ref with mat_exp and label[,1]) and query expression matrix, then call scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA' or 'HCL', num_threads = 4-10); the atlas argument enables atlas-informed marker gene identification and is essential when query contains cell types uncovered by the reference. Extract predictions from output.scMAGIC$scMAGIC.tag and quantify accuracy by cross-tabulating table(label_sc, pred.scMAGIC). The default two-round pipeline uses strictly-selected, confidently validated query cells from round one as a local reference in round two, which is what overcomes batch effects (e.g. accuracy 0.98 and 0.85 vs 0.33 and 0.34 for single_round mode on the Tasic/Campbell low-depth and cross-species benchmarks). For Windows, set method_HVGene = 'SciBet_R'; reticulate/numpy must be available since scMAGIC imports numpy. If no reference exists, use scMAGIC_atlas(exp_sc_mat, MCA_ref, atlas = 'MCA', type_ref = 'sum-counts', use_RUVseq = F, min_cell = 5, num_threads = 10) with MCA or HCL as reference. Inputs/outputs are also compatible with Seurat objects via scMAGIC_Seurat(seurat.query, seurat.ref).

## Related tools

- **scMAGIC** (Core two-round RBC classifier (scMAGIC and scMAGIC_atlas entry points)) — https://github.com/TianLab-Bioinfo/scMAGIC
- **Seurat** (Preprocessing (NormalizeData, FindVariableFeatures, ScaleData) and UMAP visualization of true vs predicted labels; Seurat-object interface via scMAGIC_Seurat)
- **SciBet** (Baseline statistical-metric-based classifier used in benchmark comparisons; also provides the SciBet_R HVGene method for Windows) — https://github.com/PaulingLiu/scibet
- **SingleR** (Competing statistical-metric-based RBC method run at optimal parameters in the 86-test benchmark) — https://github.com/dviraran/SingleR
- **scmap** (Competing statistical-metric-based RBC method in the benchmark)
- **CHETAH** (Competing tree-based RBC method in the benchmark) — https://github.com/jdekanter/CHETAH
- **COSG** (Marker gene tool installed from genecell/COSGR as a scMAGIC dependency)

## Examples

```
output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4); pred.scMAGIC <- output.scMAGIC$scMAGIC.tag
```

## Evaluation signals

- Accuracy computed via table(label_sc, pred.scMAGIC) against true query labels; benchmark two-round accuracies of 0.98 (low-depth reference) and 0.85 (cross-species) are the published reference points
- Two-round run must substantially outperform single_round mode when batch effects exist; a drop like 0.98→0.33 signals the second round was skipped or failed
- With atlas-assisted marker genes, accuracy on uncovered-cell-type benchmarks should be clearly higher than scMAGICno-Atlas (omitting the atlas argument)
- Across the 86 benchmark tests, scMAGIC attains a median accuracy rank of 1, vs rank 4 (Seurat v4) and rank 5 (scSemiCluster)
- Predicted labels should populate seurat.query@meta.data$pred.tag and reproduce UMAP concordance with original labels in the README tutorial

## Limitations

- A single round of RBC collapses under significant reference-query batch effects (accuracy 0.33-0.34 vs 0.98/0.85 with two rounds), so single_round mode is only for ablation, not production
- Omitting the atlas expression matrix (scMAGICno-Atlas) markedly degrades accuracy when query cell types are not completely covered by the reference
- Windows users must set method_HVGene = 'SciBet_R' and the Python numpy module must be importable via reticulate before running scMAGIC
- No version tags or archived code releases are stated beyond the GitHub repository, and dataset accession codes are only in Materials and Methods

## Evidence

- [abstract] scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests: "scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests"
- [intro] Second-round RBC eliminates batch effects: "scMAGIC introduces a second-round RBC in which those query cells whose cell identifies are confidently validated in the first-round RBC are used as the new reference to again classify query single"
- [methods] Core scMAGIC invocation: "output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)"
- [readme] README overview of two-round mechanism: "scMAGIC conducts a second-round of RBC by using target single cells with verified cell identity as the new reference to overcome the significant batch effects"
- [discussion] Atlas-informed marker genes for uncovered types: "scMAGIC adds an atlas expression matrix when identifying the reference cell type-specific marker genes"
