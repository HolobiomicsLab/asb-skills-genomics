---
name: convolutional-neural-network-ablation-study
description: Use when you must justify a multi-column/multi-scale CNN architecture
  over simpler single-scale variants by running controlled ablation comparisons on
  held-out bacteria (LOOCV) and an independent test set.
when_to_use_negative:
- The goal is methylation motif discovery or de novo motif enrichment rather than
  benchmarking an existing CNN architecture
- Only a pretrained model is needed for prediction — use best_model/Nanoident.pt with
  postprocess.py instead of retraining ablation variants
- Input data lacks the five Nanoident features (nanopore raw-signal statistics plus
  basecall features) required to train the convolutional variants
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: Nanoident
  role: Multi-column convolutional network framework whose fused multi-scale model
    is benchmarked against single-scale (L, M, S) ablation variants
  repo: https://github.com/cz-csu/Nanoident
- name: Python
  role: Runtime for training and evaluating the LOOCV and single-scale ablation models
    (train_loocv_model.py, train_loocv_test.py, train_test.py)
- name: conda
  role: Environment management via Nanoident/env/freeze.yml to reproduce the training
    stack
- name: nanodisco
  role: Provides the R preprocessing/motif-enrichment environment that produces the
    training data consumed by the ablation study
  repo: https://github.com/fanglab/nanodisco
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:5ba1ab13a8adea90fbf7ca2ccc58acc6a520c555252242de498e6fba1163c1ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cz-csu/Nanoident
  tools:
  - Nanoident
  - Python
  - conda
  - nanodisco
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# convolutional-neural-network-ablation-study

## Summary

Trains single-scale (large L, medium M, small S single-column) ablation variants of the Nanoident multi-column CNN and compares them against the fused multi-scale network on LOOCV and an independent dataset to quantify the benefit of multi-scale receptive-field fusion for bacterial methylation typing and fine mapping.

## When to use

Use when you must justify a multi-column/multi-scale CNN architecture over simpler single-scale variants by running controlled ablation comparisons on held-out bacteria (LOOCV) and an independent test set.

## When NOT to use

- The goal is methylation motif discovery or de novo motif enrichment rather than benchmarking an existing CNN architecture
- Only a pretrained model is needed for prediction — use best_model/Nanoident.pt with postprocess.py instead of retraining ablation variants
- Input data lacks the five Nanoident features (nanopore raw-signal statistics plus basecall features) required to train the convolutional variants

## Inputs

- Preprocessed pkl feature files for motif sequences (converted from R environment via data_from_R_to_python.ipynb)
- LOOCV dataset of 7 bacteria samples with 46 methylation motifs (7 4mC, 11 5mC, 28 6mA)
- Independent test dataset of 2 species with 12 methylation motifs (6 4mC, 1 5mC, 5 6mA)

## Outputs

- LOOCV accuracies for the fused multi-scale network and each single-scale variant (L, M, S)
- Independent-dataset accuracies for all variants
- Accuracy comparison table (e.g. loocv_scale_comparison.csv)

## How to apply

Set up the Python/conda environment from Nanoident/env/freeze.yml and load the pkl feature files (five features per motif sequence, converted from R via data_from_R_to_python.ipynb). Train each single-scale single-column variant (L, M, S) with train_loocv_model.py, leaving one of the 7 bacteria out per fold over the 46 methylation motifs, and obtain accuracies with train_loocv_test.py. Evaluate every variant on the independent dataset (12 motifs from two species) using the test portion of train_test.py. Compare the fused multi-scale network against the single-scale variants and record accuracies: LOOCV fused 83.68% vs S 83.55%, M 82.17%, L 78.56%; independent fused 73.58% vs S 72.5%, M 71.67%, L 70.58%. Save the comparison as a table (e.g. loocv_scale_comparison.csv) and confirm the fused network outranks every single-scale variant on both evaluation sets.

## Related tools

- **Nanoident** (Multi-column convolutional network framework whose fused multi-scale model is benchmarked against single-scale (L, M, S) ablation variants) — https://github.com/cz-csu/Nanoident
- **Python** (Runtime for training and evaluating the LOOCV and single-scale ablation models (train_loocv_model.py, train_loocv_test.py, train_test.py))
- **conda** (Environment management via Nanoident/env/freeze.yml to reproduce the training stack)
- **nanodisco** (Provides the R preprocessing/motif-enrichment environment that produces the training data consumed by the ablation study) — https://github.com/fanglab/nanodisco

## Examples

```
conda env create -f freeze.yml; python ./model/train_loocv_model.py; python ./model/train_loocv_test.py; python ./model/train_test.py
```

## Evaluation signals

- Fused multi-scale LOOCV accuracy (83.68%) must exceed the best single-scale variant S (83.55%)
- Same ordering must hold on the independent dataset: fused 73.58% > S 72.5% > M 71.67% > L 70.58%
- LOOCV accuracies are computed over all 46 methylation motifs with each of the 7 bacteria left out once
- Independent-dataset evaluation must cover all 12 methylation motifs from the two test species
- Single-scale variant ranking is consistent across both sets: S > M > L

## Limitations

- Small datasets: LOOCV uses only 7 bacteria / 46 motifs and the independent set only 12 motifs, so accuracy differences between fused (83.68%) and S (83.55%) are narrow
- Preprocessing must be done in chunks due to memory constraints before training can proceed
- Original nanodisco handles only mean features; the update.sh patch is required to expose the four additional features (std, time, proportion of reference bases, average quality score) the network uses

## Evidence

- [supplementary] In LOOCV, the multi-scale fusion network achieved higher accuracy (83.68%) than the single-scale networks (S: 83.55%, M: 82.17%, L: 78.56%): [evidence span withheld — anchor status: not_found]
- [methods] Train the single-scale ablation models (large L, medium M, small S single-column convolutional networks) with train_loocv_model.py: [evidence span withheld — anchor status: not_found]
- [readme] Use Nanoident/env/freeze.yml to generate the environment directly under conda.: "Use Nanoident/env/freeze.yml to generate the environment directly under conda."
- [intro] A multi-column convolutional neural network model for typing (the identification of methylation types): "A multi-column convolutional neural network model for typing (the identification of methylation types)"
