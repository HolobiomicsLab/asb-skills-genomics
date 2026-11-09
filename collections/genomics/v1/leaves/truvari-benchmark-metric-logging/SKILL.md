---
name: truvari-benchmark-metric-logging
description: Use when training the ConsensuSV-ONT binary CNN classifier and you need
  per-epoch, sample-level SV-calling quality (Truvari precision/recall/F1) plus AUC,
  loss and learning rate to detect overfitting.
when_to_use_negative:
- When the task is final SV-caller benchmarking rather than monitoring CNN training
  dynamics — the test sample must remain untouched during training
- When no sample-level partition exists, since metrics must be computed per sample
  (one validation sample) rather than on shuffled variants
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: ConsensuSV-ONT
  role: Produces the consensus SV calls and encoded variant images/labels whose filtering
    quality is benchmarked per epoch
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: TensorFlow/Keras
  role: Trains the sequential binary CNN and logs per-epoch AUC, loss and learning
    rate during fit
- name: Truvari
  role: Computes per-epoch precision, recall and F1 of CNN-filtered variants against
    the validation sample's benchmark calls
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:716a0826d98398a7c140560457fa0c076acc40f72fa5ca79bb21604b60208365
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# truvari-benchmark-metric-logging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Log per-epoch Truvari precision, recall and F1 of the ConsensuSV-ONT CNN variant filter on the validation sample alongside AUC, loss and learning rate, so that training generalisation (plateau after epoch 2) can be monitored and used to decide on learning-rate reduction.

## When to use

Use when training the ConsensuSV-ONT binary CNN classifier and you need per-epoch, sample-level SV-calling quality (Truvari precision/recall/F1) plus AUC, loss and learning rate to detect overfitting.

## When NOT to use

- When the task is final SV-caller benchmarking rather than monitoring CNN training dynamics — the test sample must remain untouched during training
- When no sample-level partition exists, since metrics must be computed per sample (one validation sample) rather than on shuffled variants

## Inputs

- Encoded structural-variant images (50x50x3) with high-quality/low-quality labels from the ConsensuSV-ONT variant encoding step
- Sample-level train/validation/test split (one sample per partition)
- Compiled Keras sequential CNN model (Conv3D/BatchNormalization/MaxPooling3D blocks, Dense(32), sigmoid output)

## Outputs

- Per-epoch training-history CSV with Truvari precision, recall, F1, AUC, loss and learning-rate columns
- Saved trained model weights file
- Validation of generalisation state (validation metrics plateau after epoch 2)

## How to apply

Partition the encoded structural-variant images (50x50x3) and their high/low-quality labels by sample into training, validation and held-out test sets (one sample each). During Keras model.fit of the sequential Conv3D model (filters 4/8/16, Flatten to 1728, Dense(32), Dense(1) sigmoid; 59,929 total params), compute per-epoch Truvari precision, recall and F1 on the validation sample together with AUC, loss and learning rate. Monitor generalisation: validation metrics should improve after epoch 2 and then plateau while training AUC keeps increasing; this overfitting signal is the trigger to reduce the learning rate during training. Keep the held-out test sample untouched throughout and export the per-epoch metric log as a training-history CSV with Truvari precision/recall/F1, AUC, loss and learning-rate columns, plus the saved model weights file.

## Related tools

- **ConsensuSV-ONT** (Produces the consensus SV calls and encoded variant images/labels whose filtering quality is benchmarked per epoch) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **TensorFlow/Keras** (Trains the sequential binary CNN and logs per-epoch AUC, loss and learning rate during fit)
- **Truvari** (Computes per-epoch precision, recall and F1 of CNN-filtered variants against the validation sample's benchmark calls)

## Examples

```
model.fit(X_train, y_train, validation_data=(X_val, y_val), callbacks=[TruvariMetricsLogger(val_sample), LRReducer()])  # log Truvari P/R/F1, AUC, loss, lr per epoch
```

## Evaluation signals

- Training-history CSV contains all logged columns: Truvari precision, recall, F1, AUC, loss and learning rate
- Validation AUC and loss improve after epoch 2 and then plateau while training AUC keeps increasing (expected overfitting signature)
- Held-out test sample is never used during training or metric logging
- Truvari precision/recall/F1 are computed on the single validation sample per epoch, not on training data
- Trained model weights file exists and model summary matches 59,929 total / 59,873 trainable parameters

## Limitations

- Only three samples are available (one each for train/validation/test), so validation metrics reflect a single sample and may be noisy
- Generalisation is judged by a plateau after epoch 2; shorter trainings or different encodings may not show the same dynamic
- The learning rate must be reduced manually during training once overfitting is observed; the pipeline does not automate it

## Evidence

- [other] The CNN training history shows generalisation after the second epoch: validation AUC and loss plateaued while training AUC kept increasing (overfitting), so the learning rate was reduced during training.: [evidence span withheld — anchor status: not_found]
