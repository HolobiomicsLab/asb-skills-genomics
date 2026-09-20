---
name: logistic-regression-model-training
description: Use when read-level m6A prediction must be learned from labeled synthetic
  RNA features (UNM y=0, MOD y=1) and validated on held-out data such as SL samples
  from unseen sequence contexts.
when_to_use_negative:
- Input features are already site-level aggregated stoichiometry estimates rather
  than per-read 768-d RODAN features
- The goal is detecting modifications outside the DRACH motifs covered by the training
  data (mAFiA classifiers are motif-specific)
- Only natural mRNA with approximate labels is available and no molecularly labeled
  synthetic controls exist to train on
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: mAFiA
  role: Extracts the 768-d feature vector per nucleotide and hosts the logistic regression
    training/pickling pipeline; models at github.com/dieterich-lab/mAFiA and zenodo.org/record/8321727
  repo: https://github.com/dieterich-lab/mAFiA
- name: RODAN
  role: Fully-convolutional basecaller whose last convolution layer 'convlayers.conv21'
    supplies the hidden features used as classifier input
  repo: https://github.com/biodlab/RODAN
- name: sklearn
  role: Provides the logistic regression optimizer (default L-BFGS solver) and cross-validation
    machinery
- name: minimap2.22
  role: Aligns basecalled reads to reference oligo sequences so target-motif nucleotides
    can be located before feature extraction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:0b86a8b0a619ab803fe7f1845f7c18f5c1418a29142ff7e9a3e33289742fdfff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dieterich-lab/mAFiA
  tools:
  - RODAN
  - mAFiA
  - minimap2.22
  - sklearn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# logistic-regression-model-training

## Summary

Train per-motif logistic regression classifiers on 768-dimensional RODAN convolutional features to predict m6A status of DRACH-motif nucleotides in direct RNA sequencing reads, using balanced UNM/MOD labels from synthetic oligo datasets.

## When to use

Use when read-level m6A prediction must be learned from labeled synthetic RNA features (UNM y=0, MOD y=1) and validated on held-out data such as SL samples from unseen sequence contexts.

## When NOT to use

- Input features are already site-level aggregated stoichiometry estimates rather than per-read 768-d RODAN features
- The goal is detecting modifications outside the DRACH motifs covered by the training data (mAFiA classifiers are motif-specific)
- Only natural mRNA with approximate labels is available and no molecularly labeled synthetic controls exist to train on

## Inputs

- RODAN basecalled fast5 reads from RL and SL synthetic oligo datasets
- 768-dimensional feature vectors per target-motif nucleotide extracted from RODAN layer 'convlayers.conv21'
- Binary labels: UNM oligo features y=0, MOD oligo features y=1

## Outputs

- Pickled sklearn logistic regression classifiers per DRACH motif
- Per-motif average precision-recall AUC metrics on the held-out SL set

## How to apply

Basecall RL and SL fast5 reads with the RODAN-based backbone and extract the 768-dimensional feature vector per target-motif nucleotide from the last convolution layer 'convlayers.conv21'. Label features from unmodified (UNM) oligos as y=0 and modified (MOD) oligos as y=1, subsampling the majority class to balance the training set. Train a logistic regression with sklearn's default L-BFGS solver on 75% of the RL features, resampling the 0.75-0.25 train-validation split by 4-fold cross-validation. Validate the trained classifiers on held-out SL dataset features from unseen sequence contexts. Save the pickled classifiers together with per-motif average precision-recall AUC metrics over the six DRACH motifs.

## Related tools

- **mAFiA** (Extracts the 768-d feature vector per nucleotide and hosts the logistic regression training/pickling pipeline; models at github.com/dieterich-lab/mAFiA and zenodo.org/record/8321727) — https://github.com/dieterich-lab/mAFiA
- **RODAN** (Fully-convolutional basecaller whose last convolution layer 'convlayers.conv21' supplies the hidden features used as classifier input) — https://github.com/biodlab/RODAN
- **sklearn** (Provides the logistic regression optimizer (default L-BFGS solver) and cross-validation machinery)
- **minimap2.22** (Aligns basecalled reads to reference oligo sequences so target-motif nucleotides can be located before feature extraction)

## Examples

```
from sklearn.linear_model import LogisticRegression; clf = LogisticRegression().fit(X_rl_train, y_rl_train)
```

## Evaluation signals

- Recomputed average precision-recall AUC across all six DRACH motifs on the SL held-out set matches the paper's model-selection criterion
- Cross-validation follows the paper's exact scheme: 4-fold resampling of the 0.75-0.25 train-validation split
- Training set is class-balanced via majority-class subsampling (no dominant UNM or MOD class in the fit)
- Feature vectors are 768-dimensional, taken from RODAN layer 'convlayers.conv21', only for predicted nucleotide A positions in target motifs
- Pickled classifiers are saved per motif and reloadable via sklearn for downstream read-level prediction

## Limitations

- Class imbalance between UNM and MOD features requires majority-class subsampling before fitting
- Generalization was demonstrated on the six most common DRACH motifs (~80% of human consensus m6A sites); other motifs are not covered
- mAFiA is now superseded by Ψ-co-mAFiA (Jan 2025), which extends coverage to all 18 m6A DRACH motifs

## Evidence

- [results] mAFiA extracts a 768-dimensional feature vector: [evidence span withheld — anchor status: not_found]
- [results] Feature labels and balancing: [evidence span withheld — anchor status: not_found]
- [results] Cross-validation scheme: "The 0.75-0.25 train-validation split is resampled by 4-fold cross-validation"
- [results] sklearn optimizer: "Optimization is performed with the python module sklearn, using the default L-BFGS solver19"
- [readme] Feature source layer from README: "backbone.torch: [RODAN](https://github.com/biodlab/RODAN)-based neural network for basecalling and feature extraction"
