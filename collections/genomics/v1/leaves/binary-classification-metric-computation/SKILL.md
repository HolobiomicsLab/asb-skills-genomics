---
name: binary-classification-metric-computation
description: Use when per-read scores from methylation-calling tools on 0%/100% methylated
  control reads need binary classification evaluation (ROC/PR curves, AUC, AUCPR)
  to rank tool accuracy.
when_to_use_negative:
- When the goal is per-site (genome-level) methylation frequency agreement rather
  than per-read binary discrimination — the article benchmarks per-site predictions
  separately via dispersion vs expected methylation frequency
- When ground truth is not binary (e.g. controlled mixtures at intermediate methylation
  frequencies, or WGBS comparisons) — ROC/PR labeling with 0/1 methylated control
  sets does not apply
- When comparing raw score values across tools rather than rankings — score types
  differ (log-likelihood ratios vs significance values vs log-probability differences)
  and are not interchangeable
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Megalodon
  role: Per-read methylation scoring tool; expected to achieve the highest AUC and
    AUCPR among the five benchmarked tools
  repo: https://github.com/nanoporetech/megalodon
- name: DeepSignal
  role: Per-read methylation scoring tool whose log-ratio methylated/unmethylated
    probability scores are ranked for ROC/PR evaluation
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Nanopolish
  role: Per-read methylation scoring tool; log-likelihood ratio scores used as the
    ROC/PR ranking statistic
- name: Guppy
  role: Per-read methylation scoring tool; log-likelihood ratio scores used as the
    ROC/PR ranking statistic
- name: Tombo
  role: Per-read methylation scoring tool; significance values used as the ROC/PR
    ranking statistic
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:ed88100dd728ef8ef6cfc036fddb8f3784a333ff238515d30459af066c5288e8
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
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# binary-classification-metric-computation

## Summary

Computes ROC curves, precision-recall curves, AUC, and AUCPR to benchmark per-read 5mC detection tools (Nanopolish, Megalodon, DeepSignal, Guppy, Tombo) against binary ground-truth labels from 0% and 100% methylated nanopore control sets.

## When to use

Use when per-read scores from methylation-calling tools on 0%/100% methylated control reads need binary classification evaluation (ROC/PR curves, AUC, AUCPR) to rank tool accuracy.

## When NOT to use

- When the goal is per-site (genome-level) methylation frequency agreement rather than per-read binary discrimination — the article benchmarks per-site predictions separately via dispersion vs expected methylation frequency
- When ground truth is not binary (e.g. controlled mixtures at intermediate methylation frequencies, or WGBS comparisons) — ROC/PR labeling with 0/1 methylated control sets does not apply
- When comparing raw score values across tools rather than rankings — score types differ (log-likelihood ratios vs significance values vs log-probability differences) and are not interchangeable

## Inputs

- Per-read score TSV files (columns ID, Chr, Pos, Strand, Score) for Nanopolish, Megalodon, DeepSignal, Guppy, and Tombo from controlled mixture dataset 1 (figshare deposit 15081264)
- Read provenance labels assigning each read to the 0% methylated or 100% methylated control set (~2400 reads per set)

## Outputs

- roc_pr_curves.png (ROC and precision-recall curves for all five tools on shared axes)
- auc_aucpr_metrics.csv (AUC and AUCPR per tool)

## How to apply

Load the per-read score TSV files (columns ID, Chr, Pos, Strand, Score) from the figshare deposit of per-read scores from controlled mixture dataset 1, one file per tool (Nanopolish, Megalodon, DeepSignal, Guppy, Tombo). Assign binary labels per read: 1 (methylated) for reads from the 100% methylated set, 0 (unmethylated) for reads from the 0% methylated set, ~2400 reads per set. Feed each tool's score column as the ranking statistic to scikit-learn's roc_curve and precision_recall_curve — score semantics differ per tool (log-likelihood ratios for Nanopolish/Guppy, significance values for Tombo, log-ratio of methylated/unmethylated probabilities for DeepSignal, log-probability differences for Megalodon), but ROC/PR only require a consistent ranking within each tool. Compute AUC and AUCPR per tool and compare across the five tools. Acceptance requires all five tools to exceed 0.8 on both metrics with Megalodon ranked highest, followed by DeepSignal and Nanopolish.

## Related tools

- **Megalodon** (Per-read methylation scoring tool; expected to achieve the highest AUC and AUCPR among the five benchmarked tools) — https://github.com/nanoporetech/megalodon
- **DeepSignal** (Per-read methylation scoring tool whose log-ratio methylated/unmethylated probability scores are ranked for ROC/PR evaluation) — https://github.com/bioinfomaticsCSU/deepsignal
- **Nanopolish** (Per-read methylation scoring tool; log-likelihood ratio scores used as the ROC/PR ranking statistic)
- **Guppy** (Per-read methylation scoring tool; log-likelihood ratio scores used as the ROC/PR ranking statistic)
- **Tombo** (Per-read methylation scoring tool; significance values used as the ROC/PR ranking statistic)

## Examples

```
from sklearn.metrics import roc_curve, precision_recall_curve, roc_auc_score, average_precision_score; fpr, tpr, _ = roc_curve(labels, scores); auc = roc_auc_score(labels, scores)
```

## Evaluation signals

- All five tools achieve both AUC and AUCPR above 0.8 on the 0%/100% control read labels
- Megalodon ranks highest on both AUC and AUCPR, followed by DeepSignal and Nanopolish
- roc_pr_curves.png and auc_aucpr_metrics.csv are produced
- Label assignment covers ~2400 reads per control set with binary 1/0 values and no missing labels
- Each tool's scores are used only as a within-tool ranking statistic (score scale differences between tools do not affect AUC)

## Limitations

- The M.SssI treatment used to generate the CpG-methylated control has only ~95% efficiency, so labels of 1 may be noisy for a small fraction of reads
- Megalodon provided overall the best performance but relies on GPUs to be time efficient, affecting practical reproducibility of the top-ranking tool
- Tools have a tradeoff between false positives and false negatives: Guppy correctly identified unmethylated sites but failed at fully methylated sites, while Nanopolish and Tombo recovered fully methylated sites but had high false-positive rates at unmethylated sites — so AUC alone does not capture directional error behavior

## Evidence

- [abstract] We found that tools have a tradeoff between false positives and false negatives: "We found that tools have a tradeoff between false positives and false negatives"
- [discussion] Megalodon best performance requires GPU: "Megalodon provided overall the best performance but relies on GPUs to be time efficient."
- [abstract] Control mixture composition grounds binary labels: "a systematic benchmarking of tools for the detection of CpG methylation from Nanopore sequencing using individual reads, control mixtures of methylated and unmethylated reads"
- [methods] Figshare deposit of per-read scores: "you can download them from figshare (https://figshare.com/articles/dataset/Per-read_scores_from_different_nanopore_methylation_tools_in_controlled_mixture_dataset_1/15081264)"
