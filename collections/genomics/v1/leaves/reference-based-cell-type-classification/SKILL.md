---
name: reference-based-cell-type-classification
description: Use when a query scRNA-seq expression matrix (e.g. Drop-seq Campbell
  dataset) must be assigned cell-type labels using an annotated reference dataset
  (e.g. SMART-seq Tasic mouse visual cortex cells), or when benchmarking RBC tools'
  runtime/accuracy.
when_to_use_negative:
- The query dataset contains cell types not covered by the reference and no atlas
  is available — plain RBC methods (and even scMAGIC's accuracy) are stressed in this
  situation; consider atlas-based scMAGIC_atlas instead
- The goal is de novo clustering without any annotated reference or atlas matrix —
  RBC is inapplicable by definition
- Only bulk reference data of unverified quality is available — scMAGIC expects well-annotated
  bulk or single-cell RNA-seq references
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMAGIC
  role: 'Primary RBC tool: two-round classification by expression profile correlation
    plus marker gene verification'
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: SingleR
  role: Competing statistical-metric-based RBC method used in the benchmark
  repo: https://github.com/dviraran/SingleR
- name: scmap
  role: Competing statistical-metric-based RBC method used in the benchmark
  repo: http://bioconductor.org/packages/scmap
- name: SciBet
  role: Competing statistical-metric-based RBC method; also scMAGIC's default HVGene
    method (SciBet_R fallback on Windows)
  repo: https://github.com/PaulingLiu/scibet
- name: CHETAH
  role: Competing tree-based RBC method used in the benchmark
  repo: https://github.com/jdekanter/CHETAH
- name: scClassify
  role: Competing tree-based RBC method used in the benchmark
- name: scPred
  role: Competing machine-learning-based RBC method used in the benchmark
  repo: https://github.com/powellgenomicslab/scPred
- name: SingleCellNet
  role: Competing machine-learning-based RBC method used in the benchmark
  repo: https://github.com/CahanLab/singleCellNet
- name: scID
  role: Competing machine-learning-based RBC method used in the benchmark
  repo: https://github.com/BatadaLab/scID
- name: Seurat
  role: Preprocessing (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, RunUMAP)
    and Seurat-object compatible input/output via scMAGIC_Seurat
- name: ggplot2
  role: Visualization of annotation results (DimPlot UMAPs, runtime scaling curves)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:214293eb79b9b477ca10c82b371bdbf6223c9774f2d895c9ebc5244750cbbeb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - CHETAH
  - SciBet
  - Seurat
  - SingleCellNet
  - SingleR
  - ggplot2
  - scClassify
  - scID
  - scMAGIC
  - scPred
  - scmap
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 21
---

# reference-based-cell-type-classification

## Summary

Annotate query single-cell RNA-seq cells by reference-based classification (RBC): assign each query cell a cell type by comparing its expression profile against a well-annotated reference scRNA-seq matrix, as implemented by scMAGIC and benchmarked against 13 competing RBC methods across 86 tests.

## When to use

Use when a query scRNA-seq expression matrix (e.g. Drop-seq Campbell dataset) must be assigned cell-type labels using an annotated reference dataset (e.g. SMART-seq Tasic mouse visual cortex cells), or when benchmarking RBC tools' runtime/accuracy.

## When NOT to use

- The query dataset contains cell types not covered by the reference and no atlas is available — plain RBC methods (and even scMAGIC's accuracy) are stressed in this situation; consider atlas-based scMAGIC_atlas instead
- The goal is de novo clustering without any annotated reference or atlas matrix — RBC is inapplicable by definition
- Only bulk reference data of unverified quality is available — scMAGIC expects well-annotated bulk or single-cell RNA-seq references

## Inputs

- Query scRNA-seq expression matrix (exp_sc_mat) and optional true labels, loaded from .Rdata (e.g. Campbell.Rdata, Campbell_2k.Rdata, MouseNeocortex.Rdata)
- Reference expression matrix with cell-type labels (ref.mtx, ref.labels from list.Ref$mat_exp and list.Ref$label, e.g. Tasic.Rdata)
- Atlas expression matrix for marker gene identification (MCA_ref, atlas = 'MCA')
- Seurat object as alternative input format

## Outputs

- Per-cell predicted cell-type labels (output.scMAGIC$scMAGIC.tag / pred.scMAGIC)
- Confusion table of true vs. predicted labels: table(label_sc, pred.scMAGIC)
- UMAP plots colored by original and predicted labels from a Seurat object
- Runtime measurements per method and query-cell subset size for benchmark comparisons

## How to apply

Load the reference (.Rdata with mat_exp and labels, e.g. Tasic.Rdata, 1727 SMART-seq mouse primary visual cortex cells) and the query expression matrix (exp_sc_mat, e.g. Campbell.Rdata Drop-seq hypothalamic arcuate-median eminence cells). Run scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 10), which first identifies the most likely reference cell type per query cell by expression profile correlation, verifies identity using reference cell-type-specific marker genes (supplemented by the atlas expression matrix, e.g. MCA), then conducts a second-round RBC using confidently validated query cells as a new reference to eliminate reference-query batch effects. For Windows, set method_HVGene = 'SciBet_R'. Retrieve labels via output.scMAGIC$scMAGIC.tag and validate against original labels with table(label_sc, pred.scMAGIC) and UMAP plots. When no reference is available, use scMAGIC_atlas(exp_sc_mat, MCA_ref, atlas = 'MCA', type_ref = 'sum-counts', use_RUVseq = F, min_cell = 5). Seurat object input/output is supported via scMAGIC_Seurat().

## Related tools

- **scMAGIC** (Primary RBC tool: two-round classification by expression profile correlation plus marker gene verification) — https://github.com/TianLab-Bioinfo/scMAGIC
- **SingleR** (Competing statistical-metric-based RBC method used in the benchmark) — https://github.com/dviraran/SingleR
- **scmap** (Competing statistical-metric-based RBC method used in the benchmark) — http://bioconductor.org/packages/scmap
- **SciBet** (Competing statistical-metric-based RBC method; also scMAGIC's default HVGene method (SciBet_R fallback on Windows)) — https://github.com/PaulingLiu/scibet
- **CHETAH** (Competing tree-based RBC method used in the benchmark) — https://github.com/jdekanter/CHETAH
- **scClassify** (Competing tree-based RBC method used in the benchmark)
- **scPred** (Competing machine-learning-based RBC method used in the benchmark) — https://github.com/powellgenomicslab/scPred
- **SingleCellNet** (Competing machine-learning-based RBC method used in the benchmark) — https://github.com/CahanLab/singleCellNet
- **scID** (Competing machine-learning-based RBC method used in the benchmark) — https://github.com/BatadaLab/scID
- **Seurat** (Preprocessing (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, RunUMAP) and Seurat-object compatible input/output via scMAGIC_Seurat)
- **ggplot2** (Visualization of annotation results (DimPlot UMAPs, runtime scaling curves))

## Examples

```
output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 10); pred.scMAGIC <- output.scMAGIC$scMAGIC.tag
```

## Evaluation signals

- Confusion matrix table(label_sc, pred.scMAGIC) should show agreement with original labels (e.g. Campbell et al. annotations) for most cells
- UMAP plots colored by original.label vs pred.tag should show spatial concordance of predicted clusters with true labels
- Runtime benchmark: scMAGIC should classify 20,000 query cells within ~6 min on the DGH reference, with little runtime increase above 5,000 query cells (Supplementary Figure S25A speed-up behavior)
- Seurat-object path: scMAGIC_Seurat(seurat.query, seurat.ref) should return the query object with prediction tags added to meta.data
- Python/numpy availability check must pass before running: print(py_module_available('numpy')) should return True

## Limitations

- RBC methods that directly compare query against reference expression profiles (statistical-, tree-, machine-learning-based) are prone to batch effects between reference and query datasets
- All RBC methods struggle when query cell types are not completely covered by the reference dataset (e.g. mouse duodenum dataset)
- scMCA and scHCL, the tools provided by MCA and HCL atlases, do not work well on the four query datasets tested
- Windows users must set method_HVGene = 'SciBet_R' and pre-import numpy via reticulate before running scMAGIC
- No versioned code release or data deposition (e.g. Zenodo DOI) is stated for scMAGIC beyond the GitHub repository link

## Evidence

- [methods] Main idea: correlation then marker verification: "first identify the most likely cell type in the reference for a single cell based on expression profile correlation"
- [methods] Second-round RBC rationale: "conducts a second-round of RBC by using target single cells with verified cell identity as the new reference"
- [methods] Concrete invocation: "output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)"
- [results] Runtime scaling finding: [evidence span withheld — anchor status: not_found]
- [readme] Atlas-based mode: "output.scMAGIC <- scMAGIC_atlas(exp_sc_mat, MCA_ref, atlas = 'MCA',
                                type_ref = 'sum-counts', use_RUVseq = F,
                                min_cell = 5, num_threads"
- [intro] Batch effect limitation of direct-comparison methods: "The first three categories of methods directly compare the query expression profiles against the reference expression profiles and are therefore prone to be affected by the batch effects between the"
