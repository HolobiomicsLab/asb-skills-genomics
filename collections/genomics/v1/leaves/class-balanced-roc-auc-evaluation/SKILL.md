---
name: class-balanced-roc-auc-evaluation
description: Use when comparing methylation callers' single-molecule performance across
  heterogeneous nanopore R10.4 datasets and you have bisulfite-defined positive and
  negative CpG site sets to score calls against.
when_to_use_negative:
- Evaluating whole-genome methylation frequency correlation with BS-seq (a per-site
  frequency analysis, not single-molecule calling evaluation)
- Training or feature extraction stages (extract_hc_sites, train_lstm.py) rather than
  caller benchmarking
- Data are not nanopore R10.4 CpG calls with an independent bisulfite ground truth
  — the class balancing and metrics assume bisulfite-defined positives and negatives
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: DeepBAM
  role: Methylation caller under test; its per-read CpG calls are scored against bisulfite
    ground truth with AUC, F1, and balanced accuracy
  repo: github.com/xiaochuanle/DeepBAM
- name: Dorado
  role: Official ONT baseline methylation caller whose dataset-dependent performance
    the class-balanced metrics quantify
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:611dec8105177362fc9d561c1d65a97c9bbcbe7fbe79e822b292d60c86f6fa74
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# class-balanced-roc-auc-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark single-molecule CpG methylation callers (DeepBAM vs Dorado) by aligning their per-read calls to bisulfite-defined ground-truth CpG sites, balancing positive/negative classes, and computing ROC AUC, F1, and balanced accuracy per R10.4 dataset.

## When to use

Use when comparing methylation callers' single-molecule performance across heterogeneous nanopore R10.4 datasets and you have bisulfite-defined positive and negative CpG site sets to score calls against.

## When NOT to use

- Evaluating whole-genome methylation frequency correlation with BS-seq (a per-site frequency analysis, not single-molecule calling evaluation)
- Training or feature extraction stages (extract_hc_sites, train_lstm.py) rather than caller benchmarking
- Data are not nanopore R10.4 CpG calls with an independent bisulfite ground truth — the class balancing and metrics assume bisulfite-defined positives and negatives

## Inputs

- Per-read CpG methylation call TSVs from DeepBAM (extract_and_call_mods)
- Per-read CpG methylation calls from Dorado on the same POD5 inputs
- High-confidence bisulfite-defined positive and negative methylation site lists per dataset
- Sorted BAM, reference genome, and POD5 datasets (human cell lines and plant R10.4 data)

## Outputs

- methylation_calling_metrics.tsv — per-dataset, per-tool ROC AUC, balanced accuracy, and F1 scores
- Per-dataset class-balanced positive/negative CpG evaluation sets
- Comparison table showing DeepBAM vs Dorado performance gaps (e.g., AUC improvement up to 7.36%)

## How to apply

Run each caller (DeepBAM 'extract_and_call_mods' with kmer_size=51, motif_type=CG, batch_size=1024; and Dorado with the official methylation model) on the same POD5 files with the same references to obtain per-read CpG methylation TSVs. Align each tool's called CpG sites to the high-confidence bisulfite-defined positive and negative methylation sites for each dataset. Balance the positive and negative CpG classes per dataset to remove methylation-frequency confounds, then compute ROC AUC, balanced accuracy, and F1 for DeepBAM and Dorado. Compile the per-dataset, per-tool metrics into a single table (methylation_calling_metrics.tsv). Judge correctness against the reported ranges: DeepBAM AUCs >=0.956 with average AUC 98.47% (up to 7.36% over Dorado), average F1 94.97%, balanced accuracies within 92.20%-98.96%, and Dorado AUCs within 0.8825-0.9852.

## Related tools

- **DeepBAM** (Methylation caller under test; its per-read CpG calls are scored against bisulfite ground truth with AUC, F1, and balanced accuracy) — github.com/xiaochuanle/DeepBAM
- **Dorado** (Official ONT baseline methylation caller whose dataset-dependent performance the class-balanced metrics quantify)

## Evaluation signals

- Every reproduced DeepBAM per-dataset AUC is >=0.956 (stability threshold stated in the article)
- Average DeepBAM AUC across the five datasets is 98.47%, with up to 7.36% improvement over Dorado
- Average DeepBAM F1 is 94.97% (up to 16.24% improvement), and balanced accuracies fall within 92.20%-98.96%
- Dorado AUCs land within the reported 0.8825-0.9852 range, confirming its dataset-dependent variability
- methylation_calling_metrics.tsv contains one row per dataset per tool (5 datasets x 2 tools) with AUC, F1, and balanced accuracy columns populated

## Limitations

- Dorado's performance varies significantly across heterogeneous human and plant datasets, so cross-dataset comparisons must be per-dataset and class-balanced, not pooled
- Fully-methylated and fully-unmethylated treated DNAs revealed significant performance variances that complicate ground-truth assumptions
- No accession IDs or hosting locations are given for the three plant datasets or human cell line datasets in the text, hindering exact reproduction
- Article acknowledges ongoing optimization of DeepBAM is necessary, so absolute thresholds may shift in newer releases

## Evidence

- [other] Balance the positive and negative CpG classes per dataset and compute ROC AUC, balanced accuracy, and F1 score for DeepBAM and Dorado.: [evidence span withheld — anchor status: not_found]
