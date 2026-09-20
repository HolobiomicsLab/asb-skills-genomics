---
name: per-read-methylation-score-evaluation
description: Use when you have per-read methylation scores (TSV with ID/Chr/Pos/Strand/Score
  columns) from one or more nanopore 5mC-calling tools and control reads of known
  methylation status (0%/100% sets), and need to quantify per-read detection accuracy
  via ROC/AUC and precision-recall curves.
when_to_use_negative:
- Your goal is per-site methylation frequency accuracy (dispersion against expected
  mixture frequency, RMSE) rather than single-molecule classification — use the METEORE
  consensus/evaluation workflow instead
- Your reads come from biological samples without control-derived ground truth (e.g.
  WGBS or Cas9-targeted comparisons), where binary per-read labels cannot be assigned
  from 0%/100% mixtures
- Scores come from tools not in the benchmark (e.g. DeepMod was excluded from the
  per-site augmented BED output and consensus comparisons) without re-verifying score
  semantics
tools:
- name: Nanopolish
  role: Per-read 5mC score source; log-likelihood-ratio scores ranked for ROC/PR analysis
- name: Megalodon
  role: Top-performing per-read 5mC detector; log-probability-difference scores ranked
    for ROC/PR analysis
  repo: https://github.com/nanoporetech/megalodon
- name: DeepSignal
  role: Per-read 5mC score source; log-ratio of methylated/unmethylated probabilities
    ranked for ROC/PR analysis; also an input to the METEORE consensus
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Guppy
  role: Per-read modified-basecalling score source (dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg
    config); scores ranked for ROC/PR analysis
- name: Tombo
  role: Per-read modified-base score source; significance-value scores ranked for
    ROC/PR analysis
- name: METEORE
  role: Consensus framework (random forest / regression) that combines Megalodon and
    DeepSignal per-read scores to improve per-site accuracy beyond individual tools
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:0894de7f9351db2183bfe7bab853392cee55135be9af79eb90e20e5e434b6430
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/megalodon
  tools:
  - DeepSignal
  - Guppy
  - METEORE
  - Megalodon
  - Nanopolish
  - Tombo
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# per-read-methylation-score-evaluation

## Summary

Benchmarking of per-read CpG (5mC) methylation detection tools (Nanopolish, Megalodon, DeepSignal, Guppy, Tombo) from nanopore fast5 data by ranking per-read scores against binary ground truth from 0% and 100% methylated control sets, quantified with ROC curves, AUC, precision-recall curves and AUCPR.

## When to use

Use when you have per-read methylation scores (TSV with ID/Chr/Pos/Strand/Score columns) from one or more nanopore 5mC-calling tools and control reads of known methylation status (0%/100% sets), and need to quantify per-read detection accuracy via ROC/AUC and precision-recall curves.

## When NOT to use

- Your goal is per-site methylation frequency accuracy (dispersion against expected mixture frequency, RMSE) rather than single-molecule classification — use the METEORE consensus/evaluation workflow instead
- Your reads come from biological samples without control-derived ground truth (e.g. WGBS or Cas9-targeted comparisons), where binary per-read labels cannot be assigned from 0%/100% mixtures
- Scores come from tools not in the benchmark (e.g. DeepMod was excluded from the per-site augmented BED output and consensus comparisons) without re-verifying score semantics

## Inputs

- Per-read methylation score TSV files (ID, Chr, Pos, Strand, Score) for Nanopolish, Megalodon, DeepSignal, Guppy and Tombo from the figshare deposit (controlled mixture dataset 1)
- Per-read ground-truth labels: reads from the 100% methylated control set (label 1) and the 0% methylated control set (label 0), ~2400 reads per set
- Tool score semantics: log-likelihood ratios (Nanopolish, Guppy), significance values (Tombo), log-probability ratios (DeepSignal), log-probability differences (Megalodon)

## Outputs

- ROC curves per tool plotted on shared axes (roc_pr_curves.png)
- Precision-recall curves per tool on the same figure
- CSV of AUC and AUCPR values per tool (auc_aucpr_metrics.csv)

## How to apply

Load per-read score TSV files (columns ID, Chr, Pos, Strand, Score) from the figshare deposit of per-read scores from controlled mixture dataset 1, and assign binary labels: 1 for reads from the 100% methylated set, 0 for reads from the 0% methylated set (~2400 reads per set). Treat each tool's score as the ranking statistic, respecting its semantics: log-likelihood ratios for Nanopolish/Guppy, significance values for Tombo, log-ratio of methylated/unmethylated probabilities for DeepSignal, and log-probability differences for Megalodon. Compute ROC and precision-recall curves per tool with scikit-learn (roc_curve, precision_recall_curve), then compute AUC and AUCPR per tool. Acceptance criterion from the article: all five tools should exceed 0.8 on both AUC and AUCPR, with Megalodon ranking highest, followed by DeepSignal and Nanopolish; plot all curves on shared axes and export the metrics table. The rationale is that control mixtures of fully methylated and fully unmethylated reads give a definitive single-molecule ground truth, exposing each tool's tradeoff between false positives and false negatives.

## Related tools

- **Nanopolish** (Per-read 5mC score source; log-likelihood-ratio scores ranked for ROC/PR analysis)
- **Megalodon** (Top-performing per-read 5mC detector; log-probability-difference scores ranked for ROC/PR analysis) — https://github.com/nanoporetech/megalodon
- **DeepSignal** (Per-read 5mC score source; log-ratio of methylated/unmethylated probabilities ranked for ROC/PR analysis; also an input to the METEORE consensus) — https://github.com/bioinfomaticsCSU/deepsignal
- **Guppy** (Per-read modified-basecalling score source (dna_r9.4.1_450bps_modbases_dam-dcm-cpg_hac.cfg config); scores ranked for ROC/PR analysis)
- **Tombo** (Per-read modified-base score source; significance-value scores ranked for ROC/PR analysis)
- **METEORE** (Consensus framework (random forest / regression) that combines Megalodon and DeepSignal per-read scores to improve per-site accuracy beyond individual tools) — https://github.com/comprna/METEORE

## Examples

```
from sklearn.metrics import roc_curve, precision_recall_curve; fpr, tpr, _ = roc_curve(labels, scores); prec, rec, _ = precision_recall_curve(labels, scores)
```

## Evaluation signals

- All five per-read tools achieve AUC > 0.8 and AUCPR > 0.8 on the 0%/100% control reads
- Megalodon attains the highest AUC and AUCPR, followed by DeepSignal and then Nanopolish
- Score files parse with exactly the ID, Chr, Pos, Strand, Score columns and each label group contains ~2400 reads
- roc_pr_curves.png and auc_aucpr_metrics.csv are produced with one curve/metric row per tool
- Curves are monotone (ROC) and step-decreasing (PR) and computed with sklearn roc_curve/precision_recall_curve using each tool's score directionality

## Limitations

- Tools exhibit a tradeoff between false positives and false negatives: Guppy correctly identified unmethylated sites but failed at fully methylated sites, while Nanopolish and Tombo recovered fully methylated sites but had high false-positive rates at unmethylated sites
- Megalodon, the best performer, relies on GPUs to be time efficient; combining tools (METEORE) may be impractical for routine analyses due to running times of methods without GPU support
- The M.SssI treatment used to generate the CpG-methylated control only has ~95% efficiency, which can depress agreement with expected methylation frequencies

## Evidence

- [abstract] a systematic benchmarking of tools for the detection of CpG methylation from Nanopore sequencing using individual reads, control mixtures of methylated and unmethylated reads, and bisulfite sequencing: "a systematic benchmarking of tools for the detection of CpG methylation from Nanopore sequencing using individual reads, control mixtures"
- [abstract] We found that tools have a tradeoff between false positives and false negatives: "We found that tools have a tradeoff between false positives and false negatives"
- [discussion] Megalodon provided overall the best performance but relies on GPUs to be time efficient.: "Megalodon provided overall the best performance but relies on GPUs to be time efficient."
- [discussion] Extreme cases were Guppy, which correctly identified unmethylated sites but failed at fully methylated sites: "Extreme cases were Guppy, which correctly identified unmethylated sites but failed at fully methylated sites"
- [methods] you can download them from figshare (https://figshare.com/articles/dataset/Per-read_scores_from_different_nanopore_methylation_tools_in_controlled_mixture_dataset_1/15081264): "you can download them from figshare (https://figshare.com/articles/dataset/Per-read_scores_from_different_nanopore_methylation_tools_in_controlled_mixture_dataset_1/15081264)"
- [readme] it provides new predictive models (random forest and multiple linear regression) that combine the outputs from the tools to produce a consensus prediction with higher accuracy than the individual tools.: "new predictive models (random forest and multiple linear regression) that combine the outputs from the tools to produce a consensus prediction with higher accuracy"
