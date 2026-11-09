---
name: training-generalization-monitoring
description: Use when fitting a small Keras CNN classifier (e.g. three Conv3D+BatchNormalization+MaxPooling3D
  blocks on encoded SV images) and you need per-epoch generalisation signals — validation
  Truvari precision/recall/F1, AUC, loss and learning rate — to decide when to reduce
  the learning rate.
when_to_use_negative:
- Evaluating final performance on the held-out test sample — that sample must stay
  untouched during training and per-epoch monitoring.
- Upstream consensus SV calling or filtering of caller outputs — this skill covers
  only CNN training dynamics, not the six-caller consensus stage.
- Datasets that cannot be split by sample into training/validation/test (one sample
  each) — the generalisation signal in this method is defined on a per-sample validation
  split.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TensorFlow/Keras
  role: Builds and compiles the sequential Conv3D CNN and fits it while logging per-epoch
    metrics (AUC, loss, learning rate) into the training history
- name: Truvari
  role: Computes per-epoch validation-sample precision, recall and F1 logged alongside
    AUC and loss during training
- name: ConsensuSV-ONT
  role: Supplies the encoded 50x50x3 SV images and high-quality/low-quality labels
    used for CNN training and validation
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:ebb56fc1bef051b38e07fb227bc6418da56f7531cd5e02cc7cd0748cb759adfc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
  tools:
  - ConsensuSV-ONT
  - TensorFlow/Keras
  - Truvari
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# training-generalization-monitoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Monitors per-epoch training dynamics of the ConsensuSV-ONT binary CNN (59,929-param Keras Conv3D model on 50x50x3 structural-variant images) by logging validation Truvari precision/recall/F1, AUC, loss and learning rate, so that the epoch-2 generalisation plateau and the onset of overfitting can be detected and the learning rate reduced during training.

## When to use

Use when fitting a small Keras CNN classifier (e.g. three Conv3D+BatchNormalization+MaxPooling3D blocks on encoded SV images) and you need per-epoch generalisation signals — validation Truvari precision/recall/F1, AUC, loss and learning rate — to decide when to reduce the learning rate.

## When NOT to use

- Evaluating final performance on the held-out test sample — that sample must stay untouched during training and per-epoch monitoring.
- Upstream consensus SV calling or filtering of caller outputs — this skill covers only CNN training dynamics, not the six-caller consensus stage.
- Datasets that cannot be split by sample into training/validation/test (one sample each) — the generalisation signal in this method is defined on a per-sample validation split.

## Inputs

- Encoded 50x50x3 structural-variant images with high-quality/low-quality labels from the ConsensuSV-ONT variant encoding step
- Sample-partitioned training, validation and held-out test sets (one sample each)
- Truth VCF for the validation sample used to compute per-epoch Truvari precision, recall and F1

## Outputs

- Trained CNN model weights file
- Training-history CSV with per-epoch Truvari precision, recall, F1, AUC, loss and learning-rate columns

## How to apply

Partition the encoded structural-variant images and their high-quality/low-quality labels by sample into a training set, a validation set and a held-out test set (one sample each), keeping the test sample untouched throughout training. Build and compile the sequential Keras model for binary classification (sigmoid output, 59,929 total / 59,873 trainable params) and fit it while logging per-epoch Truvari precision, recall and F1 computed on the validation sample together with AUC, loss and learning rate. Inspect the history for the generalisation signature: validation AUC and loss improve up to epoch 2 and then plateau while training AUC keeps increasing (overfitting). Reduce the learning rate during training once this plateau is confirmed. Finish by saving the trained model weights file and exporting the per-epoch metric log as a training-history CSV; accept training only when the history shows the epoch-2 plateau and both artefacts exist with the required columns.

## Related tools

- **TensorFlow/Keras** (Builds and compiles the sequential Conv3D CNN and fits it while logging per-epoch metrics (AUC, loss, learning rate) into the training history)
- **Truvari** (Computes per-epoch validation-sample precision, recall and F1 logged alongside AUC and loss during training)
- **ConsensuSV-ONT** (Supplies the encoded 50x50x3 SV images and high-quality/low-quality labels used for CNN training and validation) — github.com/SFGLab/ConsensuSV-ONT-pipeline

## Examples

```
model.compile(loss='binary_crossentropy', metrics=['AUC']); hist = model.fit(X_train, y_train, validation_data=(X_val, y_val), callbacks=[ReduceLROnPlateau()], epochs=E)
```

## Evaluation signals

- Training-history CSV contains per-epoch columns for Truvari precision, recall, F1, AUC, loss and learning rate.
- History shows generalisation after epoch 2: validation AUC and loss plateau while training AUC keeps increasing.
- The held-out test sample remains untouched during training (used only for final evaluation).
- Both the model weights file and the training-history CSV exist at the end of the run.
- Model architecture matches the reference summary: 59,929 total params, 59,873 trainable, 56 non-trainable.

## Limitations

- The split uses only one sample per partition (training/validation/test), so generalisation estimates rest on a single validation and test sample.
- Overfitting sets in early — validation metrics plateau after epoch 2 while training AUC keeps rising — so training beyond the plateau without reducing the learning rate wastes compute and degrades generalisation behaviour.
- Per-epoch validation metrics depend on Truvari comparison against the validation sample's truth set, so truth-VCF quality directly conditions the monitoring signal.
- Repository documentation gap: no changelog found, and no version numbers or parameters are documented for the trained model in the discussion.

## Evidence

- [results] validation AUC and loss plateaued while training AUC kept increasing (overfitting), so the learning rate was reduced during training.: [evidence span withheld — anchor status: not_found]
- [methods] Partition the data by sample into a training set, a validation set and a held-out test set (one sample each).: [evidence span withheld — anchor status: not_found]
- [methods] Compile for binary classification and fit, logging per-epoch Truvari precision, recall and F1 on the validation sample together with AUC, loss and learning rate.: [evidence span withheld — anchor status: not_found]
- [readme] Total params: 59,929
Trainable params: 59,873: "Total params: 59,929
Trainable params: 59,873"
