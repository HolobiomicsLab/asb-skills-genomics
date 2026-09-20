---
name: multiplexed-image-pixel-classification
description: Use when input is a multi-marker (e.g., 36-marker MIBI or multiplexed
  immunofluorescence) tissue image stack needing pixel-level compartment labels (nucleus,
  membrane, cytoplasm/other) before single-cell segmentation and per-cell marker quantification.
when_to_use_negative:
- Input is already a per-cell feature/quantification table or segmentation mask —
  pixel classification is upstream and unnecessary
- The imaging data is single-channel or non-multiplexed histology where marker-based
  compartment definitions (nucleus, membrane, cytoplasm) cannot be learned
- The goal is a deep-learning pretrained segmentation (e.g., Mesmer) rather than a
  trained-on-your-data random forest classifier
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Pipeline whose random-forest-based machine learning pixel classification step
    defines cellular compartments as input to membrane-extension cell segmentation
  repo: github.com/Coffey-Lab/MIRIAM
- name: ilastik
  role: Required by the MIRIAM pipeline for some tasks, including the interactive
    random-forest pixel classification training
- name: Matlab
  role: One of the two MIRIAM implementations for running pixel classification and
    downstream segmentation on multiplexed imaging data
- name: Python
  role: Alternative MIRIAM implementation showing largely similar results to Matlab
    for running the pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:ef0c4c940da0259a34d4684935d09ca089f3791eb4368cb36e3f11015e9fb02a
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
  - ilastik
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# multiplexed-image-pixel-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Random-forest-based machine learning pixel classification of multiplexed tissue imaging data (e.g., MIBI or multiplexed immunofluorescence) to label nuclei, membranes, and other intracellular structures, serving as the input to downstream cell segmentation and per-compartment quantification as in the MIRIAM pipeline.

## When to use

Use when input is a multi-marker (e.g., 36-marker MIBI or multiplexed immunofluorescence) tissue image stack needing pixel-level compartment labels (nucleus, membrane, cytoplasm/other) before single-cell segmentation and per-cell marker quantification.

## When NOT to use

- Input is already a per-cell feature/quantification table or segmentation mask — pixel classification is upstream and unnecessary
- The imaging data is single-channel or non-multiplexed histology where marker-based compartment definitions (nucleus, membrane, cytoplasm) cannot be learned
- The goal is a deep-learning pretrained segmentation (e.g., Mesmer) rather than a trained-on-your-data random forest classifier

## Inputs

- Multi-channel multiplexed tissue image stack (e.g., MIBI breast carcinoma TMA, 2048x2048 pixels at 0.5 um/pixel, 36 markers)
- Multiplexed immunofluorescence images of tissue (e.g., precancerous colorectal adenomas)
- Training annotations / labels for nuclei, membranes, and other intracellular structures supplied via ilastik

## Outputs

- Pixel-class probability/label maps identifying nuclei, membranes, and other intracellular structures
- Input features for MIRIAM membrane-extension and internal-membrane cell-separation routines
- Basis for per-cell, per-compartment marker intensity quantification (entire cell, nucleus, membrane, cytoplasm)

## How to apply

Collect the entire set of markers from the image stack (e.g., the 36-marker MIBI breast carcinoma TMA at 0.5 um/pixel) rather than a subset, because using the entire set to define intracellular structures with machine learning produced more complete membranes in MIRIAM. Train the random-forest classifier using MIRIAM's machine-learning pixel classification step (the Matlab/Python implementation requires ilastik for some tasks, so annotations are made in ilastik's interactive training workflow) to define nuclei, membranes, and other intracellular structures. The resulting pixel-class probability maps feed MIRIAM's membrane-extension and internal-membrane cell-separation routines, so completeness of membrane labeling — not just nuclear accuracy — is the quality criterion to check. After classification, proceed to per-cell quantification of each marker's intensity over the entire cell and in the nucleus, membrane, and cytoplasm compartments.

## Related tools

- **MIRIAM** (Pipeline whose random-forest-based machine learning pixel classification step defines cellular compartments as input to membrane-extension cell segmentation) — github.com/Coffey-Lab/MIRIAM
- **ilastik** (Required by the MIRIAM pipeline for some tasks, including the interactive random-forest pixel classification training)
- **Matlab** (One of the two MIRIAM implementations for running pixel classification and downstream segmentation on multiplexed imaging data)
- **Python** (Alternative MIRIAM implementation showing largely similar results to Matlab for running the pipeline)

## Evaluation signals

- Membrane classes should be visibly more complete when all markers are used for training rather than a subset — incomplete membranes downstream indicate insufficient marker coverage in classification
- Segmentation quality downstream can be assessed against manually annotated data with image-level DSC/JSC metrics; MIRIAM reported DSC of 0.65 ± 0.08 and JSC of 0.48 ± 0.08 as a reference range
- Matlab and Python implementations of the same classification/segmentation should give largely similar results (as shown in Supplemental Figure 6)
- Per-cell quantification tables must contain intensity values for every marker in each compartment (entire cell, nucleus, membrane, cytoplasm)

## Limitations

- Random-forest pixel classification requires manual training annotations; no pretrained weights or version numbers are stated for the Matlab or Python implementations in the provided context
- Performance depends on image quality — the breast carcinoma MIBI data had lower image quality and a different marker set than the colorectal adenoma data, requiring retraining on the full 36-marker set
- The autoencoder cell-shape model and downstream quantitative similarity scores are only referenced via figure pointers; training details/weights for the autoencoder are not provided in the article section

## Evidence
