---
name: deep-learning-model-training
description: Use when you have binary segmented cell masks from multiplexed imaging
  (e.g., MIBI or multiplexed immunofluorescence) and need a learned, fixed-length
  shape representation per cell rather than hand-crafted morphological features.
when_to_use_negative:
- Input is a per-cell marker-intensity feature table rather than segmented cell masks
  — shape encoding is not applicable.
- Goal is per-compartment marker quantification (nucleus, membrane, cytoplasm) — that
  is MIRIAM's quantification stage, not the shape-descriptor stage.
- Latent vectors have already been computed and the goal is downstream analysis such
  as t-SNE on the encodings.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3838
tools:
- name: MIRIAM
  role: Provides the autoencoder cell-shape descriptor implementation and example
    segmented-cell data used for training and encoding
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Matlab Deep Learning Toolbox implementation for training the cell-shape autoencoder
- name: Python
  role: Python Keras implementation for training the cell-shape autoencoder
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:d1b922948d6091637b5fa12d4c676c220802e277ac3a504e202483aee3323e2e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - MIRIAM
  - Matlab
  - Python
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3838
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# deep-learning-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains an autoencoder neural network to produce a compact latent-vector descriptor of cell shape from segmented multiplexed tissue images, as implemented in the MIRIAM pipeline (Matlab Deep Learning Toolbox or Python Keras) for downstream shape-based analyses such as t-SNE.

## When to use

Use when you have binary segmented cell masks from multiplexed imaging (e.g., MIBI or multiplexed immunofluorescence) and need a learned, fixed-length shape representation per cell rather than hand-crafted morphological features.

## When NOT to use

- Input is a per-cell marker-intensity feature table rather than segmented cell masks — shape encoding is not applicable.
- Goal is per-compartment marker quantification (nucleus, membrane, cytoplasm) — that is MIRIAM's quantification stage, not the shape-descriptor stage.
- Latent vectors have already been computed and the goal is downstream analysis such as t-SNE on the encodings.

## Inputs

- Segmented cell masks from multiplexed tissue images (e.g., MIRIAM example dataset from github.com/Coffey-Lab/MIRIAM)
- Binary cell masks, reoriented by major axis and resized to 128 x 128 pixel matrices
- Matlab Deep Learning Toolbox or the Python Keras implementation provided in the MIRIAM repository

## Outputs

- Trained autoencoder model (encoder weights) for cell shape
- Shape latent vectors for all cells (CSV file)
- Encoded latent vectors suitable for t-SNE analysis, excluding cells used for autoencoder training

## How to apply

Extract each segmented cell mask from the MIRIAM example dataset, binarize it, and rotate it so the cell's major axis is consistently aligned. Resize each aligned, binarized cell to a fixed 128 x 128 pixel matrix. Train the autoencoder on a random 20% subset of the cells, using either the Matlab Deep Learning Toolbox or the Python Keras implementation provided in the Coffey-Lab/MIRIAM repository. Pass all cells (including those not used for training) through the trained encoder to obtain shape latent vectors, and save the latent vectors for all cells to a CSV file. An autoencoder was chosen in MIRIAM because methods using defined shape descriptors were less suitable; the resulting encodings can be used, for example, as t-SNE inputs excluding training cells.

## Related tools

- **MIRIAM** (Provides the autoencoder cell-shape descriptor implementation and example segmented-cell data used for training and encoding) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Matlab Deep Learning Toolbox implementation for training the cell-shape autoencoder)
- **Python** (Python Keras implementation for training the cell-shape autoencoder)

## Evaluation signals

- Every encoded cell is a binarized mask, major-axis aligned, and resized to exactly a 128 x 128 pixel matrix before feeding to the autoencoder.
- Training uses a random 20% subset of cells; t-SNE downstream inputs must exclude cells used for autoencoder training.
- Latent vectors for all cells are saved as a CSV file with one vector per segmented cell.
- Matlab and Python implementations should produce largely similar results (as reported in Supplemental Figure 6 of the MIRIAM paper).
- Latent vectors are checkable by reproducing the t-SNE shape analysis; note the article reports no discernible cellular populations or clear marker-expression patterns from shape alone.

## Limitations

- Training details and model weights for the autoencoder cell-shape model were not fully provided in the article.
- No version numbers are stated for the Matlab or Python implementations.
- The t-SNE shape analysis in MIRIAM did not reveal discernible cellular populations or clear patterns in marker expression, so shape latents alone may have limited biological interpretability.
- Image stacks are too large to ship with the MIRIAM example data and must be regenerated by the user; example data contains all other files.

## Evidence

- [other] training and encoding procedure (source card workflow): [evidence span withheld — anchor status: not_found]
