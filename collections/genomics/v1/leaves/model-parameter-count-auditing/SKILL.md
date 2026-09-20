---
name: model-parameter-count-auditing
description: Use when comparing neural architectures for methylation motif typing
  under an identical protocol and you must report accuracy together with total trainable
  parameters to verify a fair comparison.
when_to_use_negative:
- The goal is de novo methylation motif discovery or enrichment from nanopore signals
  rather than comparing trained classifiers
- Only a single architecture is being trained and no cross-architecture fairness claim
  is needed
- Input is raw nanopore signal or modBAM data that still requires the nanodisco/bam-readcount
  preprocessing pipeline
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: Nanoident
  role: Reference convolutional model whose 14,319 trainable parameters and 83.68%
    LOOCV accuracy serve as the audit baseline
  repo: https://github.com/cz-csu/Nanoident
- name: train_loocv_model.py
  role: Trains each architecture variant (Bi-LSTM, Transformer) under the LOOCV protocol
  repo: https://github.com/cz-csu/Nanoident
- name: train_loocv_test.py
  role: Runs the leave-one-out cross-validation evaluation across the 7 training bacteria
    (46 methylation motifs)
  repo: https://github.com/cz-csu/Nanoident
- name: train_test.py
  role: Evaluates trained variants on the independent 12-motif test dataset and includes
    a section for test results
  repo: https://github.com/cz-csu/Nanoident
- name: data_from_R_to_python.ipynb
  role: Converts R-preprocessed feature data into pkl files consumable by the Python
    model-training scripts
  repo: https://github.com/cz-csu/Nanoident
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:d5358eca2f587f44e7477467042d3c203832ecbba6c8cc034e92a1541fab9388
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cz-csu/Nanoident
  tools:
  - Nanoident
  - conda
  - data_from_R_to_python.ipynb
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
yes_rate: 0.55
n_claims: 20
---

# model-parameter-count-auditing

## Summary

Audits the trainable parameter counts and LOOCV/independent-set accuracies of alternative architectures (Nanoident CNN, two-layer Bi-LSTM, one-layer Transformer) used for bacterial methylation typing, to separate accuracy gains from model size differences.

## When to use

Use when comparing neural architectures for methylation motif typing under an identical protocol and you must report accuracy together with total trainable parameters to verify a fair comparison.

## When NOT to use

- The goal is de novo methylation motif discovery or enrichment from nanopore signals rather than comparing trained classifiers
- Only a single architecture is being trained and no cross-architecture fairness claim is needed
- Input is raw nanopore signal or modBAM data that still requires the nanodisco/bam-readcount preprocessing pipeline

## Inputs

- pkl-formatted train/validation feature matrices converted from R via data_from_R_to_python.ipynb
- pkl-formatted independent test feature matrix (12 methylation motifs from two test bacteria)
- Model variant configurations for Nanoident, two-layer Bi-LSTM, and one-layer Transformer

## Outputs

- loocv_accuracy_results.csv with LOOCV accuracy, independent-set accuracy, and total trainable parameter count per variant
- Trained model checkpoints per variant (e.g., best_model/Nanoident.pt style outputs)

## How to apply

Load the pkl-formatted train/validation and test feature matrices converted from the R environment with data_from_R_to_python.ipynb. Train each candidate variant — the Nanoident multi-column CNN, a two-layer Bi-LSTM, and a one-layer Transformer — with train_loocv_model.py under the same LOOCV setup spanning the 7 training bacteria (46 methylation motifs), then run the LOOCV evaluation with train_loocv_test.py and independent 12-motif test-set evaluation with train_test.py. For every variant, record LOOCV accuracy, independent-set accuracy, and total trainable parameter count, writing all metrics to loocv_accuracy_results.csv. The rationale is that a baseline with far more parameters (e.g., Bi-LSTM 36,693 vs Nanoident 14,319) that still underperforms (71.51% vs 83.68% LOOCV) demonstrates the convolutional multi-scale design, not capacity, drives performance.

## Related tools

- **Nanoident** (Reference convolutional model whose 14,319 trainable parameters and 83.68% LOOCV accuracy serve as the audit baseline) — https://github.com/cz-csu/Nanoident
- **train_loocv_model.py** (Trains each architecture variant (Bi-LSTM, Transformer) under the LOOCV protocol) — https://github.com/cz-csu/Nanoident
- **train_loocv_test.py** (Runs the leave-one-out cross-validation evaluation across the 7 training bacteria (46 methylation motifs)) — https://github.com/cz-csu/Nanoident
- **train_test.py** (Evaluates trained variants on the independent 12-motif test dataset and includes a section for test results) — https://github.com/cz-csu/Nanoident
- **data_from_R_to_python.ipynb** (Converts R-preprocessed feature data into pkl files consumable by the Python model-training scripts) — https://github.com/cz-csu/Nanoident

## Examples

```
python ./model/train_loocv_model.py  # per variant; then python ./model/train_loocv_test.py && python ./model/train_test.py; record metrics to loocv_accuracy_results.csv
```

## Evaluation signals

- Reproduced LOOCV accuracies match the reported values: Bi-LSTM 71.51% and Transformer 67.61%, both below nanodisco/Nanoident's 83.68%
- Independent 12-motif test accuracies match reported values: Bi-LSTM 70.25% and Transformer 72.42%
- Trainable parameter counts match exactly: Nanoident 14,319, Bi-LSTM 36,693, Transformer 27,733
- All metrics for every variant are present as rows in loocv_accuracy_results.csv
- All variants are evaluated on identical LOOCV folds and the same independent test motif set, confirming protocol parity

## Limitations

- Parameter counts are for the specific implementations in the Nanoident repo (two-layer Bi-LSTM, one-layer Transformer); other layer counts change the numbers and the comparison
- The audit covers only methylation typing accuracy; fine mapping (methylated base position identification) metrics are not part of this comparison
- Preprocessing must reproduce the nanodisco-updated five-feature representation (std, time, proportion of reference bases, average quality score, plus base feature); differing features invalidate the fairness claim

## Evidence

- [other] LOOCV accuracies match reported Bi-LSTM/Transformer figures: [evidence span withheld — anchor status: not_found]
- [other] Validation parameters from the source task card: [evidence span withheld — anchor status: not_found]
- [other] Metric recording workflow: [evidence span withheld — anchor status: not_found]
- [abstract] LOOCV dataset scale: "The Leave-One-Out Cross-Validation (LOOCV) on the dataset containing 7 bacteria samples with 46 methylation motifs shows that Nanoident achieves ∼10% improvement in accuracy"
- [supplementary] Independent test dataset scale: "There are 12 different motifs in these two species, including 6 4mC motifs, 1 5mC motif, and 5 6mA motifs."
- [methods] Data conversion step into Python: "we need to convert it to a pkl file that can be used in a python environment using data_from_R_to_python.ipynb"
