---
name: latent-vector-encoding-and-export
description: Use when segmented single-cell masks from multiplexed imaging (e.g.,
  MIBI or multiplexed immunofluorescence) need a compact numeric shape descriptor,
  and latent vectors must be exported (e.g., to CSV) for clustering or embedding.
when_to_use_negative:
- The goal is marker-intensity quantification per cell compartment (nucleus, membrane,
  cytoplasm) rather than shape characterization — the pixel-classification quantification
  step of MIRIAM handles that
- Input is already a feature/latent table or embedding (e.g., prior t-SNE output)
  rather than raw segmented cell masks
- Cells lack identifiable major axes (e.g., round, non-epithelial morphologies) where
  major-axis alignment adds no discriminating orientation information
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Provides the autoencoder cell-shape descriptor pipeline, example segmented
    data, and Matlab/Python implementations used to generate and export latent vectors
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Matlab Deep Learning Toolbox implementation for training the autoencoder and
    encoding all cells to latent vectors
- name: Python
  role: Python Keras implementation provided in the MIRIAM repository for training
    the autoencoder and encoding all cells to latent vectors
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:b5a751f0093b310dc5fa11396b03c375f185a5b2472a4b6fc2b718a730eb986d
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# latent-vector-encoding-and-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes segmented cell shapes from multiplexed tissue images by passing aligned, binarized 128x128 cell masks through a trained MIRIAM autoencoder, producing shape latent vectors that are exported for downstream analyses such as t-SNE.

## When to use

Use when segmented single-cell masks from multiplexed imaging (e.g., MIBI or multiplexed immunofluorescence) need a compact numeric shape descriptor, and latent vectors must be exported (e.g., to CSV) for clustering or embedding.

## When NOT to use

- The goal is marker-intensity quantification per cell compartment (nucleus, membrane, cytoplasm) rather than shape characterization — the pixel-classification quantification step of MIRIAM handles that
- Input is already a feature/latent table or embedding (e.g., prior t-SNE output) rather than raw segmented cell masks
- Cells lack identifiable major axes (e.g., round, non-epithelial morphologies) where major-axis alignment adds no discriminating orientation information

## Inputs

- Segmented single-cell binary masks from MIRIAM segmentation of multiplexed tissue images
- Trained MIRIAM autoencoder model (Matlab Deep Learning Toolbox or Python Keras implementation)
- Random 20% subset of cells designated as the autoencoder training set

## Outputs

- Shape latent vectors (encoder outputs) for every segmented cell
- CSV file containing the latent vectors for all cells

## How to apply

Load the segmented cell masks from the MIRIAM example dataset in the Coffey-Lab/MIRIAM repository, binarize each cell, rotate it so the major axis is consistently aligned, and resize it to a fixed 128x128 pixel matrix. Train the autoencoder neural network on a random 20% subset of the cells using either the Matlab Deep Learning Toolbox or the Python Keras implementation provided in the MIRIAM repository. Then pass all cells — including the ~80% held out from training — through the trained encoder half of the network to obtain the compressed shape latent vectors. Save the latent vectors for all cells to a CSV file so they can serve as inputs to downstream analyses like t-SNE on shape, excluding the vectors of cells used for autoencoder training.

## Related tools

- **MIRIAM** (Provides the autoencoder cell-shape descriptor pipeline, example segmented data, and Matlab/Python implementations used to generate and export latent vectors) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Matlab Deep Learning Toolbox implementation for training the autoencoder and encoding all cells to latent vectors)
- **Python** (Python Keras implementation provided in the MIRIAM repository for training the autoencoder and encoding all cells to latent vectors)

## Evaluation signals

- Every segmented cell at each imaging position yields exactly one latent vector; the exported CSV row count matches the segmented cell count
- All input masks are verified to be binarized, major-axis aligned, and resized to 128x128 matrices before encoding
- Cells whose vectors were used for autoencoder training (the random 20% subset) are flagged or identified so they can be excluded from downstream shape-based t-SNE
- Latent vectors from the Matlab and Python implementations should be largely similar, per the article's comparison of the two implementations (Supplemental Figure 6)
- Downstream t-SNE using the exported shape latent vectors (excluding training cells) can be reproduced as in Fig. 3B

## Limitations

- Training details and pretrained weights for the autoencoder cell-shape model are not fully provided in the article text
- No version numbers are stated for the Matlab or Python implementations
- In the article's colorectal adenoma application, t-SNE analysis on the shape latent vectors did not reveal any discernible cellular populations or clear patterns in marker expression (Supplemental Figure 5)

## Evidence
