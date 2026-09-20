---
name: train-validation-test-sample-splitting
description: 'Use when training the ConsensuSV-ONT CNN variant filter: you have encoded
  SV images plus high-quality/low-quality labels and need a per-sample split to measure
  generalisation without leaking variants between samples.'
when_to_use_negative:
- Variants from the same sample are split across train and validation sets — this
  leaks sample-level signal and invalidates the generalisation check
- You need per-callers consensus calling or SV detection itself; this skill only covers
  training the CNN filter, not running the six SV callers
- Your variants are not encoded as images and lack high-quality/low-quality labels
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ConsensuSV-ONT
  role: Upstream pipeline whose variant encoding produces the images and high-quality/low-quality
    labels used for CNN training
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: TensorFlow/Keras
  role: Framework used to build and fit the sequential Conv3D binary classifier (50x50x3
    inputs, Dense(1) sigmoid output)
- name: Truvari
  role: Computes per-epoch precision, recall and F1 on the validation sample to monitor
    generalisation during fitting
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:1340850d973184fd2ed8fcbe32d3be89db262e6fd1f61a6ed028f440997d5d26
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
  repo_url: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
  tools:
  - ConsensuSV-ONT
  - TensorFlow/Keras
  - Truvari
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# Train-validation-test sample splitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partition encoded structural-variant images from ConsensuSV-ONT into train, validation, and held-out test sets split by sample, so the binary CNN (50x50x3 Conv3D model) can be fit while validation AUC/loss and per-epoch Truvari metrics monitor generalisation.

## When to use

Use when training the ConsensuSV-ONT CNN variant filter: you have encoded SV images plus high-quality/low-quality labels and need a per-sample split to measure generalisation without leaking variants between samples.

## When NOT to use

- Variants from the same sample are split across train and validation sets — this leaks sample-level signal and invalidates the generalisation check
- You need per-callers consensus calling or SV detection itself; this skill only covers training the CNN filter, not running the six SV callers
- Your variants are not encoded as images and lack high-quality/low-quality labels

## Inputs

- Encoded structural-variant images (50x50x3) from the ConsensuSV-ONT variant encoding step
- Binary high-quality/low-quality labels for the encoded variants
- Sample-level identifiers so variants can be grouped by ONT sample

## Outputs

- Trained CNN model weights file
- Training-history CSV with per-epoch Truvari precision/recall/F1, AUC, loss and learning-rate columns
- Fitted Keras sequential model (59,929 total params, 59,873 trainable) ready for variant filtering

## How to apply

Load the encoded structural-variant images and their high-quality/low-quality labels produced by the ConsensuSV-ONT variant encoding step, then partition them by sample into a training set, a validation set, and a held-out test set (one sample each). Build the sequential Keras model described in the repository README — three Conv3D + BatchNormalization + MaxPooling3D blocks (filters 4/8/16 on 50x50x3 inputs), Flatten to 1728 units, Dense(32) and a final Dense(1) sigmoid output (59,929 total params, 59,873 trainable) — and compile it for binary classification. Fit while logging per-epoch Truvari precision, recall and F1 on the validation sample together with AUC, loss and learning rate. Keep the held-out test sample completely untouched during fitting. Accept training when the history shows generalisation after epoch 2 — validation AUC and loss plateau while training AUC keeps increasing — and reduce the learning rate at that decision point if overfitting appears.

## Related tools

- **ConsensuSV-ONT** (Upstream pipeline whose variant encoding produces the images and high-quality/low-quality labels used for CNN training) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **TensorFlow/Keras** (Framework used to build and fit the sequential Conv3D binary classifier (50x50x3 inputs, Dense(1) sigmoid output))
- **Truvari** (Computes per-epoch precision, recall and F1 on the validation sample to monitor generalisation during fitting)

## Examples

```
model.fit(X_train, y_train, validation_data=(X_val, y_val), epochs=10)  # sequential Keras Conv3D model, 50x50x3 inputs, sigmoid output
```

## Evaluation signals

- Validation AUC and loss improve after epoch 2 and then plateau while training AUC keeps increasing — the expected generalisation signature
- Saved model weights file exists and reloads into the documented architecture (59,929 total params, 59,873 trainable, 56 non-trainable)
- Training-history CSV exists and contains Truvari precision, recall, F1, AUC, loss and learning-rate columns for every epoch
- Held-out test sample shows zero presence in any training or validation log
- Learning-rate log shows the reduction applied when validation metrics plateau

## Limitations

- The split is one sample per set, so validation and test metrics rest on a single sample each and may not capture population diversity
- Generalisation signal is based on a small number of epochs (plateau after epoch 2), so longer training may behave differently
- The pipeline README provides the architecture but no version numbers or hyperparameters for the training procedure in this context

## Evidence
