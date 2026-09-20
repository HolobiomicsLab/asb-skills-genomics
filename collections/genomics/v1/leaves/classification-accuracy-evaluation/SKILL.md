---
name: classification-accuracy-evaluation
description: Use when a scRNA-seq cell-type classifier has been run on query datasets
  with known true labels and you must compute per-test annotation accuracy, aggregate
  it across benchmark tests, and compare methods or ablations (e.g., full scMAGIC
  vs scMAGICno-Atlas).
when_to_use_negative:
- No reference dataset is available and the goal is reference-free annotation — in
  that case use scMAGIC_atlas with an atlas matrix (e.g., MCA_ref) rather than evaluating
  reference-based classification accuracy
- Ground-truth query labels are unavailable, so correctness of predictions cannot
  be tabulated against true labels
- The question is marker-gene discovery or clustering quality rather than assigning
  and scoring discrete cell-type labels
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMAGIC
  role: Primary RBC classifier whose scMAGIC.tag predictions are scored against true
    query labels; also benchmarked against 13 competing methods
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: Seurat
  role: Preprocessing (NormalizeData, FindVariableFeatures, ScaleData) so all methods
    receive comparable reference-query input matrices; DimPlot visualization of true
    vs predicted labels
- name: SingleR
  role: Competing statistical-metric-based RBC method scored alongside scMAGIC in
    the benchmark comparison
  repo: https://github.com/dviraran/SingleR
- name: SciBet
  role: Competing statistical-metric-based RBC method scored in the benchmark comparison;
    also used by scMAGIC for HVGene selection (method_HVGene)
  repo: https://github.com/PaulingLiu/scibet
- name: scmap
  role: Competing statistical-metric-based RBC method included in the 86-test benchmark
    ranking
  repo: https://github.com/hemberg-lab/scmap-shiny
- name: CHETAH
  role: Competing tree-based RBC method included in the benchmark comparison
  repo: https://github.com/jdekanter/CHETAH
- name: scClassify
  role: Competing tree-based RBC method included in the benchmark comparison
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:bbaddbec4cbfe0610aacafdea69660163da657d52c6ef58a89f3a2227d3e0fa1
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
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 21
---

# classification-accuracy-evaluation

## Summary

Quantifies single-cell annotation quality by tabulating predicted versus true cell-type labels and computing accuracy (fraction of correctly assigned cells) per benchmark test, enabling ranked comparison of reference-based classification (RBC) methods such as scMAGIC against 13 competing methods across 86 benchmark tests.

## When to use

Use when a scRNA-seq cell-type classifier has been run on query datasets with known true labels and you must compute per-test annotation accuracy, aggregate it across benchmark tests, and compare methods or ablations (e.g., full scMAGIC vs scMAGICno-Atlas).

## When NOT to use

- No reference dataset is available and the goal is reference-free annotation — in that case use scMAGIC_atlas with an atlas matrix (e.g., MCA_ref) rather than evaluating reference-based classification accuracy
- Ground-truth query labels are unavailable, so correctness of predictions cannot be tabulated against true labels
- The question is marker-gene discovery or clustering quality rather than assigning and scoring discrete cell-type labels

## Inputs

- Paired reference and query scRNA-seq datasets stored as .Rdata files (list objects with mat_exp and label components), e.g., Tasic.Rdata, Campbell_2k.Rdata, Campbell.Rdata, MouseNeocortex.Rdata
- True query cell-type labels (label_sc) for every query cell
- Predicted cell-type labels from each classifier (e.g., output.scMAGIC$scMAGIC.tag) or ablation setting

## Outputs

- Per-test annotation accuracy values (fraction of correctly assigned cells) for each method or ablation on each benchmark test
- Confusion tabulations of predicted versus true labels via table(label_sc, pred.scMAGIC)
- Aggregated ranking tables (e.g., median accuracy ranks across the 86 benchmark tests) and summary CSVs such as scmagic_noatlas_accuracy.csv

## How to apply

For each reference-query pair (e.g., the Tasic SMART-seq reference annotating the Campbell Drop-seq hypothalamic arcuate-median eminence query, or MCA annotating the MouseNeocortex dataset), load the .Rdata files with readRDS to obtain exp_sc_mat and label_sc, then run the classifier (scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas='MCA' or 'HCL', num_threads=4-10), with the atlas argument omitted for the scMAGICno-Atlas ablation). Extract predictions via output.scMAGIC$scMAGIC.tag, cross-tabulate with table(label_sc, pred.scMAGIC), and compute accuracy as the fraction of cells whose predicted label matches the true label. Preprocess reference and query pairs identically with Seurat (NormalizeData, FindVariableFeatures, ScaleData) so all methods receive comparable input. Compile per-test accuracies into a table (e.g., scmagic_noatlas_accuracy.csv) and aggregate across tests (median ranks) to reproduce comparisons such as scMAGIC's median rank of 1 versus Seurat v4 (rank 4) and scSemiCluster (rank 5) across the 86 benchmark tests in the three RBC situations.

## Related tools

- **scMAGIC** (Primary RBC classifier whose scMAGIC.tag predictions are scored against true query labels; also benchmarked against 13 competing methods) — https://github.com/TianLab-Bioinfo/scMAGIC
- **Seurat** (Preprocessing (NormalizeData, FindVariableFeatures, ScaleData) so all methods receive comparable reference-query input matrices; DimPlot visualization of true vs predicted labels)
- **SingleR** (Competing statistical-metric-based RBC method scored alongside scMAGIC in the benchmark comparison) — https://github.com/dviraran/SingleR
- **SciBet** (Competing statistical-metric-based RBC method scored in the benchmark comparison; also used by scMAGIC for HVGene selection (method_HVGene)) — https://github.com/PaulingLiu/scibet
- **scmap** (Competing statistical-metric-based RBC method included in the 86-test benchmark ranking) — https://github.com/hemberg-lab/scmap-shiny
- **CHETAH** (Competing tree-based RBC method included in the benchmark comparison) — https://github.com/jdekanter/CHETAH
- **scClassify** (Competing tree-based RBC method included in the benchmark comparison)

## Examples

```
output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, num_threads = 4); pred.scMAGIC <- output.scMAGIC$scMAGIC.tag; table(label_sc, pred.scMAGIC)
```

## Evaluation signals

- Accuracy equals the diagonal sum of table(label_sc, pred.scMAGIC) divided by the total number of query cells; recompute the fraction from the confusion table to verify the compiled CSV
- Across the 86 benchmark tests the reproduced median accuracy rank for scMAGIC should be 1, with Seurat v4 at rank 4 and scSemiCluster at rank 5
- On benchmark tests where query cell types are not fully covered by the reference, scMAGICno-Atlas accuracies must be lower than the atlas-assisted scMAGIC accuracies (consistent with Figure 6C)
- Predicted label sets should only contain reference cell-type names; any novel-type queries appearing misassigned confirms the uncovered-cell-type failure mode
- All method comparisons should use identically Seurat-preprocessed input matrices (NormalizeData, FindVariableFeatures, ScaleData) so accuracy differences reflect classifiers, not preprocessing

## Limitations

- All RBC methods struggle when query cell types are not covered by the reference; statistical-, tree- and machine-learning-based methods are also prone to batch effects between reference and query datasets
- Without the atlas expression matrix (scMAGICno-Atlas), accuracy drops evidently on benchmarks whose query includes cell types absent from the reference
- Dataset accession codes and tool version numbers are deferred to Materials and Methods / installation instructions and are not stated in the main comparison section; no archived code release or Zenodo DOI is provided beyond the GitHub repository

## Evidence

- [abstract] scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests: "scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests"
- [intro] Three benchmark situations: "We have designed three situations that users may encounter when conducting RBC and have prepared a total of 86 benchmark tests using real datasets for these situations."
- [readme] Running scMAGIC and extracting predictions: "output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)"
- [readme] Tabulating predictions against true labels: "# classification results
table(label_sc, pred.scMAGIC)"
- [discussion] Batch-effect rationale for two-round RBC: "replacing the original reference expression matrix with a local reference expression matrix constructed from query cells with verified cell labels"
