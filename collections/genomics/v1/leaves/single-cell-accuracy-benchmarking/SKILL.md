---
name: single-cell-accuracy-benchmarking
description: Use when query scRNA-seq datasets have known ground-truth cell-type labels
  and you need to quantify or compare annotation accuracy of reference-based classifiers,
  including atlas-referenced settings where some query types are absent from the reference.
when_to_use_negative:
- Query datasets lack credible ground-truth cell-type labels, so accuracy cannot be
  tabulated
- You need a single-method annotation result rather than a comparative benchmark across
  methods
- Query cell types are completely covered by a well-matched specific reference, where
  the exploratory atlas-referenced setting (and its labeled-accuracy caveats) does
  not apply
tools:
- name: scMAGIC
  role: Method under test; two-round reference-based classifier run via scMAGIC_atlas
    with MCA/HCL as atlas reference
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: scMCA
  role: Atlas-native comparator classifier for mouse queries via MCA
  repo: https://github.com/ggjlab/scMCA
- name: scHCL
  role: Atlas-native comparator classifier for human queries via HCL
  repo: https://github.com/ggjlab/scHCL
- name: Seurat
  role: Competing RBC method in the CDI stability regression (slope Z-score 0.85)
- name: scClassify
  role: Competing tree-based RBC method in the CDI stability regression (slope Z-score
    1.2)
  repo: https://github.com/SydneyBioX/scClassify_analysis
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:b86a7f6e1cc443d139aa536bffb11e1d57dccccec80471e266012ae2d99ce166
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - Seurat
  - scClassify
  - scHCL
  - scMAGIC
  - scMCA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# single-cell-accuracy-benchmarking

## Summary

Benchmark single-cell annotation accuracy by running a query classifier (e.g. scMAGIC, scMCA, scHCL) against datasets with known true cell-type labels and comparing per-method accuracy and labeled balanced accuracy across matched method-dataset pairs.

## When to use

Use when query scRNA-seq datasets have known ground-truth cell-type labels and you need to quantify or compare annotation accuracy of reference-based classifiers, including atlas-referenced settings where some query types are absent from the reference.

## When NOT to use

- Query datasets lack credible ground-truth cell-type labels, so accuracy cannot be tabulated
- You need a single-method annotation result rather than a comparative benchmark across methods
- Query cell types are completely covered by a well-matched specific reference, where the exploratory atlas-referenced setting (and its labeled-accuracy caveats) does not apply

## Inputs

- Query scRNA-seq expression matrices with true cell-type labels (e.g. GSE115746, GSE92332, GSE84133, GSE132044)
- Reference atlas expression matrix and marker-gene table (MCA_ref, HCL_ref)
- Annotation predictions from each method-dataset pair

## Outputs

- Accuracy and labeled accuracy / labeled balanced accuracy per method-dataset pair
- Comparison table (e.g. scMAGIC accuracy 0.88–0.98 vs scMCA/scHCL labeled balanced accuracy 0.25–0.46)
- Figure 7A-style bar-plot comparison of method accuracies
- CDI regression table with slope Z-scores per method

## How to apply

Load the reference atlas expression matrices and marker-gene tables shipped with the tool (e.g. data('MCA_ref') and data('HCL_ref') in scMAGIC) and download query datasets with true labels (e.g. mouse neocortex GSE115746, mouse duodenum GSE92332, human pancreas GSE84133, human PBMC GSE132044). Run the method under test on each query with fixed settings — for scMAGIC_atlas: type_ref='sum-counts', use_RUVseq=FALSE, min_cell=5, num_threads=10 — using MCA for mouse and HCL for human queries, and run comparator tools (scMCA, scHCL) on the same inputs. Compute accuracy by tabulating predicted versus true labels with table(true_label, predicted_label), and compute labeled accuracy by treating cell types absent from the atlas as unassignable. To test robustness, rank benchmark tests by a classification difficulty index (CDI) computed from an F1-ASW batch-effect score and the proportion of classifiable query cells, fit accuracy-versus-CDI-rank with lm(), and convert slopes to Z-slopes divided by their standard errors. Compare slope Z-scores against reported anchors (scMAGIC 2.32, Seurat v4 0.85, scClassify 1.2, scSemiCluster 0.72) to judge performance stability.

## Related tools

- **scMAGIC** (Method under test; two-round reference-based classifier run via scMAGIC_atlas with MCA/HCL as atlas reference) — https://github.com/TianLab-Bioinfo/scMAGIC
- **scMCA** (Atlas-native comparator classifier for mouse queries via MCA) — https://github.com/ggjlab/scMCA
- **scHCL** (Atlas-native comparator classifier for human queries via HCL) — https://github.com/ggjlab/scHCL
- **Seurat** (Competing RBC method in the CDI stability regression (slope Z-score 0.85))
- **scClassify** (Competing tree-based RBC method in the CDI stability regression (slope Z-score 1.2)) — https://github.com/SydneyBioX/scClassify_analysis

## Examples

```
output.scMAGIC <- scMAGIC_atlas(exp_sc_mat, MCA_ref, atlas = 'MCA', type_ref = 'sum-counts', use_RUVseq = FALSE, min_cell = 5, num_threads = 10)
```

## Evaluation signals

- scMAGIC accuracy on the four atlas-annotated query datasets falls in the 0.88–0.98 range with balanced accuracy 0.77–0.88
- scMAGIC labeled accuracy is 0.92–0.98 with labeled balanced accuracy 0.85–0.96, exceeding scMCA/scHCL labeled balanced accuracy of 0.25–0.46
- Each method-dataset pair's accuracy is reproducible from table(true_label, predicted_label) on the same predictions
- CDI slope Z-scores reproduce reported anchors: scMAGIC 2.32, Seurat v4 0.85, scClassify 1.2, scSemiCluster 0.72
- Validation expectation: scMAGIC's accuracy and labeled accuracy on the four atlas-annotated queries exceed scMCA and scHCL, consistent with the reported benchmark result

## Limitations

- A query dataset may include cell types not covered by the atlas (e.g. the mouse duodenum dataset), so atlas-absent types must be treated as unassignable and only contribute to labeled accuracy
- Atlas-native tools (scMCA, scHCL) perform poorly on these queries (labeled balanced accuracy 0.25–0.46), so low comparator scores may reflect batch effects and reference coverage rather than query quality
- No version numbers or archived code release tags are given for the benchmarked tools (beyond the scMAGIC GitHub repository), so exact reproducibility may vary

## Evidence

- [abstract] atlas-annotated accuracy ranges: "when no reference dataset is available, scMAGIC can annotate query cells with reasonably high accuracy by using an atlas dataset as the reference"
- [abstract] benchmark scope: "scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests"
- [results] CDI slope anchors: [evidence span withheld — anchor status: not_found]
- [methods] example invocation: "output.scMAGIC <- scMAGIC_atlas(exp_sc_mat, MCA_ref, atlas = 'MCA',"
- [discussion] comparator performance: "scMCA and scHCL, the tools provided by MCA and HCL, do not work well in the four query datasets"
- [readme] scMAGIC purpose (README): "is a novel single-cell classification tool using well-annotated bulk or single-cell RNA-seq data as the reference"
