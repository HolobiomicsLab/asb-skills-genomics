---
name: precision-recall-analysis
description: Use when you have per-item continuous scores with binary ground-truth
  labels (e.g., per-read CpG methylation scores from methylated vs unmethylated control
  sets) and need to quantify and compare classification accuracy via ROC/PR curves.
when_to_use_negative:
- When ground truth is only site-level expected methylation frequency (controlled
  mixtures) rather than per-read binary labels — use site-level dispersion/agreement
  analysis instead
- When comparing tools on whole-genome or WGBS-aligned data without methylated/unmethylated
  control reads, since no per-read binary labels exist for ROC/PR computation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3295
tools:
- name: Nanopolish
  role: Per-read methylation detection tool whose log-likelihood-ratio scores are
    ranked via ROC/PR analysis
- name: Megalodon
  role: Per-read methylation detection tool whose log-probability-difference scores
    achieved the highest AUC and AUCPR
  repo: https://github.com/nanoporetech/megalodon
- name: DeepSignal
  role: Per-read methylation detection tool whose log-ratio of methylated/unmethylated
    probabilities is ranked via ROC/PR analysis
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Guppy
  role: Per-read methylation detection tool whose log-likelihood-ratio scores are
    ranked via ROC/PR analysis
- name: Tombo
  role: Per-read methylation detection tool whose significance values are ranked via
    ROC/PR analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:e3567c1c5d3540f3b30085e398ba279a920f6e060c2001d29a708c41cf3c6c63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/megalodon
  tools:
  - DeepSignal
  - Guppy
  - Megalodon
  - Nanopolish
  - Tombo
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# precision-recall-analysis

## Summary

Computes ROC curves, precision-recall curves, AUC, and AUCPR to benchmark how well binary classifiers rank positive examples. Here it is applied to per-read 5mC methylation scores from nanopore tools (Nanopolish, Megalodon, DeepSignal, Guppy, Tombo) on 0% and 100% methylated control read sets.

## When to use

Use when you have per-item continuous scores with binary ground-truth labels (e.g., per-read CpG methylation scores from methylated vs unmethylated control sets) and need to quantify and compare classification accuracy via ROC/PR curves.

## When NOT to use

- When ground truth is only site-level expected methylation frequency (controlled mixtures) rather than per-read binary labels — use site-level dispersion/agreement analysis instead
- When comparing tools on whole-genome or WGBS-aligned data without methylated/unmethylated control reads, since no per-read binary labels exist for ROC/PR computation

## Inputs

- Per-read score TSV files (columns: ID, Chr, Pos, Strand, Score) for Nanopolish, Megalodon, DeepSignal, Guppy, and Tombo from the figshare deposit (controlled mixture dataset 1)
- Binary ground-truth labels per read: 1 for reads from the 100% methylated control set, 0 for reads from the 0% methylated control set (~2400 reads per set)

## Outputs

- ROC curves and precision-recall curves per tool (roc_pr_curves.png)
- CSV of AUC and AUCPR values per tool (auc_aucpr_metrics.csv)

## How to apply

Load the per-read score TSV files (columns ID, Chr, Pos, Strand, Score) from the figshare deposit of per-read scores from controlled mixture dataset 1. Assign binary labels per read: methylated (1) for reads from the 100% methylated set and unmethylated (0) for the 0% methylated set (~2400 reads per set). Treat each tool's score as the ranking statistic — log-likelihood ratios for Nanopolish/Guppy, significance values for Tombo, log-ratio of methylated/unmethylated probabilities for DeepSignal, and log-probability differences for Megalodon — and compute ROC curves and precision-recall curves per tool with scikit-learn's roc_curve and precision_recall_curve. Compute AUC and AUCPR per tool, then plot all five tools' curves on shared axes with matplotlib and export the metrics to CSV. Acceptance: all five tools exceed 0.8 on both AUC and AUCPR, with Megalodon highest, followed by DeepSignal and Nanopolish.

## Related tools

- **Nanopolish** (Per-read methylation detection tool whose log-likelihood-ratio scores are ranked via ROC/PR analysis)
- **Megalodon** (Per-read methylation detection tool whose log-probability-difference scores achieved the highest AUC and AUCPR) — https://github.com/nanoporetech/megalodon
- **DeepSignal** (Per-read methylation detection tool whose log-ratio of methylated/unmethylated probabilities is ranked via ROC/PR analysis) — https://github.com/bioinfomaticsCSU/deepsignal
- **Guppy** (Per-read methylation detection tool whose log-likelihood-ratio scores are ranked via ROC/PR analysis)
- **Tombo** (Per-read methylation detection tool whose significance values are ranked via ROC/PR analysis)

## Examples

```
from sklearn.metrics import roc_curve, precision_recall_curve, roc_auc_score, average_precision_score; fpr, tpr, _ = roc_curve(labels, scores); auc = roc_auc_score(labels, scores); aucpr = average_precision_score(labels, scores)
```

## Evaluation signals

- All five tools achieve AUC > 0.8 and AUCPR > 0.8 on the 0%/100% methylated control read sets
- Megalodon ranks highest on both AUC and AUCPR, followed by DeepSignal and Nanopolish
- roc_pr_curves.png is produced with all five tools plotted on shared axes
- auc_aucpr_metrics.csv is produced with one AUC and one AUCPR row per tool

## Limitations

- Scores are not on a common scale across tools (log-likelihood ratios vs significance values vs log-probability differences), so only ranking-based metrics like AUC/AUCPR are comparable
- M.SssI methylated control only has ~95% efficiency, which can limit perfect labeling of positives
- Megalodon's best performance relies on GPUs to be time efficient

## Evidence

- [methods] Per-read ROC/PR benchmarking of five tools: [evidence span withheld — anchor status: not_found]
- [discussion] Megalodon best overall performance: "Megalodon provided overall the best performance but relies on GPUs to be time efficient."
- [methods] Tool-specific score semantics: [evidence span withheld — anchor status: not_found]
- [discussion] M.SssI control efficiency caveat: "the M.Sssl treatment used to generate the CpG-methylated control only has 95% efficiency"
