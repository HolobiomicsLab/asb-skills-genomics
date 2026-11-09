---
name: convolutional-neural-network-training
description: Use when a labelled set of encoded structural-variant images (high-quality
  vs low-quality) needs a binary deep-learning classifier trained and its per-epoch
  generalisation behaviour monitored.
when_to_use_negative:
- The input is raw FASTQ/BAM data rather than already-encoded SV images — variant
  calling and image encoding must happen first in the nextflow pipeline
- You need multi-class SV type prediction rather than the binary high-quality/low-quality
  filter the sigmoid output provides
- The task is per-sample consensus SV calling rather than training/monitoring the
  CNN classifier itself
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TensorFlow/Keras
  role: Framework for building the sequential Conv3D model, compiling it for binary
    classification, and fitting while logging per-epoch metrics
- name: ConsensuSV-ONT
  role: Pipeline whose variant encoding step produces the labelled 50x50x3 SV images
    used as training input; trained CNN weights are reused for variant filtering
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Truvari
  role: Benchmarking tool used to compute per-epoch validation precision, recall and
    F1 during CNN training
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:efc67b535d3915450c36d00c7ac047f03d027bda51fbd9e9fa71272033c306f9
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# convolutional-neural-network-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train the ConsensuSV-ONT binary CNN that filters high-quality structural variants: three Conv3D+BatchNorm+MaxPooling3D blocks over 50x50x3 encoded SV images, fit in Keras with per-epoch monitoring of AUC, loss, learning rate and Truvari precision/recall/F1.

## When to use

Use when a labelled set of encoded structural-variant images (high-quality vs low-quality) needs a binary deep-learning classifier trained and its per-epoch generalisation behaviour monitored.

## When NOT to use

- The input is raw FASTQ/BAM data rather than already-encoded SV images — variant calling and image encoding must happen first in the nextflow pipeline
- You need multi-class SV type prediction rather than the binary high-quality/low-quality filter the sigmoid output provides
- The task is per-sample consensus SV calling rather than training/monitoring the CNN classifier itself

## Inputs

- Encoded structural-variant images (50x50x3) produced by the ConsensuSV-ONT variant encoding step
- Binary high-quality/low-quality labels for each encoded SV image
- Sample-level partition assignment (one training sample, one validation sample, one held-out test sample)

## Outputs

- Trained binary CNN model weights file (sequential Keras model, 59,929 total / 59,873 trainable parameters)
- Per-epoch training-history CSV with Truvari precision, recall, F1, AUC, loss and learning-rate columns
- Trained binary classifier used to filter high-quality variants in the ConsensuSV-ONT pipeline

## How to apply

Load the encoded structural-variant images with their high-quality/low-quality labels and partition the data by sample into training, validation and held-out test sets (one sample each). Build the sequential Keras model exactly as specified: Conv3D(4)+BatchNormalization+MaxPooling3D, Conv3D(8)+BatchNormalization+MaxPooling3D, Conv3D(16)+BatchNormalization+MaxPooling3D on 50x50x3 inputs, Flatten to 1728 units, Dense(32), and a final Dense(1) sigmoid output giving 59,929 total parameters (59,873 trainable). Compile for binary classification and fit, logging per-epoch Truvari precision, recall and F1 on the validation sample alongside AUC, loss and learning rate. Judge generalisation by the paper's observation that validation AUC and loss improve after epoch 2 and then plateau while training AUC keeps rising (overfitting), at which point the learning rate was reduced during training; keep the held-out test sample untouched throughout fitting. Finish by saving the trained model weights file and exporting the per-epoch metric log as a training-history CSV.

## Related tools

- **TensorFlow/Keras** (Framework for building the sequential Conv3D model, compiling it for binary classification, and fitting while logging per-epoch metrics)
- **ConsensuSV-ONT** (Pipeline whose variant encoding step produces the labelled 50x50x3 SV images used as training input; trained CNN weights are reused for variant filtering) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Truvari** (Benchmarking tool used to compute per-epoch validation precision, recall and F1 during CNN training)

## Examples

```
nextflow pipeline.nf --input files.csv --outdir ConsensuSV-ONT --ref reference.fa
```

## Evaluation signals

- Model summary matches the paper architecture: 59,929 total parameters, 59,873 trainable, 56 non-trainable, ending in a Dense(1) sigmoid output
- Training history shows validation AUC and loss improving after epoch 2 and then plateauing while training AUC keeps increasing (overfitting), triggering a learning-rate reduction
- Held-out test sample was untouched during fitting (only used after training completes)
- Exported training-history CSV contains per-epoch columns for Truvari precision, recall, F1, AUC, loss and learning rate
- Saved model weights file exists and reloads into the specified sequential architecture

## Limitations

- Overfitting appears after epoch 2 (training AUC rises while validation metrics plateau), so long training runs without learning-rate reduction degrade generalisation
- Data split is by sample with only one sample per partition, so validation/test metric variance is high and results depend heavily on which sample fills each split
- No version numbers or hyperparameter details beyond the logged learning-rate behaviour are provided in the article or README

## Evidence
