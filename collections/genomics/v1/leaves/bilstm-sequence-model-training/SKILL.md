---
name: bilstm-sequence-model-training
description: Use when benchmarking alternative sequence architectures (Bi-LSTM, Transformer)
  against a CNN-based methylation typing model on the same pkl feature matrices and
  LOOCV split.
when_to_use_negative:
- The goal is to use the pretrained convolutional Nanoident model rather than benchmark
  alternative architectures (use best_model/Nanoident.pt with postprocess.py instead).
- Input features have not yet been converted from the R environment to pkl format.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: train_loocv_model.py
  role: Trains the two-layer Bi-LSTM variant for leave-one-out cross-validation
  repo: https://github.com/cz-csu/Nanoident
- name: train_loocv_test.py
  role: Runs the LOOCV evaluation across the 7 training bacteria with 46 methylation
    motifs
  repo: https://github.com/cz-csu/Nanoident
- name: train_test.py
  role: Evaluates the trained Bi-LSTM variant on the independent 12-motif test dataset
  repo: https://github.com/cz-csu/Nanoident
- name: Nanoident
  role: Convolutional reference model against which the Bi-LSTM baseline accuracies
    and parameter counts are compared
  repo: https://github.com/cz-csu/Nanoident
- name: conda
  role: Builds the Python training environment from Nanoident/env/freeze.yml
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:734e2b75c2233a3836c8c33698b1b4bd14bd8499609970033e0091a9b9925418
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cz-csu/Nanoident
  tools:
  - Nanoident
  - conda
  - train_loocv_model.py
  - train_loocv_test.py
  - train_test.py
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# bilstm-sequence-model-training

## Summary

Train and evaluate a two-layer Bi-LSTM baseline against the convolutional Nanoident model for bacterial methylation typing under the same LOOCV protocol on nanopore-derived methylation motif features.

## When to use

Use when benchmarking alternative sequence architectures (Bi-LSTM, Transformer) against a CNN-based methylation typing model on the same pkl feature matrices and LOOCV split.

## When NOT to use

- The goal is to use the pretrained convolutional Nanoident model rather than benchmark alternative architectures (use best_model/Nanoident.pt with postprocess.py instead).
- Input features have not yet been converted from the R environment to pkl format.

## Inputs

- pkl-formatted train/validation feature matrices converted from the R environment via data_from_R_to_python.ipynb
- pkl-formatted test feature matrix for the independent 12-motif dataset
- Python environment created from Nanoident/env/freeze.yml via conda

## Outputs

- Trained two-layer Bi-LSTM LOOCV model
- LOOCV accuracy, independent-set accuracy, and trainable parameter count written to loocv_accuracy_results.csv

## How to apply

Convert the R-preprocessed train/validation and test feature matrices to pkl with data_from_R_to_python.ipynb, then train the two-layer Bi-LSTM variant with python ./model/train_loocv_model.py under its Bi-LSTM configuration. Run the leave-one-out cross-validation evaluation across the 7 training bacteria (46 methylation motifs: 7 4mC, 11 5mC, 28 6mA) with python ./model/train_loocv_test.py. Evaluate on the independent 12-motif test dataset using python ./model/train_test.py. Record LOOCV accuracy, independent-set accuracy, and total trainable parameter count for each variant in loocv_accuracy_results.csv so architectures are compared under the identical protocol as nanodisco/Nanoident.

## Related tools

- **train_loocv_model.py** (Trains the two-layer Bi-LSTM variant for leave-one-out cross-validation) — https://github.com/cz-csu/Nanoident
- **train_loocv_test.py** (Runs the LOOCV evaluation across the 7 training bacteria with 46 methylation motifs) — https://github.com/cz-csu/Nanoident
- **train_test.py** (Evaluates the trained Bi-LSTM variant on the independent 12-motif test dataset) — https://github.com/cz-csu/Nanoident
- **Nanoident** (Convolutional reference model against which the Bi-LSTM baseline accuracies and parameter counts are compared) — https://github.com/cz-csu/Nanoident
- **conda** (Builds the Python training environment from Nanoident/env/freeze.yml)

## Examples

```
conda env create -f freeze.yml; python ./model/train_loocv_model.py; python ./model/train_loocv_test.py; python ./model/train_test.py
```

## Evaluation signals

- Reproduced LOOCV accuracy for the Bi-LSTM matches the reported 71.51%
- Independent-set accuracy for the Bi-LSTM matches the reported 70.25%
- Total trainable parameter count matches the reported Bi-LSTM value of 36,693
- Bi-LSTM LOOCV accuracy is lower than nanodisco/Nanoident's 83.68%
- All metrics are recorded in loocv_accuracy_results.csv for direct comparison with the Transformer (67.61% LOOCV) and Nanoident (14,319 parameters)

## Limitations

- The Bi-LSTM baseline underperforms the convolutional Nanoident model under LOOCV (71.51% vs 83.68%), so it is a comparison baseline rather than a recommended production model.
- Results depend on the identical LOOCV protocol and the same pkl feature matrices; changing splits or features invalidates the reported 71.51% / 70.25% reference values.

## Evidence

- [results] In the LOOCV evaluation, the two-layer Bi-LSTM and one-layer Transformer baselines achieved average accuracies of only 71.51% and 67.61%: [evidence span withheld — anchor status: not_found]
- [methods] convert it to a pkl file that can be used in a python environment: "we need to convert it to a pkl file that can be used in a python environment using data_from_R_to_python.ipynb"
- [supplementary] There are 46 different motifs in these seven bacteria: "There are 46 different motifs in these seven bacteria, including 7 4mC motifs, 11 5mC motifs, and 28 6mA motifs."
- [readme] Use Nanoident/env/freeze.yml to generate the environment directly under conda: "Use Nanoident/env/freeze.yml to generate the environment directly under conda."
- [methods] python ./model/train_loocv_model.py: "python ./model/train_loocv_model.py"
