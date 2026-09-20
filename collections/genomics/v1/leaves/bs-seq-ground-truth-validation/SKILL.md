---
name: bs-seq-ground-truth-validation
description: Use when assessing whether a nanopore methylation caller (DeepBAM, Dorado)
  reproduces bisulfite-sequencing ground truth on human or plant R10.4 datasets, at
  single-molecule CpG and whole-genome frequency levels.
when_to_use_negative:
- Datasets lack high-confidence bisulfite-derived positive/negative CpG site labels,
  so no ground truth exists to balance or score against
- The task is raw modification calling on R10.4 data rather than benchmarking caller
  accuracy against BS-seq
- Whole-genome methylation frequencies alone are already computed and no single-molecule
  validation is needed
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: DeepBAM
  role: Tool under test; its per-read CpG methylation calls ('extract_and_call_mods')
    are scored against bisulfite ground truth
  repo: github.com/xiaochuanle/DeepBAM
- name: Dorado
  role: Baseline methylation caller whose dataset-dependent CpG performance is benchmarked
    against the same BS-seq site sets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:629d2fb06479c2a940fe49b327067a995add21aef094b3817d65319a6caf601e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  tools:
  - DeepBAM
  - Dorado
  - libtorch
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bs-seq-ground-truth-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates nanopore CpG methylation calls (e.g., DeepBAM vs Dorado on R10.4 POD5 datasets) against high-confidence bisulfite-defined methylated/unmethylated sites, scoring single-molecule performance with ROC AUC, F1, and balanced accuracy, and whole-genome frequency agreement via BS-seq correlation.

## When to use

Use when assessing whether a nanopore methylation caller (DeepBAM, Dorado) reproduces bisulfite-sequencing ground truth on human or plant R10.4 datasets, at single-molecule CpG and whole-genome frequency levels.

## When NOT to use

- Datasets lack high-confidence bisulfite-derived positive/negative CpG site labels, so no ground truth exists to balance or score against
- The task is raw modification calling on R10.4 data rather than benchmarking caller accuracy against BS-seq
- Whole-genome methylation frequencies alone are already computed and no single-molecule validation is needed

## Inputs

- POD5 nanopore signal files (R10.4) for five heterogeneous human and plant datasets
- Sorted BAM alignments and reference genome FASTA per dataset
- High-confidence bisulfite-defined positive and negative CpG methylation site lists (BS-seq ground truth)
- Per-read CpG methylation call TSVs from DeepBAM and Dorado

## Outputs

- Per-dataset, per-tool ROC AUC, F1 score, and balanced accuracy compiled into methylation_calling_metrics.tsv
- ROC curves per tool per dataset
- Correlation coefficients between tool-derived whole-genome CpG methylation frequencies and BS-seq frequencies

## How to apply

Run the methylation caller (e.g., DeepBAM 'extract_and_call_mods' with kmer_size=51, motif_type=CG, batch_size=1024) and the baseline Dorado caller on each POD5 dataset with its sorted BAM and reference genome. Align each tool's called CpG sites to the high-confidence bisulfite-defined positive and negative methylation sites per dataset. Balance the positive and negative CpG classes per dataset, then compute ROC AUC, balanced accuracy, and F1 score per tool per dataset. Compile metrics into a comparison table (e.g., methylation_calling_metrics.tsv) and additionally correlate whole-genome CpG methylation frequencies with BS-seq. Judging correctness relies on the paper's benchmark ranges: DeepBAM AUCs >=0.956 (average 98.47%), F1 average 94.97%, balanced accuracy 92.20%-98.96%, and BS-seq correlations >0.95 on four of five datasets, outperforming Dorado in all instances.

## Related tools

- **DeepBAM** (Tool under test; its per-read CpG methylation calls ('extract_and_call_mods') are scored against bisulfite ground truth) — github.com/xiaochuanle/DeepBAM
- **Dorado** (Baseline methylation caller whose dataset-dependent CpG performance is benchmarked against the same BS-seq site sets)

## Examples

```
from sklearn.metrics import roc_auc_score, f1_score, balanced_accuracy_score; auc = roc_auc_score(bs_labels, pred_scores)
```

## Evaluation signals

- Reproduced DeepBAM AUCs are >=0.956 across all five datasets with average AUC of 98.47%
- DeepBAM average F1 score is 94.97% and balanced accuracies fall within 92.20%-98.96%
- Dorado AUCs fall within the reported 0.8825-0.9852 range, confirming dataset-dependent variability
- Whole-genome CpG methylation frequency correlation with BS-seq exceeds 0.95 on four of five datasets for DeepBAM
- Positive and negative CpG classes are balanced per dataset before metric computation

## Limitations

- No accession IDs or hosting locations are given for the three plant datasets or human cell line datasets, complicating direct reproduction
- Ongoing optimization of DeepBAM is acknowledged as necessary, implying incomplete current performance
- Quantitative speed/memory comparisons are deferred to supplementary material not present in the text

## Evidence
