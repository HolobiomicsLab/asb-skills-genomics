---
name: cell-mask-binarization-and-resizing
description: Use when segmented single-cell masks from multiplexed tissue images (e.g.,
  MIBI or multiplexed immunofluorescence) must be converted to a fixed-size, orientation-normalized
  binary format for downstream deep learning shape characterization with an autoencoder.
when_to_use_negative:
- Input is unsegmented multiplexed image data — MIRIAM segmentation (pixel classification
  and membrane extension) must be run first to obtain cell masks
- The goal is marker intensity quantification per cell compartment (nucleus, membrane,
  cytoplasm), which MIRIAM performs separately from shape characterization
- Cells need per-compartment membrane/cytoplasm measurements rather than whole-cell
  shape encoding
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Pipeline whose segmentation output provides the cell masks that are binarized,
    aligned, and resized; repository supplies the autoencoder training/encoding code
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Matlab Deep Learning Toolbox implementation for training the autoencoder and
    encoding cell shapes
- name: Python
  role: Python Keras implementation provided in the MIRIAM repository as an alternative
    for autoencoder training/encoding
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:ef61939bff103c8291bf350f9109ee41066508931f6140bca3d48c7b226132f1
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
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cell-mask-binarization-and-resizing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Standardizes segmented cell shapes from multiplexed tissue images (per the MIRIAM pipeline) by binarizing each cell mask, rotating it so the cell's major axis is consistently aligned, and resizing it to a fixed 128 x 128 pixel matrix, ready for autoencoder encoding into shape latent vectors.

## When to use

Use when segmented single-cell masks from multiplexed tissue images (e.g., MIBI or multiplexed immunofluorescence) must be converted to a fixed-size, orientation-normalized binary format for downstream deep learning shape characterization with an autoencoder.

## When NOT to use

- Input is unsegmented multiplexed image data — MIRIAM segmentation (pixel classification and membrane extension) must be run first to obtain cell masks
- The goal is marker intensity quantification per cell compartment (nucleus, membrane, cytoplasm), which MIRIAM performs separately from shape characterization
- Cells need per-compartment membrane/cytoplasm measurements rather than whole-cell shape encoding

## Inputs

- Segmented cell masks from MIRIAM (per-cell segmentation output for each imaging position)
- MIRIAM example dataset from the Coffey-Lab/MIRIAM repository (illustrative masks)

## Outputs

- Aligned, binarized cell images resized to a 128 x 128 pixel matrix
- Shape latent vectors for all cells saved to a CSV file

## How to apply

Starting from the segmented cell masks produced by the MIRIAM pipeline, binarize each segmented cell at each imaging position, then rotate each binary mask so the cell's major axis is in a consistent orientation across all cells. Resize every aligned, binarized cell to a fixed 128 x 128 pixel matrix so cells of different sizes are comparable. Train the autoencoder on a random 20% subset of the resized cells using the Matlab Deep Learning Toolbox or the Python Keras implementation provided in the Coffey-Lab/MIRIAM repository, then pass all cells (including held-out cells) through the trained encoder. Save the resulting shape latent vectors for all cells to a CSV file. The rationale is that a fixed, axis-aligned representation removes position, scale, and orientation variability so the autoencoder captures only shape information, which the authors chose because methods using defined shape descriptors are limited.

## Related tools

- **MIRIAM** (Pipeline whose segmentation output provides the cell masks that are binarized, aligned, and resized; repository supplies the autoencoder training/encoding code) — https://github.com/Coffey-Lab/MIRIAM
- **Matlab** (Matlab Deep Learning Toolbox implementation for training the autoencoder and encoding cell shapes)
- **Python** (Python Keras implementation provided in the MIRIAM repository as an alternative for autoencoder training/encoding)

## Evaluation signals

- Every binarized cell is a 128 x 128 pixel matrix after resizing — verify output dimensions
- All cells share a consistent major-axis orientation after the rotation/alignment step
- The autoencoder is trained only on a random 20% subset of cells; latent vectors must be produced for all cells, including those excluded from training
- Latent vectors for all cells are persisted to a CSV file
- t-SNE run on the encoded latent vectors excluding training cells should reproduce the paper's shape-space analysis (Fig. 3B); Matlab and Python implementations should yield largely similar results (Supplemental Figure 6)

## Limitations

- MIRIAM's t-SNE shape analysis of the encoded latent vectors did not reveal discernible cellular populations or clear patterns in marker expression (Supplemental Figure 5), so shape latents may not cluster informatively
- Training details/weights for the autoencoder cell-shape model are not fully provided; users must rely on the repository tutorial and example data
- Image stacks for the example data are not included in the repository (too large) and must be regenerated or downloaded separately
- The pipeline requires Ilastik for some tasks, adding an external dependency

## Evidence

- [intro] Finally, cell shapes are characterized using an autoencoder neural network.: "Finally, cell shapes are characterized using an autoencoder neural network."
- [results] When only the encoded latent vectors for cell shape, excluding those used to train the autoencoder, are used as an input to t-SNE analysis (Fig. 3B): "When only the encoded latent vectors for cell shape, excluding those used to train the autoencoder, are used as an input to t-SNE analysis (Fig. 3B)"
- [discussion] We chose an autoencoder as other methods that use defined shape descriptors: "We chose an autoencoder as other methods that use defined shape descriptors"
- [results] Results of MIRIAM using Matlab or Python implementations show largely similar results (Supplemental Figure 6).: "Results of MIRIAM using Matlab or Python implementations show largely similar results (Supplemental Figure 6)."
- [readme] This pipeline requires Iltastik for some tasks. Download Ilastik here: https://www.ilastik.org/download.html: "This pipeline requires Iltastik for some tasks. Download Ilastik here: https://www.ilastik.org/download.html"
