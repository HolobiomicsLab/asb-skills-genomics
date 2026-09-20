---
name: reference-based-classification-benchmarking
description: 'Use when a user must choose or validate a reference-based single-cell
  annotation method: paired reference/query scRNA-seq datasets exist and comparative
  accuracy (accuracy, balanced accuracy, cluster accuracy, median rank) must be measured
  across benchmark tests.'
when_to_use_negative:
- When no annotated reference dataset exists for the query tissue/species — use the
  reference-free scMAGIC_atlas mode instead, which classifies against an atlas (MCA)
  expression matrix
- When the goal is de novo clustering of an unlabeled dataset rather than transferring
  labels from an annotated reference
- When the analysis targets tools that integrate reference and query into a shared
  embedding space by design (e.g. Seurat v4 integration, scSemiCluster) rather than
  direct expression-profile comparison
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scMAGIC
  role: Primary method under evaluation; two-round reference-based classification
    with marker-gene identification
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: SingleR
  role: Statistical-metric-based competitor run at optimal parameter settings
  repo: https://github.com/dviraran/SingleR
- name: SciBet
  role: Statistical-metric-based competitor; also provides scMAGIC's method_HVGene='SciBet'
    highly variable gene selection
  repo: https://github.com/PaulingLiu/scibet
- name: scmap
  role: Statistical-metric-based competitor
- name: CHETAH
  role: Tree-based competitor run at optimal parameter settings
  repo: https://github.com/jdekanter/CHETAH
- name: scClassify
  role: Tree-based competitor
- name: scPred
  role: Machine-learning-based competitor
- name: SingleCellNet
  role: Machine-learning-based competitor
- name: scID
  role: Machine-learning-based competitor
- name: CaSTLe
  role: Machine-learning-based competitor
- name: SVMrejection
  role: Machine-learning-based competitor
- name: Seurat
  role: Preprocessing (NormalizeData, FindVariableFeatures, ScaleData) to harmonize
    reference-query pairs; also competitor (Seurat v4, median rank 4)
- name: scMAGIC_scripts
  role: Source of benchmark .Rdata datasets (Tasic.Rdata, Campbell_2k.Rdata) and reproduction
    scripts
  repo: https://github.com/Drizzle-Zhang/scMAGIC_scripts
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:665440e7aa9ea15ed35cb176fc7fc04709b59dc05e91f587740343bcaff2588f
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 21
---

# reference-based-classification-benchmarking

## Summary

Benchmark scMAGIC and 13 competing reference-based classification (RBC) methods for single-cell annotation across 86 reference-query scRNA-seq tests spanning three situations (covered types, uncovered types, batch effects/cross-species), comparing accuracy, balanced accuracy, and median accuracy ranks.

## When to use

Use when a user must choose or validate a reference-based single-cell annotation method: paired reference/query scRNA-seq datasets exist and comparative accuracy (accuracy, balanced accuracy, cluster accuracy, median rank) must be measured across benchmark tests.

## When NOT to use

- When no annotated reference dataset exists for the query tissue/species — use the reference-free scMAGIC_atlas mode instead, which classifies against an atlas (MCA) expression matrix
- When the goal is de novo clustering of an unlabeled dataset rather than transferring labels from an annotated reference
- When the analysis targets tools that integrate reference and query into a shared embedding space by design (e.g. Seurat v4 integration, scSemiCluster) rather than direct expression-profile comparison

## Inputs

- Paired reference-query scRNA-seq expression matrices and cell-type labels (.Rdata via readRDS, e.g. Tasic.Rdata, Campbell_2k.Rdata) for 86 benchmark tests covering three RBC situations
- Atlas expression matrices MCA_ref / HCL for atlas-informed marker gene identification
- Installable R implementations of the 13 competing RBC methods with optimal parameter settings

## Outputs

- Per-test, per-method classification accuracy, balanced accuracy, and cluster accuracy tables
- Median accuracy ranks per method across all 86 benchmark tests (reproducing Figures 2-5 rankings)
- Predicted cell-type label vectors (output.scMAGIC$scMAGIC.tag) and cross-tabulations table(label_sc, pred.scMAGIC)

## How to apply

Download paired reference-query scRNA-seq benchmark datasets (e.g. Tasic SMART-seq 1727 mouse primary visual cortex cells as reference, Campbell Drop-seq hypothalamic arcuate-median eminence cells as query) from public accessions or the Drizzle-Zhang/scMAGIC_scripts repository, and preprocess with Seurat (NormalizeData, FindVariableFeatures, ScaleData) so all methods receive comparable input matrices and labels. Run scMAGIC via scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas='MCA' or 'HCL', num_threads=n), extracting predictions from output.scMAGIC$scMAGIC.tag; run the 13 competitors at their optimal parameter settings, grouped as statistical-metric-based (SingleR, scmap, SciBet), tree-based (CHETAH, scClassify), and machine-learning-based (scPred, SingleCellNet, scID, CaSTLe, SVMrejection), installing SciBet from PaulingLiu/scibet and COSG from genecell/COSGR. Cross-tabulate predicted versus true labels with table(label_sc, pred.scMAGIC) and compute accuracy, balanced accuracy, and cluster accuracy per method per test. Aggregate across the 86 tests per situation into median accuracy ranks; success is scMAGIC attaining median rank 1 versus rank 4 (Seurat v4) and rank 5 (scSemiCluster) for the next best methods.

## Related tools

- **scMAGIC** (Primary method under evaluation; two-round reference-based classification with marker-gene identification) — https://github.com/TianLab-Bioinfo/scMAGIC
- **SingleR** (Statistical-metric-based competitor run at optimal parameter settings) — https://github.com/dviraran/SingleR
- **SciBet** (Statistical-metric-based competitor; also provides scMAGIC's method_HVGene='SciBet' highly variable gene selection) — https://github.com/PaulingLiu/scibet
- **scmap** (Statistical-metric-based competitor)
- **CHETAH** (Tree-based competitor run at optimal parameter settings) — https://github.com/jdekanter/CHETAH
- **scClassify** (Tree-based competitor)
- **scPred** (Machine-learning-based competitor)
- **SingleCellNet** (Machine-learning-based competitor)
- **scID** (Machine-learning-based competitor)
- **CaSTLe** (Machine-learning-based competitor)
- **SVMrejection** (Machine-learning-based competitor)
- **Seurat** (Preprocessing (NormalizeData, FindVariableFeatures, ScaleData) to harmonize reference-query pairs; also competitor (Seurat v4, median rank 4))
- **scMAGIC_scripts** (Source of benchmark .Rdata datasets (Tasic.Rdata, Campbell_2k.Rdata) and reproduction scripts) — https://github.com/Drizzle-Zhang/scMAGIC_scripts

## Examples

```
output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4); pred.scMAGIC <- output.scMAGIC$scMAGIC.tag; table(label_sc, pred.scMAGIC)
```

## Evaluation signals

- Median accuracy rank of scMAGIC across all 86 benchmark tests equals 1, with Seurat v4 at rank 4 and scSemiCluster at rank 5 (Figure 5A)
- On batch-effect tests (Tasic SMART-seq reference vs Campbell Drop-seq query; cross-species pairs), two-round scMAGIC accuracies of 0.98 and 0.85 are reproduced, versus 0.33 and 0.34 for single_round mode
- Predicted labels are cross-tabulated against true labels via table(label_sc, pred.scMAGIC) and per-test accuracy is computed as the fraction of correctly assigned cells
- scMAGICno-Atlas matches full scMAGIC accuracy when query types are fully covered by the reference, but drops evidently on uncovered-cell-type benchmarks (Figure 6C)
- All methods receive identically preprocessed Seurat input matrices, so accuracy differences reflect method rather than preprocessing variance

## Limitations

- Dataset accession codes and detailed dataset information are deferred to Materials and Methods and not provided inline
- No version numbers are stated for several tools (Seurat, R, sciBet, scSemiCluster, ggplot2, devtools) and no archived code release or Zenodo DOI exists for scMAGIC beyond the GitHub link
- scMAGIC's advantage is scenario-dependent: it is most pronounced when query cell types are not completely covered by the reference or when significant batch effects exist; its second-round RBC is critical only under such conditions
- On Windows, method_HVGene must be set to 'SciBet_R' and the reticulate/numpy environment must be preconfigured

## Evidence

- [abstract] scMAGIC significantly outperforms 13 competing RBC methods with optimal parameter settings across 86 benchmark tests: "scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests"
- [methods] Two-round vs single-round ablation under batch effects: [evidence span withheld — anchor status: not_found]
- [intro] Second-round RBC rationale: "scMAGIC introduces a second-round RBC in which those query cells whose cell identifies are confidently validated in the first-round RBC are used as the new reference to again classify query single"
- [readme] Core scMAGIC invocation from README tutorial: "output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)"
- [intro] Competitor grouping in article intro: "statistical-metric-based methods [e.g. SingleR (9), scmap (10) and SciBet (11)]"
- [methods] Installation via devtools from methods: "devtools::install_github("Drizzle-Zhang/scMAGIC")"
