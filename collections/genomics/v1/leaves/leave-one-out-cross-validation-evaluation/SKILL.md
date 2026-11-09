---
name: leave-one-out-cross-validation-evaluation
description: 'Use when a trained methylation typing model must be assessed for generalization
  to unseen motifs: hold out each of the 46 motifs (from 7 bacterial species) in turn,
  retrain on the remaining 45, and aggregate fold-level accuracies.'
when_to_use_negative:
- The goal is performance on the independent test set of 12 motifs from two species
  (use train_test.py instead)
- Only a pretrained model is available and no retraining per fold is possible — LOOCV
  requires training 46 separate fold models
- The dataset has far more samples/motifs than 46, making 46 retraining runs computationally
  prohibitive without stratified k-fold
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: Nanoident
  role: Multi-column/multi-scale CNN model being trained per LOOCV fold on 45 motifs
    and tested on the held-out motif
  repo: https://github.com/cz-csu/Nanoident
- name: train_loocv_model.py
  role: Trains each fold model on the 45 non-held-out motifs
  repo: https://github.com/cz-csu/Nanoident
- name: train_loocv_test.py
  role: Evaluates each fold model on its held-out motif to produce fold-level LOOCV
    accuracy
  repo: https://github.com/cz-csu/Nanoident
- name: nanodisco
  role: Baseline methylation typing method retrained under the same LOOCV protocol
    in the R/Singularity environment for comparison
  repo: https://github.com/fanglab/nanodisco
- name: conda
  role: Provides the Python environment for model training via Nanoident/env/freeze.yml
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:a299abacf0f8ae1e96a177490c1fe52d238914fbab380b4cdb60abaae4fc733a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cz-csu/Nanoident
  tools:
  - Nanoident
  - R
  - Rscript
  - conda
  - nanodisco
  - train_loocv_model.py
  - train_loocv_test.py
  - train_test.py
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# leave-one-out-cross-validation-evaluation

## Summary

Evaluates the Nanoident multi-scale CNN for bacterial methylation typing via leave-one-out cross-validation (LOOCV) over 46 methylation motifs from 7 bacteria, training on 45 motifs and testing on the held-out motif per fold, and aggregating fold accuracies for comparison against nanodisco and alternative architectures.

## When to use

Use when a trained methylation typing model must be assessed for generalization to unseen motifs: hold out each of the 46 motifs (from 7 bacterial species) in turn, retrain on the remaining 45, and aggregate fold-level accuracies.

## When NOT to use

- The goal is performance on the independent test set of 12 motifs from two species (use train_test.py instead)
- Only a pretrained model is available and no retraining per fold is possible — LOOCV requires training 46 separate fold models
- The dataset has far more samples/motifs than 46, making 46 retraining runs computationally prohibitive without stratified k-fold

## Inputs

- pkl-format train/validation feature matrices converted from R via data_from_R_to_python.ipynb
- Training dataset: 7 bacterial samples with 46 methylation motifs (7 4mC, 11 5mC, 28 6mA) from BioProject PRJNA559199
- Optional architecture configuration (multi-scale fused, single-scale L/M/S, Bi-LSTM, Transformer)

## Outputs

- Fold-level LOOCV typing accuracies for each held-out motif
- Aggregated per-motif and average accuracy tables (e.g. loocv_accuracy_results.csv, loocv_scale_comparison.csv)
- Trained per-fold models from train_loocv_model.py

## How to apply

Load the pkl-format train/validation feature matrices previously converted from the R environment with data_from_R_to_python.ipynb, covering the 7 training bacteria and their 46 methylation motifs (7 4mC, 11 5mC, 28 6mA). For each LOOCV fold, hold out one motif and train the model with train_loocv_model.py (Python/conda environment), then evaluate the fold model on its held-out motif with train_loocv_test.py to get fold-level typing accuracy. Retrain the nanodisco baseline (in the R/Singularity environment) under the same protocol when a head-to-head comparison is needed, and optionally repeat the loop for architecture variants (single-scale L/M/S networks, two-layer Bi-LSTM, one-layer Transformer). Aggregate fold accuracies into per-motif and average tables (e.g. loocv_accuracy_results.csv or loocv_scale_comparison.csv). Validation: the fused multi-scale Nanoident should reach ~83.68% average LOOCV accuracy, exceeding retrained nanodisco (73.93%), Bi-LSTM (71.51%), Transformer (67.61%), and every single-scale variant (S 83.55%, M 82.17%, L 78.56%).

## Related tools

- **Nanoident** (Multi-column/multi-scale CNN model being trained per LOOCV fold on 45 motifs and tested on the held-out motif) — https://github.com/cz-csu/Nanoident
- **train_loocv_model.py** (Trains each fold model on the 45 non-held-out motifs) — https://github.com/cz-csu/Nanoident
- **train_loocv_test.py** (Evaluates each fold model on its held-out motif to produce fold-level LOOCV accuracy) — https://github.com/cz-csu/Nanoident
- **nanodisco** (Baseline methylation typing method retrained under the same LOOCV protocol in the R/Singularity environment for comparison) — https://github.com/fanglab/nanodisco
- **conda** (Provides the Python environment for model training via Nanoident/env/freeze.yml)

## Examples

```
python ./model/train_loocv_model.py && python ./model/train_loocv_test.py
```

## Evaluation signals

- Nanoident fused multi-scale LOOCV average accuracy should reproduce ~83.68% across the 46 motifs
- Nanoident's LOOCV average should exceed retrained nanodisco's 73.93% by roughly 10 percentage points
- Architecture baselines should match reported values: Bi-LSTM 71.51%, Transformer 67.61% (LOOCV)
- Single-scale ablation ordering should hold: S (83.55%) > M (82.17%) > L (78.56%), all below the fused 83.68%
- All 46 folds must complete with one motif held out per fold and per-fold accuracies aggregated without missing entries

## Limitations

- LOOCV requires 46 separate training runs, so memory and runtime are dominated by repeated training; the authors note preprocessing must be chunked 'because of memory constraints'
- Fold-level accuracy varies by motif; the 83.68% is an average, so individual held-out motifs may score substantially lower
- Comparison to nanodisco requires retraining the baseline in the R/Singularity environment under the same protocol rather than reusing published numbers

## Evidence

- [abstract] LOOCV protocol over 46 motifs: "The Leave-One-Out Cross-Validation (LOOCV) on the dataset containing 7 bacteria samples with 46 methylation motifs shows that Nanoident achieves ∼10% improvement in accuracy than the previous method."
- [readme] LOOCV evaluation commands: "python ./model/train_loocv_model.py"
- [readme] LOOCV fold testing command: "python ./model/train_loocv_test.py"
- [supplementary] Motif composition of training set: "There are 46 different motifs in these seven bacteria, including 7 4mC motifs, 11 5mC motifs, and 28 6mA motifs."
- [methods] R-to-Python data conversion: "we need to convert it to a pkl file that can be used in a python environment using data_from_R_to_python.ipynb"
