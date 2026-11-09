---
name: multi-method-rank-aggregation
description: Use when comparing a cell-type annotation method against statistical-metric-based
  (SingleR, scmap, SciBet), tree-based (CHETAH, scClassify) and machine-learning-based
  (scPred, SingleCellNet, scID, CaSTLe, SVMrejection) competitors on many paired reference-query
  tests.
when_to_use_negative:
- You only need a single annotation of one query dataset, not a systematic multi-method
  benchmark across many reference-query pairs
- The query dataset has no paired, well-annotated reference and no atlas (MCA/HCL)
  is applicable, so RBC methods cannot be compared fairly
- Competing methods lack configurable 'optimal parameter settings' or cannot accept
  Seurat-preprocessed expression matrices, making a fair rank comparison infeasible
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMAGIC
  role: Target classification method being benchmarked; run via scMAGIC(exp_sc_mat,
    ref.mtx, ref.labels, atlas='MCA'|'HCL')
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: SingleR
  role: Competing statistical-metric-based RBC method run at optimal parameters
  repo: dviraran/SingleR
- name: scmap
  role: Competing statistical-metric-based RBC method run at optimal parameters
  repo: hemberg-lab/scmap-shiny
- name: SciBet
  role: Competing statistical-metric-based RBC method; installed from GitHub, also
    used as scMAGIC's HVGene selector (SciBet_R on Windows)
  repo: https://github.com/PaulingLiu/scibet
- name: CHETAH
  role: Competing tree-based RBC method run at optimal parameters
  repo: jdekanter/CHETAH
- name: scClassify
  role: Competing tree-based RBC method run at optimal parameters
- name: scPred
  role: Competing machine-learning-based RBC method run at optimal parameters
  repo: powellgenomicslab/scPred
- name: SingleCellNet
  role: Competing machine-learning-based RBC method run at optimal parameters
  repo: CahanLab/singleCellNet
- name: scID
  role: Competing machine-learning-based RBC method run at optimal parameters
  repo: BatadaLab/scID
- name: CaSTLe
  role: Competing machine-learning-based RBC method run at optimal parameters
  repo: yuvallb/CaSTLe
- name: SVMrejection
  role: Competing machine-learning-based RBC method run at optimal parameters
- name: Seurat
  role: Preprocessing (NormalizeData, FindVariableFeatures, ScaleData) for comparable
    inputs; Seurat v4 is also a competitor and second-ranked method
- name: scMAGIC_scripts
  role: Source of benchmark data files (Tasic.Rdata, Campbell_2k.Rdata) used in the
    comparison
  repo: https://github.com/Drizzle-Zhang/scMAGIC_scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:9df754fd0652e1927f1cec862e52b75aef144fcbd8c1203ba7963914adcd9483
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
  - scMAGIC_scripts
  - scPred
  - scmap
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6956521739130435
n_claims: 23
---

# multi-method-rank-aggregation

## Summary

Benchmark one target method (scMAGIC) against 13 competing reference-based classification (RBC) methods across 86 paired reference-query scRNA-seq tests spanning three RBC situations, then aggregate per-test accuracy, balanced accuracy and cluster accuracy into median ranks per method.

## When to use

Use when comparing a cell-type annotation method against statistical-metric-based (SingleR, scmap, SciBet), tree-based (CHETAH, scClassify) and machine-learning-based (scPred, SingleCellNet, scID, CaSTLe, SVMrejection) competitors on many paired reference-query tests.

## When NOT to use

- You only need a single annotation of one query dataset, not a systematic multi-method benchmark across many reference-query pairs
- The query dataset has no paired, well-annotated reference and no atlas (MCA/HCL) is applicable, so RBC methods cannot be compared fairly
- Competing methods lack configurable 'optimal parameter settings' or cannot accept Seurat-preprocessed expression matrices, making a fair rank comparison infeasible

## Inputs

- 86 paired reference-query scRNA-seq benchmark datasets (gene expression matrices plus cell-type labels) covering three RBC situations
- Reference expression matrix and reference cell-type labels (e.g. Tasic.Rdata SMART-seq mouse primary visual cortex, 1727 cells)
- Query expression matrix and true query labels (e.g. Campbell_2k.Rdata Drop-seq mouse hypothalamic arcuate-median eminence cells)
- Atlas reference matrices for reference-free situations (MCA_ref, HCL)

## Outputs

- Per-method per-benchmark-test accuracy, balanced accuracy and cluster accuracy
- Median accuracy ranks per method aggregated across the 86 tests per RBC situation
- Comparative ranking figure reproducing Figure 5A (scMAGIC median rank 1, Seurat v4 rank 4, scSemiCluster rank 5)

## How to apply

Download the 86 paired reference-query scRNA-seq benchmark datasets covering the three RBC situations (covered cell types, uncovered cell types, cross-species/batch effects) from their public accessions. Preprocess each reference-query pair with Seurat (NormalizeData, FindVariableFeatures, ScaleData) so all methods receive comparable expression matrices and labels. Run scMAGIC via scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas='MCA' or 'HCL', num_threads=n) and run the 13 competitors at their optimal parameter settings, installing SciBet from PaulingLiu/scibet and COSG from genecell/COSGR. Compare predicted labels against true query labels, computing accuracy, balanced accuracy and cluster accuracy per method per test. Aggregate ranks across tests within each situation (e.g. median accuracy rank) to reproduce the comparative rankings of Figures 2-5; correct application yields scMAGIC at median rank 1, Seurat v4 at rank 4 and scSemiCluster at rank 5.

## Related tools

- **scMAGIC** (Target classification method being benchmarked; run via scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas='MCA'|'HCL')) — https://github.com/TianLab-Bioinfo/scMAGIC
- **SingleR** (Competing statistical-metric-based RBC method run at optimal parameters) — dviraran/SingleR
- **scmap** (Competing statistical-metric-based RBC method run at optimal parameters) — hemberg-lab/scmap-shiny
- **SciBet** (Competing statistical-metric-based RBC method; installed from GitHub, also used as scMAGIC's HVGene selector (SciBet_R on Windows)) — https://github.com/PaulingLiu/scibet
- **CHETAH** (Competing tree-based RBC method run at optimal parameters) — jdekanter/CHETAH
- **scClassify** (Competing tree-based RBC method run at optimal parameters)
- **scPred** (Competing machine-learning-based RBC method run at optimal parameters) — powellgenomicslab/scPred
- **SingleCellNet** (Competing machine-learning-based RBC method run at optimal parameters) — CahanLab/singleCellNet
- **scID** (Competing machine-learning-based RBC method run at optimal parameters) — BatadaLab/scID
- **CaSTLe** (Competing machine-learning-based RBC method run at optimal parameters) — yuvallb/CaSTLe
- **SVMrejection** (Competing machine-learning-based RBC method run at optimal parameters)
- **Seurat** (Preprocessing (NormalizeData, FindVariableFeatures, ScaleData) for comparable inputs; Seurat v4 is also a competitor and second-ranked method)
- **scMAGIC_scripts** (Source of benchmark data files (Tasic.Rdata, Campbell_2k.Rdata) used in the comparison) — https://github.com/Drizzle-Zhang/scMAGIC_scripts

## Examples

```
output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4); pred.scMAGIC <- output.scMAGIC$scMAGIC.tag
```

## Evaluation signals

- The reproduced median accuracy rank of scMAGIC across all 86 benchmark tests equals 1, matching Figure 5A
- Second-ranked method is Seurat v4 (median rank 4) and third is scSemiCluster (median rank 5)
- Every method receives the same Seurat-preprocessed input matrix per benchmark test (identical reference/query matrices and labels)
- Predicted labels are compared against true query labels with three metrics (accuracy, balanced accuracy, cluster accuracy) computed for all 14 methods on all 86 tests
- Rankings per RBC situation are consistent with scMAGIC outperforming all 13 competitors in each of the three situations

## Limitations

- Dataset accession codes and publication citations for all benchmark datasets are deferred to Materials and Methods and not provided in the section text
- No version numbers are stated for tools such as Seurat, R, sciBet or scSemiCluster, complicating exact reproduction
- Competing methods must be run at their own optimal parameter settings; results depend on those settings being correctly determined
- SciBet's R implementation (SciBet_R / scibetR) must be used on Windows, which may alter runtime characteristics rather than accuracy

## Evidence

- [abstract] Top overall performance claim: "scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests"
- [intro] Three-situation benchmark design: "We have designed three situations that users may encounter when conducting RBC and have prepared a total of 86 benchmark tests using real datasets"
- [methods] scMAGIC invocation used per benchmark test: "output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)"
- [methods] Competitor installation (SciBet): "devtools::install_github("PaulingLiu/scibet")"
- [results] Median-rank aggregation result: [evidence span withheld — anchor status: not_found]
- [readme] Competing methods run via scMAGIC_scripts: [evidence span withheld — anchor status: external_doc]
