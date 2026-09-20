---
name: ablation-comparison-of-pipeline-stages
description: Use when you need to attribute measured performance of a multi-stage
  classification or analysis pipeline to a specific stage, especially to test whether
  a stage added to overcome batch effects or other confounders is genuinely critical.
when_to_use_negative:
- The pipeline has only one stage or no parameter exists to disable the stage under
  test, so an ablated variant cannot be run on identical inputs
- Query and reference data lack significant batch effects or uncovered cell types,
  so the stage under test (e.g. second-round RBC) is expected to have little effect
  and the ablation would not be informative
- True query labels are unavailable, making accuracy-based comparison between modes
  impossible
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: scMAGIC
  role: Classification pipeline ablated between single-round and two-round RBC modes
    via the single_round argument; predictions extracted from output.scMAGIC$scMAGIC.tag
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: Seurat
  role: Optional preprocessing and visualization (NormalizeData, FindVariableFeatures,
    ScaleData, RunPCA, RunUMAP) to inspect per-mode classification results; scMAGIC
    input/output is Seurat-compatible via scMAGIC_Seurat
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:238f173d2078d26a324ce52a0c573ab9868e470a5e93b6e6a0924bcd10011132
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - Seurat
  - scMAGIC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ablation-comparison-of-pipeline-stages

## Summary

Quantify the contribution of an individual pipeline stage (e.g. scMAGIC's second-round reference-based classification) by running the full pipeline versus a variant with the stage disabled, then computing accuracy in both modes on the same query data and reference to isolate the stage's effect.

## When to use

Use when you need to attribute measured performance of a multi-stage classification or analysis pipeline to a specific stage, especially to test whether a stage added to overcome batch effects or other confounders is genuinely critical.

## When NOT to use

- The pipeline has only one stage or no parameter exists to disable the stage under test, so an ablated variant cannot be run on identical inputs
- Query and reference data lack significant batch effects or uncovered cell types, so the stage under test (e.g. second-round RBC) is expected to have little effect and the ablation would not be informative
- True query labels are unavailable, making accuracy-based comparison between modes impossible

## Inputs

- Reference scRNA-seq expression matrix and cell-type labels (e.g. Tasic.Rdata: 1727 SMART-seq mouse primary visual cortex cells, list.Ref$mat_exp and list.Ref$label)
- Query scRNA-seq expression matrix and true labels (e.g. Campbell_2k.Rdata: 2000 Drop-seq mouse hypothalamic arcuate-median eminence cells, list.demo$mat_exp and list.demo$label)
- A toggleable pipeline parameter disabling the stage under test (scMAGIC single_round argument)

## Outputs

- Predicted cell-type labels per query cell for each pipeline mode (output.scMAGIC$scMAGIC.tag)
- Accuracy values for the ablated and full pipelines plus their difference (single_vs_two_round_accuracy.csv)
- Cross-tabulation of true vs predicted labels (table(label_sc, pred.scMAGIC)) per mode

## How to apply

Download or prepare matched reference and query datasets exhibiting the confounder the stage addresses — here the Tasic SMART-seq reference (1727 mouse primary visual cortex cells) and the Drop-seq Campbell hypothalamic arcuate-median eminence query, both loaded via readRDS from the scMAGIC_scripts repository. Run the pipeline twice on identical inputs: once with the stage under test disabled (scMAGIC with the single_round argument, stopping after the first correlation-plus-marker-gene round) and once with the full pipeline (default settings, where first-round-validated query cells become the new reference for a second-round RBC). Extract predictions (output.scMAGIC$scMAGIC.tag) from each run, cross-tabulate against true query labels with table(label_sc, pred.scMAGIC) to compute per-mode accuracy, and assemble the accuracies and their difference into a summary file (single_vs_two_round_accuracy.csv). Validate by checking that the direction and magnitude of the drop match published benchmark results (e.g. 0.98→0.33 and 0.85→0.34 under significant reference-query batch effects).

## Related tools

- **scMAGIC** (Classification pipeline ablated between single-round and two-round RBC modes via the single_round argument; predictions extracted from output.scMAGIC$scMAGIC.tag) — https://github.com/TianLab-Bioinfo/scMAGIC
- **Seurat** (Optional preprocessing and visualization (NormalizeData, FindVariableFeatures, ScaleData, RunPCA, RunUMAP) to inspect per-mode classification results; scMAGIC input/output is Seurat-compatible via scMAGIC_Seurat) — https://github.com/TianLab-Bioinfo/scMAGIC

## Examples

```
output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4); pred.scMAGIC <- output.scMAGIC$scMAGIC.tag
```

## Evaluation signals

- The full two-round run attains substantially higher accuracy than the single_round run (published: 0.98 vs 0.33 and 0.85 vs 0.34 under significant batch effects)
- The accuracy drop direction and magnitude are consistent with the Figure 6A benchmark results (0.98→0.33 and 0.85→0.34)
- Both modes are run on identical reference (Tasic.Rdata) and query (Campbell_2k.Rdata) inputs; only the single_round flag differs
- Accuracy is computed from a confusion matrix against true query labels: table(label_sc, pred.scMAGIC)
- A single_vs_two_round_accuracy.csv contains per-mode accuracies and their difference with no missing modes

## Limitations

- The ablation is only informative in scenarios where the stage matters — the article's second-round RBC advantage is 'especially significant when the cell types in the query dataset are not completely covered by the reference dataset and when there exist significant batch effects'
- scMAGIC is implemented in R 4.0.0 with specific dependency versions; on Windows, method_HVGene = 'SciBet_R' must be set, and reticulate/numpy must be configured before running
- No version numbers or archived code releases are stated for auxiliary tools, which may complicate exact reproduction

## Evidence

- [abstract] Second-round RBC rationale: the second round eliminates reference-query batch effects by using validated query cells as the new reference: "those query cells whose cell identities are confidently validated in the first round are used as a new reference to again classify query cells, therefore eliminating the batch effects"
- [intro] First-round classification mechanism that the ablation leaves intact: "first identify a candidate reference cell type for a query cell by expression profile correlation and then validate its cell identity using the marker genes of the"
- [intro] Benchmark scope in which the second-round RBC advantage is expected: "especially when the cell types in the query dataset are not completely covered by the reference dataset and when there exist significant batch effects"
- [readme] Concrete invocation for the classification run used in the ablation: "output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)"
- [readme] Accuracy evaluation procedure via confusion table against true labels: "# classification results
table(label_sc, pred.scMAGIC)"
