---
name: batch-effect-robustness-evaluation
description: Use when evaluating whether a reference-based cell-type classifier (or
  comparing several classifiers) retains accuracy when reference and query scRNA-seq
  datasets differ by sequencing protocol, species, or batch, or when query cell types
  are incompletely covered by the reference.
when_to_use_negative:
- When no labeled reference or atlas dataset exists and the goal is de novo clustering
  rather than evaluating reference-based classification robustness
- When the analysis goal is batch-effect correction or data integration itself (e.g.
  Seurat integration) rather than measuring how batch effects degrade classification
  accuracy
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMAGIC
  role: Method under evaluation; two-round RBC (correlation-based candidate identification,
    marker-gene validation, reclassification using validated query cells as new reference)
    that is tested for batch-effect robustness
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: Seurat
  role: Preprocessing of reference-query pairs (NormalizeData, FindVariableFeatures,
    ScaleData) so all methods receive comparable input; also a competing integration-based
    method (Seurat v4)
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: SingleR
  role: Competing statistical-metric-based RBC method run at optimal parameters
  repo: https://github.com/dviraran/SingleR
- name: scmap
  role: Competing statistical-metric-based RBC method run at optimal parameters
  repo: https://github.com/hemberg-lab/scmap-shiny
- name: SciBet
  role: Competing statistical-metric-based RBC method installed from GitHub; also
    used inside scMAGIC for high-variable-gene selection (method_HVGene)
  repo: https://github.com/PaulingLiu/scibet
- name: CHETAH
  role: Competing tree-based RBC method run at optimal parameters
  repo: https://github.com/jdekanter/CHETAH
- name: scClassify
  role: Competing tree-based RBC method run at optimal parameters
- name: scPred
  role: Competing machine-learning-based RBC method run at optimal parameters
- name: SingleCellNet
  role: Competing machine-learning-based RBC method run at optimal parameters
- name: scID
  role: Competing machine-learning-based RBC method run at optimal parameters
- name: CaSTLe
  role: Competing machine-learning-based (transfer-learning) RBC method run at optimal
    parameters
- name: SVMrejection
  role: Competing machine-learning-based RBC method run at optimal parameters
- name: COSG
  role: Marker-gene method installed from genecell/COSGR as a scMAGIC dependency
  repo: https://github.com/genecell/COSGR
- name: scSemiCluster
  role: Semi-supervised-learning-based competitor that integrates reference and query
    into the same space; ranked third by median accuracy rank
  repo: https://github.com/xuebaliang/scSemiCluster
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:8bea1eb1ddda754f0f25e168c6eeeea6520434fffb4d042b60c9b00c7daa8063
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
  - scSemiCluster
  - scmap
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 21
---

# batch-effect-robustness-evaluation

## Summary

Benchmark protocol for assessing how robustly reference-based cell classification methods withstand batch effects between reference and query scRNA-seq data, by running scMAGIC and 13 competing methods at their optimal parameter settings across 86 paired reference-query benchmark tests spanning three real-data situations.

## When to use

Use when evaluating whether a reference-based cell-type classifier (or comparing several classifiers) retains accuracy when reference and query scRNA-seq datasets differ by sequencing protocol, species, or batch, or when query cell types are incompletely covered by the reference.

## When NOT to use

- When no labeled reference or atlas dataset exists and the goal is de novo clustering rather than evaluating reference-based classification robustness
- When the analysis goal is batch-effect correction or data integration itself (e.g. Seurat integration) rather than measuring how batch effects degrade classification accuracy

## Inputs

- Paired reference-query scRNA-seq benchmark datasets (86 tests across three RBC situations, including cross-species pairs) with cell-type labels
- Reference expression matrix and cell-type labels (e.g. Tasic.Rdata SMART-seq mouse visual cortex, MCA or HCL atlas)
- Query expression matrix with true labels (e.g. Campbell Drop-seq hypothalamic arcuate-median eminence cells, MouseNeocortex.Rdata)

## Outputs

- Per-method accuracy, balanced accuracy, and cluster accuracy for each of the 86 benchmark tests
- Median accuracy ranks per method aggregated across tests (expected: scMAGIC rank 1, Seurat v4 rank 4, scSemiCluster rank 5, cf. Figure 5A)
- Comparative rankings across the three RBC situations (Figures 2-5)

## How to apply

Assemble paired reference-query scRNA-seq benchmark datasets covering the three RBC situations (covered types, uncovered types, and significant batch effects, plus cross-species pairs), obtained from their public accessions. Preprocess each pair with Seurat (NormalizeData, FindVariableFeatures, ScaleData) so all methods receive comparable input matrices and labels. Run scMAGIC via scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas='MCA' or 'HCL', num_threads=n) and the 13 competitors — statistical-metric-based (SingleR, scmap, SciBet), tree-based (CHETAH, scClassify), and machine-learning-based (scPred, SingleCellNet, scID, CaSTLe, SVMrejection) — at their optimal parameter settings, installing SciBet and COSG from GitHub (PaulingLiu/scibet, genecell/COSGR). Compare predicted labels against true query labels and compute accuracy, balanced accuracy and cluster accuracy for every method on every benchmark test. Aggregate per-situation results into median ranks across the 86 tests to reproduce the comparative rankings of Figures 2-5.

## Related tools

- **scMAGIC** (Method under evaluation; two-round RBC (correlation-based candidate identification, marker-gene validation, reclassification using validated query cells as new reference) that is tested for batch-effect robustness) — https://github.com/TianLab-Bioinfo/scMAGIC
- **Seurat** (Preprocessing of reference-query pairs (NormalizeData, FindVariableFeatures, ScaleData) so all methods receive comparable input; also a competing integration-based method (Seurat v4)) — https://github.com/TianLab-Bioinfo/scMAGIC
- **SingleR** (Competing statistical-metric-based RBC method run at optimal parameters) — https://github.com/dviraran/SingleR
- **scmap** (Competing statistical-metric-based RBC method run at optimal parameters) — https://github.com/hemberg-lab/scmap-shiny
- **SciBet** (Competing statistical-metric-based RBC method installed from GitHub; also used inside scMAGIC for high-variable-gene selection (method_HVGene)) — https://github.com/PaulingLiu/scibet
- **CHETAH** (Competing tree-based RBC method run at optimal parameters) — https://github.com/jdekanter/CHETAH
- **scClassify** (Competing tree-based RBC method run at optimal parameters)
- **scPred** (Competing machine-learning-based RBC method run at optimal parameters)
- **SingleCellNet** (Competing machine-learning-based RBC method run at optimal parameters)
- **scID** (Competing machine-learning-based RBC method run at optimal parameters)
- **CaSTLe** (Competing machine-learning-based (transfer-learning) RBC method run at optimal parameters)
- **SVMrejection** (Competing machine-learning-based RBC method run at optimal parameters)
- **COSG** (Marker-gene method installed from genecell/COSGR as a scMAGIC dependency) — https://github.com/genecell/COSGR
- **scSemiCluster** (Semi-supervised-learning-based competitor that integrates reference and query into the same space; ranked third by median accuracy rank) — https://github.com/xuebaliang/scSemiCluster

## Examples

```
library(scMAGIC); list.Ref <- readRDS('Tasic.Rdata'); list.demo <- readRDS('Campbell_2k.Rdata'); output.scMAGIC <- scMAGIC(list.demo$mat_exp, list.Ref$mat_exp, list.Ref$label[,1], atlas='MCA', num_threads=4)
```

## Evaluation signals

- Across all 86 benchmark tests, scMAGIC should attain a median accuracy rank of 1, with Seurat v4 at median rank 4 and scSemiCluster at median rank 5 (Figure 5A)
- Rankings must be reproduced consistently within each of the three RBC situations, with scMAGIC especially ahead in the uncovered-cell-type and significant-batch-effect situations
- Accuracy, balanced accuracy, and cluster accuracy must be computed for all 14 methods on every benchmark test; missing-method tests invalidate the comparison
- All methods must receive identically preprocessed matrices (Seurat NormalizeData/FindVariableFeatures/ScaleData) so differences reflect the classifiers, not preprocessing
- Sanity-check the scMAGIC run itself: classification results should reproduce table(label_sc, pred.scMAGIC) concordance on the Tasic/Campbell tutorial pair

## Limitations

- Batch effects between reference and query data strongly degrade statistical-, tree- and machine-learning-based RBC methods that directly compare expression profiles; results are only valid when this is accounted for
- All RBC methods struggle when query cell types are not covered by the reference; accuracy metrics in the uncovered-type situation must be interpreted with this in mind
- Dataset accession codes, tool version numbers (Seurat, R, sciBet, scSemiCluster, ggplot2, devtools), and an archived scMAGIC code release are not provided in the cited sections; exact reproduction requires Materials and Methods detail
- Some competitors (sciBet, scSemiCluster) wrongly assign all unclassifiable query cells rather than abstaining, biasing accuracy in uncovered-type tests

## Evidence

- [abstract] Top performance claim: "scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests"
- [abstract] Batch-effect robustness scope: "especially when the cell types in the query dataset are not completely covered by the reference dataset and when there exist significant batch effects"
- [intro] Benchmark design: "We have designed three situations that users may encounter when conducting RBC and have prepared a total of 86 benchmark tests using real datasets for these situations."
- [methods] Second-round RBC rationale: "conducts a second-round of RBC by using target single cells with verified cell identity as the new reference"
- [methods] Installation of competitors: "devtools::install_github("PaulingLiu/scibet")"
- [readme] scMAGIC call signature: "output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)"
- [discussion] Batch-effect mechanism: "replacing the original reference expression matrix with a local reference expression matrix constructed from query cells with verified cell labels and redoing the classification"
