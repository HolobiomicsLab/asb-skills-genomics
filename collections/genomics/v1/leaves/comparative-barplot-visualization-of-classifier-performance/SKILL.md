---
name: comparative-barplot-visualization-of-classifier-performance
description: Use when benchmarking two or more single-cell annotation/classification
  methods on the same query datasets and you need a figure that juxtaposes accuracy
  (or labeled/balanced accuracy) per method-dataset pair as grouped bars.
when_to_use_negative:
- The analysis produces continuous per-cell confidence scores rather than discrete
  predicted labels, so accuracy-based bars would be misleading
- Only a single method on a single dataset is being evaluated — a bar-plot comparison
  needs at least two methods or conditions to contrast
- Query cell types are fully covered by the reference atlas, making the labeled-accuracy
  distinction (unassignable types) unnecessary and the metric identical to plain accuracy
edam_operation: http://edamontology.org/operation_0579
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scMAGIC
  role: Method under comparison; its scMAGIC_atlas accuracy/labeled-accuracy values
    form one bar group in the plot
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: scMCA
  role: Atlas-native baseline classifier (MCA reference) whose accuracy values are
    plotted against scMAGIC for mouse query datasets
- name: scHCL
  role: Atlas-native baseline classifier (HCL reference) whose accuracy values are
    plotted against scMAGIC for human query datasets
- name: ggplot2
  role: R plotting library used to render the Figure 7A grouped bar-plot comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:cc630b993e2f9a72238001abd35e5a26729cb75c97bc0bad514e079916748ee9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - ggplot2
  - scHCL
  - scMAGIC
  - scMCA
  edam_operation: http://edamontology.org/operation_0579
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# comparative-barplot-visualization-of-classifier-performance

## Summary

Assembles per-method accuracy and labeled-accuracy values for cell-type classifiers across multiple query datasets and renders them as a grouped bar plot (e.g. Figure 7A in scMAGIC) so that competing methods such as scMAGIC versus scMCA/scHCL can be compared visually on the same benchmark tasks.

## When to use

Use when benchmarking two or more single-cell annotation/classification methods on the same query datasets and you need a figure that juxtaposes accuracy (or labeled/balanced accuracy) per method-dataset pair as grouped bars.

## When NOT to use

- The analysis produces continuous per-cell confidence scores rather than discrete predicted labels, so accuracy-based bars would be misleading
- Only a single method on a single dataset is being evaluated — a bar-plot comparison needs at least two methods or conditions to contrast
- Query cell types are fully covered by the reference atlas, making the labeled-accuracy distinction (unassignable types) unnecessary and the metric identical to plain accuracy

## Inputs

- Per-method-dataset prediction vectors (predicted cell-type labels) and ground-truth label vectors for the four query datasets (GSE115746, GSE92332, GSE84133, GSE132044)
- Accuracy and labeled-accuracy (and balanced-accuracy) values computed via table(true_label, predicted_label) for scMAGIC, scMCA and scHCL
- Method and dataset metadata (tool name, reference atlas MCA or HCL, query dataset name) for grouping/faceting the bars

## Outputs

- A comparison table of accuracy / labeled accuracy / balanced accuracy per method-dataset pair
- A grouped bar plot (Figure 7A-style, ggplot2) comparing scMAGIC versus scMCA/scHCL performance across the four query datasets

## How to apply

For each method-dataset pair, tabulate predicted versus true cell-type labels with table(true_label, predicted_label) and compute overall accuracy plus labeled accuracy, treating query cell types absent from the reference atlas (e.g. MCA/HCL) as unassignable for labeled accuracy. Collect the values into a comparison table — in the scMAGIC atlas benchmark this covers scMAGIC_atlas versus scMCA (mouse queries) and scHCL (human queries) on mouse neocortex GSE115746, mouse duodenum GSE92332, human pancreas GSE84133 and human PBMC GSE132044. Render the table as a grouped bar plot (Figure 7A style) with one bar group per query dataset and bars colored/faceted by method, using ggplot2 in R. Validate that the plotted scMAGIC bars exceed the atlas-native tool bars, consistent with the reported ranges (scMAGIC accuracy 0.88–0.98 versus labeled balanced accuracy of only 0.25–0.46 for scMCA/scHCL).

## Related tools

- **scMAGIC** (Method under comparison; its scMAGIC_atlas accuracy/labeled-accuracy values form one bar group in the plot) — https://github.com/TianLab-Bioinfo/scMAGIC
- **scMCA** (Atlas-native baseline classifier (MCA reference) whose accuracy values are plotted against scMAGIC for mouse query datasets)
- **scHCL** (Atlas-native baseline classifier (HCL reference) whose accuracy values are plotted against scMAGIC for human query datasets)
- **ggplot2** (R plotting library used to render the Figure 7A grouped bar-plot comparison)

## Examples

```
tab <- table(true_label, predicted_label); acc <- sum(diag(tab))/sum(tab); ggplot(acc_df, aes(dataset, accuracy, fill = method)) + geom_col(position = 'dodge')
```

## Evaluation signals

- Plotted scMAGIC accuracy on the four atlas-annotated query datasets falls in the reported 0.88–0.98 range (balanced accuracy 0.77–0.88) and labeled accuracy 0.92–0.98 (labeled balanced accuracy 0.85–0.96)
- Plotted scMCA and scHCL labeled balanced accuracy falls in the reported 0.25–0.46 range and is visibly lower than scMAGIC for every dataset
- Each bar corresponds to exactly one method-dataset pair, with the same four query datasets represented for every method
- Labeled-accuracy bars must exclude query cell types absent from the reference atlas; plain accuracy and labeled accuracy bars should differ for datasets containing unassignable types (e.g. mouse duodenum)

## Limitations

- Accuracy bars alone can hide unequal cell-type prevalence; balanced and labeled balanced accuracy should accompany plain accuracy when cell types are absent from the atlas
- The comparison depends on fixed scMAGIC_atlas settings (type_ref='sum-counts', use_RUVseq=FALSE, min_cell=5, num_threads=10); changing parameters changes the bars
- Query datasets containing cell types not covered by MCA/HCL (e.g. mouse duodenum) make plain accuracy an optimistic metric for methods that force assignment

## Evidence

- [results] scMAGIC achieves reasonably high accuracy (0.88–0.98; balanced accuracy 0.77–0.88) and labeled accuracy 0.92–0.98 with labeled balanced accuracy 0.85–0.96, while scMCA and scHCL reach a labeled balanced accuracy of only 0.25–0.46.: [evidence span withheld — anchor status: not_found]
- [methods] Compute accuracy and labeled accuracy by tabulating predicted versus true labels.: [evidence span withheld — anchor status: not_found]
- [methods] Assemble the comparison table and reproduce the Figure 7A bar-plot.: [evidence span withheld — anchor status: not_found]
- [discussion] scMCA and scHCL, the tools provided by MCA and HCL, do not work well in the four query datasets.: "scMCA and scHCL, the tools provided by MCA and HCL, do not work well in the four query datasets"
- [discussion] a query dataset may include some cell types not covered by the atlas.: "a query dataset may include some cell types not covered by the atlas, such as the mouse duodenum dataset"
