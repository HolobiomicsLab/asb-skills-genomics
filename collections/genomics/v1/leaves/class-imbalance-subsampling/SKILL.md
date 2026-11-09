---
name: class-imbalance-subsampling
description: Use when training a binary classifier (e.g. mAFiA m6A logistic regression)
  from nanopore read features where one class (UNM vs MOD) substantially outnumbers
  the other and unbiased training would bias P(m6A) predictions.
when_to_use_negative:
- Input is already a balanced feature table or a fitted model — no subsampling step
  is needed
- Goal is site-level stoichiometry quantification on natural mRNA (pileup with min_coverage
  50), not classifier training
- Training data are natural mRNA reads with approximate labels rather than the synthetic
  RL oligos with precisely controlled per-nucleotide modification status
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: mAFiA
  role: Hosts the logistic-regression training pipeline into which the balanced (subsampled)
    feature-label sets are fed; classifiers are pickled per DRACH motif
  repo: https://github.com/dieterich-lab/mAFiA
- name: RODAN
  role: Supplies the 768-dimensional feature vectors per target-motif nucleotide that
    constitute the subsampled training instances
  repo: https://github.com/biodlab/RODAN
- name: sklearn
  role: Provides the LogisticRegression classifier with the default L-BFGS solver
    trained on the balanced set and the cross-validation machinery
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:5b05686b06028093eb4216096b9f1bb47ab3275abf1deecdabefa3255b68e695
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dieterich-lab/mAFiA
  tools:
  - RODAN
  - mAFiA
  - minimap2.22
  - sklearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# class-imbalance-subsampling

## Summary

Balances training data for mAFiA logistic-regression m6A classifiers by subsampling the majority class of RODAN-extracted 768-dimensional features, labeling UNM oligo features as y=0 and MOD features as y=1 so the RL-trained model generalizes to held-out SL samples.

## When to use

Use when training a binary classifier (e.g. mAFiA m6A logistic regression) from nanopore read features where one class (UNM vs MOD) substantially outnumbers the other and unbiased training would bias P(m6A) predictions.

## When NOT to use

- Input is already a balanced feature table or a fitted model — no subsampling step is needed
- Goal is site-level stoichiometry quantification on natural mRNA (pileup with min_coverage 50), not classifier training
- Training data are natural mRNA reads with approximate labels rather than the synthetic RL oligos with precisely controlled per-nucleotide modification status

## Inputs

- 768-dimensional per-nucleotide feature vectors extracted from RODAN layer 'convlayers.conv21' for RL and SL fast5 reads
- Oligo modification labels (UNM vs MOD) for the 15 synthetic training sequences
- DRACH motif annotation of target-motif adenosines

## Outputs

- Pickled per-motif logistic regression classifiers
- Balanced (subsampled) training feature-label set
- Per-DRACH-motif average precision-recall AUC metrics on the SL held-out set

## How to apply

Extract the 768-dimensional per-nucleotide feature vectors from the last convolution layer ('convlayers.conv21') of the RODAN-based backbone for target-motif nucleotides in the random-ligation (RL) synthetic oligo reads. Assign label y=0 to features from unmodified (UNM) oligos and y=1 to features from modified (MOD) oligos, then subsample the majority class to balance the training set. Fit a logistic regression with sklearn's default L-BFGS solver on 75% of the RL features, resampling the 0.75-0.25 train-validation split via 4-fold cross-validation. Pickle the resulting classifiers and evaluate them on held-out splint-ligation (SL) features from unseen sequence contexts, computing average precision-recall AUC per DRACH motif. The rationale is that balanced classes prevent the logistic regression from defaulting to the unmodified prediction, which would destroy sensitivity on the six DRACH motifs.

## Related tools

- **mAFiA** (Hosts the logistic-regression training pipeline into which the balanced (subsampled) feature-label sets are fed; classifiers are pickled per DRACH motif) — https://github.com/dieterich-lab/mAFiA
- **RODAN** (Supplies the 768-dimensional feature vectors per target-motif nucleotide that constitute the subsampled training instances) — https://github.com/biodlab/RODAN
- **sklearn** (Provides the LogisticRegression classifier with the default L-BFGS solver trained on the balanced set and the cross-validation machinery)

## Examples

```
from sklearn.linear_model import LogisticRegression; clf = LogisticRegression().fit(X_train, y_train)
```

## Evaluation signals

- After subsampling, the UNM (y=0) and MOD (y=1) training feature counts are approximately equal per motif
- The 0.75-0.25 train-validation split is resampled by 4-fold cross-validation before model selection
- Average precision-recall AUC is recomputed for all six DRACH motifs on the SL held-out set and shows robust performance on unseen sequence contexts
- Saved artifacts include the pickled logistic regression classifiers together with the per-motif AUC metrics

## Limitations

- Subsampling discards majority-class instances, reducing effective UNM training depth
- Generalization was demonstrated only on the six most common DRACH motifs (~80% of human consensus m6A sites) using synthetic RL/SL oligos, not the full DRACH space
- Labels are precise only because the synthetic oligos carry controlled modification status; approximate labels from natural mRNA would undermine the y=0/y=1 assignment

## Evidence

- [results] UNM y=0 / MOD y=1 labeling: [evidence span withheld — anchor status: not_found]
- [results] Cross-validation split: "The 0.75-0.25 train-validation split is resampled by 4-fold cross-validation"
- [results] sklearn solver: "Optimization is performed with the python module sklearn, using the default L-BFGS solver"
- [readme] Feature source: "backbone.torch: [RODAN](https://github.com/biodlab/RODAN)-based neural network for basecalling and feature extraction"
