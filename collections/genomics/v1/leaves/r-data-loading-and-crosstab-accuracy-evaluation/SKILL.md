---
name: r-data-loading-and-crosstab-accuracy-evaluation
description: 'Use when reproducing the single-round vs two-round scMAGIC ablation:
  query/reference matrices must be read via readRDS, predictions extracted, and accuracy
  computed by cross-tabulating predicted against true labels.'
when_to_use_negative:
- Reference and query datasets have no significant batch effects — the ablation's
  large accuracy gap (e.g. 0.98→0.33) will not reproduce, so the comparison is uninformative
- Query cell types are not covered by the reference dataset at all — this skill evaluates
  the RBC ablation, not scMAGIC's handling of uncovered cell types
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMAGIC
  role: Performs the two-round reference-based classification being evaluated; runs
    in single_round and default (two-round) modes
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: Seurat
  role: R framework compatible with scMAGIC input/output objects for loading and handling
    single-cell expression data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:6e1a2f82c8a0e4cf93f91aa07b035f8d64b73d5d591107b43e5077d71ab7f2be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - Seurat
  - scMAGIC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# r-data-loading-and-crosstab-accuracy-evaluation

## Summary

An R-based evaluation routine from the scMAGIC ablation study that loads reference/query scRNA-seq expression matrices and labels from .Rdata/.RDS files, runs scMAGIC in single-round and two-round RBC modes, and computes per-mode classification accuracy by cross-tabulating predicted versus true query cell-type labels.

## When to use

Use when reproducing the single-round vs two-round scMAGIC ablation: query/reference matrices must be read via readRDS, predictions extracted, and accuracy computed by cross-tabulating predicted against true labels.

## When NOT to use

- Reference and query datasets have no significant batch effects — the ablation's large accuracy gap (e.g. 0.98→0.33) will not reproduce, so the comparison is uninformative
- Query cell types are not covered by the reference dataset at all — this skill evaluates the RBC ablation, not scMAGIC's handling of uncovered cell types

## Inputs

- Tasic SMART-seq reference dataset (1727 mouse primary visual cortex cells) as .Rdata/.RDS expression matrix with labels
- Campbell Drop-seq query dataset (mouse hypothalamic arcuate-median eminence complex cells) expression matrix with true labels
- scMAGIC output object (output.scMAGIC) containing the scMAGIC.tag predicted-label field for each run mode

## Outputs

- Predicted cell-type label vector per query cell (output.scMAGIC$scMAGIC.tag) for single-round and two-round modes
- Cross-tabulation (table(label_sc, pred.scMAGIC)) of true vs predicted labels per mode
- single_vs_two_round_accuracy.csv containing per-mode accuracies and their difference

## How to apply

Download the Tasic SMART-seq reference (1727 mouse primary visual cortex cells) and the Drop-seq Campbell query dataset (mouse hypothalamic arcuate-median eminence complex cells) from the Drizzle-Zhang/scMAGIC_scripts repository and load expression matrices and labels with readRDS (e.g. list.Ref <- readRDS('Tasic.Rdata')). Run scMAGIC twice on the same inputs: once with the single_round argument so classification stops after the first correlation-plus-marker-gene round, and once with defaults so the second-round RBC executes, where confidently validated query cells become the new reference (atlas = 'MCA', num_threads = 4). Extract predicted labels from output.scMAGIC$scMAGIC.tag for each run. Compute accuracy for both modes by cross-tabulating predictions against true labels with table(label_sc, pred.scMAGIC) and summing the diagonal over total cells. Assemble the two accuracies and their difference (expected ~0.98 vs ~0.33 and ~0.85 vs ~0.34 under significant batch effects) into single_vs_two_round_accuracy.csv.

## Related tools

- **scMAGIC** (Performs the two-round reference-based classification being evaluated; runs in single_round and default (two-round) modes) — https://github.com/TianLab-Bioinfo/scMAGIC
- **Seurat** (R framework compatible with scMAGIC input/output objects for loading and handling single-cell expression data)

## Examples

```
list.Ref <- readRDS('Tasic.Rdata'); output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4); acc <- sum(diag(table(label_sc, output.scMAGIC$scMAGIC.tag)))/length(label_sc)
```

## Evaluation signals

- Two-round scMAGIC accuracy must be substantially higher than single-round accuracy in both benchmarks with significant batch effects
- Direction and magnitude of the drop should match Figure 6A benchmarks: ~0.98→0.33 (low-depth reference scenario) and ~0.85→0.34 (cross-species situation)
- table(label_sc, pred.scMAGIC) must have row and column names covering the true query cell-type labels, with accuracy equal to the diagonal sum divided by total cells
- output.scMAGIC$scMAGIC.tag must be a non-empty label vector of the same length as the number of query cells for each run

## Limitations

- No version numbers for R, Seurat, or scMAGIC are stated beyond the GitHub repository link, so exact reproducibility may vary
- Accuracy conclusions depend on the specific Tasic/Campbell benchmark pair; the second-round RBC benefit is only demonstrated under significant reference-query batch effects
- Dataset accession codes are deferred to Materials and Methods and not given inline

## Evidence

- [methods] Tasic reference dataset description: "The reference dataset includes 1727 mouse primary visual cortex cells profiled by SMART-seq."
- [methods] Campbell query dataset description: "The target dataset is from a study profiling mouse hypothalamic arcuate-median eminence complex cells by Drop-seq"
- [methods] scMAGIC invocation with atlas and threads: "output.scMAGIC <- scMAGIC(exp_sc_mat, ref.mtx, ref.labels, atlas = 'MCA', num_threads = 4)"
- [intro] Second-round RBC batch effect rationale: "scMAGIC introduces a second-round RBC in which those query cells whose cell identifies are confidently validated in the first-round RBC are used as the new reference"
- [discussion] Repository maintenance statement: "The source code is maintained at https://github.com/TianLab-Bioinfo/scMAGIC."
- [readme] scMAGIC README overview of the tool being evaluated: [evidence span withheld — anchor status: not_found]
