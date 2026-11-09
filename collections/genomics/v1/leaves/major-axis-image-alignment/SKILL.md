---
name: major-axis-image-alignment
description: Use when segmented single-cell masks from multiplexed tissue images (e.g.,
  MIRIAM's multiplexed immunofluorescence or MIBI datasets) must be orientation- and
  size-normalized prior to training or applying an autoencoder shape descriptor.
when_to_use_negative:
- Cells are not individually segmented (e.g., raw multi-channel image stacks without
  cell masks) — run MIRIAM segmentation first
- The goal is marker intensity quantification per compartment (nucleus, membrane,
  cytoplasm), which does not require shape alignment or the autoencoder
- Only classification by marker intensity is needed (e.g., t-SNE on marker intensities
  as in Figure 3A), where shape descriptors are irrelevant
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Pipeline providing the segmented cell masks and the autoencoder shape-descriptor
    stage into which the aligned, resized cell images are fed
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Implementation (with Deep Learning Toolbox) for training the autoencoder on
    the aligned 128×128 cell images
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative Keras implementation for training the autoencoder on the aligned
    cell images
  repo: https://github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:1a8cfa29dc38ba9bc4f99110d019a288682419eb2ac4e68d98bac4d56137626f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  repo_url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - MIRIAM
  - Matlab
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Major-axis image alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocessing step in MIRIAM's deep learning cell-shape descriptor in which each segmented cell mask is binarized, rotated so its major axis is consistently oriented, and resized to a fixed 128×128 pixel matrix before autoencoder encoding.

## When to use

Use when segmented single-cell masks from multiplexed tissue images (e.g., MIRIAM's multiplexed immunofluorescence or MIBI datasets) must be orientation- and size-normalized prior to training or applying an autoencoder shape descriptor.

## When NOT to use

- Cells are not individually segmented (e.g., raw multi-channel image stacks without cell masks) — run MIRIAM segmentation first
- The goal is marker intensity quantification per compartment (nucleus, membrane, cytoplasm), which does not require shape alignment or the autoencoder
- Only classification by marker intensity is needed (e.g., t-SNE on marker intensities as in Figure 3A), where shape descriptors are irrelevant

## Inputs

- Segmented single-cell binary masks from multiplexed tissue images (MIRIAM segmentation output)
- MIRIAM example dataset from the Coffey-Lab/MIRIAM repository (image stacks excluded; regenerated or downloaded via the README Dropbox link)

## Outputs

- Orientation-aligned, binarized cell images resized to 128×128 pixel matrices
- Latent shape vectors for all cells, saved as a CSV file

## How to apply

Load the segmented cell masks (e.g., from the MIRIAM example dataset in the Coffey-Lab/MIRIAM repository), binarize each mask, and compute the cell's major axis. Rotate every binarized cell so that its major axis points in the same orientation across all cells, eliminating rotational variability as a confounder of shape. Then resize each aligned, binarized cell to a fixed 128×128 pixel matrix so all cells share an identical input dimensionality. These normalized images feed a single-layer autoencoder trained on a random 20% subset of cells (via Matlab Deep Learning Toolbox or the Python Keras implementation in MIRIAM); all cells, including held-out ones, are then passed through the trained encoder to yield latent shape vectors. Correct application is confirmed when the encoded latent vectors, excluding training cells, can be used as t-SNE inputs (e.g., Figure 3B in the paper) without orientation-driven artifacts.

## Related tools

- **MIRIAM** (Pipeline providing the segmented cell masks and the autoencoder shape-descriptor stage into which the aligned, resized cell images are fed) — https://github.com/Coffey-Lab/MIRIAM
- **Matlab** (Implementation (with Deep Learning Toolbox) for training the autoencoder on the aligned 128×128 cell images) — https://github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative Keras implementation for training the autoencoder on the aligned cell images) — https://github.com/Coffey-Lab/MIRIAM

## Evaluation signals

- All output cell images are exactly 128×128 pixels and binary (values in {0,1})
- Every cell's major axis is oriented consistently (e.g., same angle) across the full cell set before encoding
- Latent vectors are produced for all segmented cells, including those excluded from the 20% autoencoder training subset
- Downstream t-SNE on latent vectors (excluding training cells) runs to completion, mirroring the paper's Figure 3B analysis
- Latent vectors are saved in a CSV file with one row per cell

## Limitations

- The paper's t-SNE shape analysis did not reveal any discernible cellular populations or clear patterns in marker expression, so aligned shape descriptors may not yield biologically separable clusters on their own
- Training details/weights for the autoencoder cell-shape model are not fully provided in the article text
- Example data in the repository excludes image stacks (too large), which must be regenerated or downloaded separately via the README link

## Evidence
