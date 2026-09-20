---
name: precision-recall-auc-evaluation
description: Use when you need to quantify m6A detection performance from per-read
  modification probabilities with ground-truth labels (e.g., TEST1 synthetic heteropolymers
  or RL/SL synthetic oligos), by computing precision-recall curves and their AUC per
  method or per DRACH motif.
when_to_use_negative:
- Input is already a site-level stoichiometry table rather than per-read modification
  probabilities with ground-truth labels
- The RNA sequences are natural mRNA without molecular-resolution modification ground
  truth (e.g., HEK293 chrX data), where labels are only approximate and PRC AUC cannot
  be computed
- You need site-level aggregate performance with coverage filters (e.g., minimum dRNA-Seq
  coverage of 50 for GLORI comparison) instead of single-read classification performance
edam_operation: http://edamontology.org/operation_2239
tools:
- name: sklearn
  role: Trains logistic regression classifiers (default L-BFGS solver) with 4-fold
    cross-validated 0.75-0.25 splits and provides precision-recall curve computation
- name: mAFiA
  role: Produces per-read and per-site m6A probabilities via test_mAFiA with --min_coverage
    50, --max_num_reads 1000, --mod_prob_thresh 0.5, evaluated by PRC AUC
  repo: https://github.com/dieterich-lab/mAFiA
- name: CHEUI
  role: Competing single-molecule m6A detector run on the same TEST1 reads to obtain
    read-level predictions for the three-method PRC comparison
- name: m6Anet
  role: Competing single-molecule m6A detector run on the same TEST1 reads to obtain
    read-level predictions for the three-method PRC comparison
- name: RODAN
  role: Basecalls RL/SL/TEST1 fast5 reads and supplies the 768-dimensional feature
    vector per nucleotide used to train and cross-validate the logistic regression
    models
  repo: https://github.com/biodlab/RODAN
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:382cce429c4a22f1cd32edf7575f5ebd06429c12c4c34b32236fb6a4cab8646c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dieterich-lab/mAFiA
  tools:
  - CHEUI
  - RODAN
  - m6Anet
  - mAFiA
  - minimap2.22
  - sklearn
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# precision-recall-auc-evaluation

## Summary

Evaluates single-molecule m6A detection performance by thresholding read-level P(m6A) at 0.5 and computing precision-recall curve AUCs against ground-truth modification labels, both for method benchmarking (mAFiA vs CHEUI vs m6Anet on TEST1) and for cross-validation of mAFiA logistic-regression models across the six DRACH motifs.

## When to use

Use when you need to quantify m6A detection performance from per-read modification probabilities with ground-truth labels (e.g., TEST1 synthetic heteropolymers or RL/SL synthetic oligos), by computing precision-recall curves and their AUC per method or per DRACH motif.

## When NOT to use

- Input is already a site-level stoichiometry table rather than per-read modification probabilities with ground-truth labels
- The RNA sequences are natural mRNA without molecular-resolution modification ground truth (e.g., HEK293 chrX data), where labels are only approximate and PRC AUC cannot be computed
- You need site-level aggregate performance with coverage filters (e.g., minimum dRNA-Seq coverage of 50 for GLORI comparison) instead of single-read classification performance

## Inputs

- Per-read m6A probabilities from mAFiA (test_mAFiA output with --mod_prob_thresh 0.5), CHEUI, and m6Anet on the TEST1 synthetic heteropolymer benchmark
- Ground-truth modification labels for TEST1 synthetic heteropolymer reads
- 768-dimensional RODAN feature vectors (from 'convlayers.conv21') per target-motif nucleotide extracted from RL and SL synthetic oligo fast5 reads
- Binary labels: UNM features as y=0, MOD features as y=1, with majority class subsampled to balance the training set

## Outputs

- Precision-recall curves with AUC values per method (mAFiA, CHEUI, m6Anet), rendered as prc_comparison_TEST1.png
- Average precision-recall AUC across the six DRACH motifs on the SL held-out set
- Pickled sklearn logistic regression classifiers and per-motif AUC metrics

## How to apply

For each method, obtain read-level m6A probabilities and threshold them at 0.5 to classify each nucleotide as modified or unmodified, then compute precision-recall curves over the ground-truth labels and report the area under each curve (e.g., mAFiA in blue, CHEUI in orange, m6Anet in green in the TEST1 comparison, Fig. 2c). For model generalization (task RL->SL), train logistic regression on 75% of the random-ligation (RL) features with the sklearn default L-BFGS solver, resampling the 0.75-0.25 train-validation split by 4-fold cross-validation, then validate on held-out splint-ligation (SL) features from unseen sequence contexts. Compute the average precision-recall AUC over the six DRACH motifs on the SL held-out set and save the pickled classifiers together with per-motif AUC metrics. Acceptance for the cross-validation task is judged by whether the recomputed average PRC AUC across all six DRACH motifs on the SL held-out set matches the paper's 4-fold cross-validated 0.75-0.25 split criterion.

## Related tools

- **sklearn** (Trains logistic regression classifiers (default L-BFGS solver) with 4-fold cross-validated 0.75-0.25 splits and provides precision-recall curve computation)
- **mAFiA** (Produces per-read and per-site m6A probabilities via test_mAFiA with --min_coverage 50, --max_num_reads 1000, --mod_prob_thresh 0.5, evaluated by PRC AUC) — https://github.com/dieterich-lab/mAFiA
- **CHEUI** (Competing single-molecule m6A detector run on the same TEST1 reads to obtain read-level predictions for the three-method PRC comparison)
- **m6Anet** (Competing single-molecule m6A detector run on the same TEST1 reads to obtain read-level predictions for the three-method PRC comparison)
- **RODAN** (Basecalls RL/SL/TEST1 fast5 reads and supplies the 768-dimensional feature vector per nucleotide used to train and cross-validate the logistic regression models) — https://github.com/biodlab/RODAN

## Examples

```
test_mAFiA --min_coverage 50 --max_num_reads 1000 --mod_prob_thresh 0.5 ; from sklearn.metrics import precision_recall_curve, average_precision_score
```

## Evaluation signals

- All three PRC curves (mAFiA, CHEUI, m6Anet) with their AUC values are reported and prc_comparison_TEST1.png exists, with mAFiA plotted in blue alongside CHEUI (orange) and m6Anet (green) as in Fig. 2c
- Precision-recall curves are produced for all six DRACH motifs on the SL held-out set, showing the RL-trained model performs robustly on unseen sequence contexts
- The train-validation procedure matches the paper's model-selection criterion: 4-fold cross-validated 0.75-0.25 split of RL features
- Read-level classification uses the exact threshold P(m6A) >= 0.5, and recomputed average PRC AUC is consistent across resampled folds

## Limitations

- The evaluation relies on synthetic samples (TEST1, RL/SL oligos) with precisely controlled modification labels; natural mRNA lacks read-level ground-truth labels, so PRC AUC is not computable there
- The six covered DRACH motifs collectively account for almost 80% of consensus m6A sites in human mRNA, so the motif coverage of the evaluation is not exhaustive
- Performance depends on the majority-class subsampling used to balance the training set, which introduces resampling variability handled only via 4-fold cross-validation

## Evidence

- [results] Read-level P_n(m6A) is thresholded at 0.5 to classify each nucleotide on a read into modified or unmodified status: [evidence span withheld — anchor status: not_found]
- [results] The 0.75-0.25 train-validation split is resampled by 4-fold cross-validation: "The 0.75-0.25 train-validation split is resampled by 4-fold cross-validation"
- [results] Optimization is performed with the python module sklearn, using the default L-BFGS solver: "Optimization is performed with the python module sklearn, using the default L-BFGS solver"
- [results] Features from unmodified (UNM) samples carry the label y=0, while those from modified (MOD) carry the label y=1: [evidence span withheld — anchor status: not_found]
- [other] --mod_prob_thresh 0.5: [evidence span withheld — anchor status: not_found]
