---
name: confusion-matrix-accuracy-computation
description: 'Use when evaluating a cell-type annotation method against known true
  labels: you have per-cell predicted labels and ground-truth labels for query datasets
  (e.g. GSE115746, GSE92332, GSE84133, GSE132044) and need accuracy and labeled accuracy.'
when_to_use_negative:
- No ground-truth (true) cell-type labels exist for the query dataset, so a confusion
  matrix cannot be tabulated
- Query cell types are entirely uncovered by the reference atlas, making even 'labeled'
  accuracy meaningless rather than just excluding uncovered types
- Evaluating unsupervised clustering outputs without a label-matching step between
  clusters and reference cell types
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: scMAGIC
  role: Method whose predictions are tabulated against true labels via the confusion
    matrix; provides scMAGIC_atlas for annotation with MCA/HCL atlas references
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: scMCA
  role: Atlas-native mouse classifier whose predictions are compared against scMAGIC
    using the same accuracy/labeled-accuracy metrics
  repo: https://github.com/ggjlab/scMCA
- name: scHCL
  role: Atlas-native human classifier whose predictions are compared against scMAGIC
    using the same accuracy/labeled-accuracy metrics
  repo: https://github.com/ggjlab/scHCL
- name: SciBet
  role: Competing RBC method benchmarked against scMAGIC across 86 tests using the
    same accuracy metrics
  repo: https://github.com/PaulingLiu/scibet
- name: SingleR
  role: Competing statistical-metric-based RBC method benchmarked against scMAGIC
    using the same accuracy metrics
  repo: https://github.com/dviraran/SingleR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:9a416fd922914cf85d5d3f18c100e99bd68c22d4a527360095638fcfcc221a00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - SciBet
  - SingleR
  - scHCL
  - scMAGIC
  - scMCA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# confusion-matrix-accuracy-computation

## Summary

Compute per-method accuracy and labeled accuracy by tabulating predicted versus true cell-type labels with table(true_label, predicted_label), treating cell types absent from the reference/atlas as unassignable. This is the core evaluation metric used to benchmark scMAGIC against atlas-native tools (scMCA, scHCL) and 13 competing RBC methods across 86 benchmark tests.

## When to use

Use when evaluating a cell-type annotation method against known true labels: you have per-cell predicted labels and ground-truth labels for query datasets (e.g. GSE115746, GSE92332, GSE84133, GSE132044) and need accuracy and labeled accuracy.

## When NOT to use

- No ground-truth (true) cell-type labels exist for the query dataset, so a confusion matrix cannot be tabulated
- Query cell types are entirely uncovered by the reference atlas, making even 'labeled' accuracy meaningless rather than just excluding uncovered types
- Evaluating unsupervised clustering outputs without a label-matching step between clusters and reference cell types

## Inputs

- Vector of true cell-type labels for query cells (e.g. from GSE115746 mouse neocortex, GSE92332 mouse duodenum, GSE84133 human pancreas, GSE132044 human PBMC)
- Vector of predicted cell-type labels from each annotation method (scMAGIC_atlas, scMCA, scHCL, or other RBC methods)
- Optional: reference/atlas cell-type list to define which query cell types are 'labeled' (covered by the atlas)

## Outputs

- Confusion matrix from table(true_label, predicted_label)
- Overall accuracy and balanced accuracy per method-dataset pair
- Labeled accuracy and labeled balanced accuracy restricted to atlas-covered cell types
- Assembled accuracy comparison table and bar-plot figure (e.g. Figure 7A-style)

## How to apply

For each method-dataset pair, cross-tabulate predicted versus true cell labels with table(true_label, predicted_label) in R and compute overall accuracy and balanced accuracy. Compute labeled accuracy by restricting the tabulation to cell types present in the reference atlas, treating query cell types absent from the atlas (e.g. cell types in the mouse duodenum dataset not covered by MCA) as unassignable rather than errors. Run the same computation on all competing methods (scMAGIC_atlas, scMCA, scHCL) with identical query datasets and true labels so the comparison is fair. In the scMAGIC atlas benchmark this yielded accuracy 0.88–0.98 (balanced 0.77–0.88) and labeled accuracy 0.92–0.98 (labeled balanced 0.85–0.96) for scMAGIC versus labeled balanced accuracy of only 0.25–0.46 for scMCA/scHCL. Finally assemble the accuracy/labeled-accuracy comparison table and visualize it (e.g. the Figure 7A bar-plot of scMAGIC versus scMCA/scHCL).

## Related tools

- **scMAGIC** (Method whose predictions are tabulated against true labels via the confusion matrix; provides scMAGIC_atlas for annotation with MCA/HCL atlas references) — https://github.com/TianLab-Bioinfo/scMAGIC
- **scMCA** (Atlas-native mouse classifier whose predictions are compared against scMAGIC using the same accuracy/labeled-accuracy metrics) — https://github.com/ggjlab/scMCA
- **scHCL** (Atlas-native human classifier whose predictions are compared against scMAGIC using the same accuracy/labeled-accuracy metrics) — https://github.com/ggjlab/scHCL
- **SciBet** (Competing RBC method benchmarked against scMAGIC across 86 tests using the same accuracy metrics) — https://github.com/PaulingLiu/scibet
- **SingleR** (Competing statistical-metric-based RBC method benchmarked against scMAGIC using the same accuracy metrics) — https://github.com/dviraran/SingleR

## Examples

```
conf <- table(true_label, predicted_label); acc <- sum(diag(conf)) / sum(conf); lab <- true_label %in% atlas_types; acc_lab <- sum(true_label[lab] == predicted_label[lab]) / sum(lab)
```

## Evaluation signals

- Confusion matrix dimensions equal the number of unique cell types observed in true and predicted labels combined; diagonal cells contain correct assignments
- Accuracy values for scMAGIC on the four atlas-annotated query datasets fall in the reported range 0.88–0.98 (balanced accuracy 0.77–0.88)
- Labeled accuracy values fall in 0.92–0.98 (labeled balanced 0.85–0.96), and scMAGIC exceeds scMCA/scHCL labeled balanced accuracy (0.25–0.46) on every dataset pair
- Cell types absent from the atlas are excluded from labeled-accuracy denominators rather than counted as errors (e.g. duodenum types not in MCA)
- All methods are evaluated on identical query datasets and true labels so the comparison table rows are directly comparable

## Limitations

- Accuracy can be misleading when query cell types are not covered by the reference; labeled accuracy partially addresses this but discards uncovered cells from evaluation
- The article does not report per-cell-type precision/recall breakdowns in this section, and dataset accession codes and tool version numbers are deferred to Materials and Methods
- Class imbalance across cell types requires reporting balanced accuracy alongside raw accuracy to avoid over-weighting abundant types

## Evidence

- [methods] labeled accuracy is computed by tabulating predicted versus true labels with table(true_label, predicted_label), treating cell types absent from the atlas as unassignable: [evidence span withheld — anchor status: not_found]
- [results] scMAGIC achieves accuracy 0.88–0.98 and labeled accuracy 0.92–0.98 while scMCA and scHCL reach only 0.25–0.46 labeled balanced accuracy: [evidence span withheld — anchor status: not_found]
- [discussion] a query dataset may include cell types not covered by the atlas, motivating labeled accuracy: "a query dataset may include some cell types not covered by the atlas, such as the mouse duodenum dataset"
- [abstract] scMAGIC significantly outperforms 13 competing RBC methods across 86 benchmark tests: "scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests"
- [discussion] scMCA and scHCL, the atlas-native tools, do not work well on the four query datasets: "scMCA and scHCL, the tools provided by MCA and HCL, do not work well in the four query datasets"
