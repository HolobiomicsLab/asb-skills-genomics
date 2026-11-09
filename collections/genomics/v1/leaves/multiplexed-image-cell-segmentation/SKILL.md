---
name: multiplexed-image-cell-segmentation
description: Use when segmenting individual cells in multiplexed immunofluorescence
  (e.g., colorectal adenoma) or MIBI breast carcinoma tissue images where columnar
  epithelial cells have incomplete membranes and nuclear-dilation or Voronoi methods
  undersegment.
when_to_use_negative:
- The goal is comparison against deep-learning segmentation on non-epithelial tissues
  — Mesmer was trained on normal mouse small intestine, normal human colon, and colorectal
  cancer but not colorectal adenomas, so cross-method conclusions are tissue-specific
- Input images are single-channel or lack membrane markers (e.g., NaKATPase) — excluding
  membrane markers and DAPI from pixel classification decreased similarity to manually
  annotated borders
- The task is cell clustering or population discovery rather than segmentation — t-SNE
  shape analysis in this study did not reveal discernible cellular populations or
  clear patterns in marker expression
tools:
- name: MIRIAM
  role: 'Primary pipeline: pixel classification input, internal-membrane detection,
    ''connects the dots'' resegmentation, and per-cell quantification'
  repo: github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: Random-forest pixel classification of nucleus/membrane/cytoplasm compartments
    feeding the MIRIAM segmentation; explicitly required by the Matlab pipeline
  repo: https://www.ilastik.org/download.html
- name: Mesmer
  role: Deep-learning comparator segmentation method used to benchmark MIRIAM against
    manual annotations (DSC 0.59 ± 0.10 vs MIRIAM 0.65 ± 0.08)
- name: Matlab
  role: Original implementation language of MIRIAM (Matlab R2018b); Matlab and Python
    implementations show highly concordant segmentations (DSC 0.84 ± 0.06, JSC 0.74
    ± 0.09)
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative MIRIAM implementation (Python 3.7) validated as concordant with
    the Matlab implementation across 120 colon adenoma TMA cores
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_005
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:bfdac15a1bc8c8fe180926428de39cef2022d12cd99923e13968e49b1ac3cfd3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - Ilastik
  - MIRIAM
  - Matlab
  - Mesmer
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multiplexed-image-cell-segmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MIRIAM-style segmentation of single cells in multiplexed tissue imaging: random-forest (Ilastik) pixel classification to define nucleus/membrane/cytoplasm compartments, followed by detection and 'connects the dots' resegmentation of cells with internal aberrant membranes.

## When to use

Use when segmenting individual cells in multiplexed immunofluorescence (e.g., colorectal adenoma) or MIBI breast carcinoma tissue images where columnar epithelial cells have incomplete membranes and nuclear-dilation or Voronoi methods undersegment.

## When NOT to use

- The goal is comparison against deep-learning segmentation on non-epithelial tissues — Mesmer was trained on normal mouse small intestine, normal human colon, and colorectal cancer but not colorectal adenomas, so cross-method conclusions are tissue-specific
- Input images are single-channel or lack membrane markers (e.g., NaKATPase) — excluding membrane markers and DAPI from pixel classification decreased similarity to manually annotated borders
- The task is cell clustering or population discovery rather than segmentation — t-SNE shape analysis in this study did not reveal discernible cellular populations or clear patterns in marker expression

## Inputs

- Multiplexed immunofluorescence image stacks of tissue (e.g., 512x512 sub-images tiled from TMA cores)
- Multiplexed Ion Beam Imaging (MIBI) image stacks (e.g., breast carcinoma)
- Manually annotated cell-border / membrane ground-truth labels
- Random-forest pixel-classification probability maps (membrane/cytoplasm/nucleus) and watershed segmentation labels from Ilastik

## Outputs

- Per-cell segmentation label image (e.g., resegmented_labels.tif) with unique integer cell IDs
- Per-cell marker intensity quantification over whole cell, nucleus, membrane, and cytoplasm compartments
- Image-level and per-cell DSC/JSC similarity metrics versus manual annotations (e.g., segmentation_similarity_metrics.csv)

## How to apply

Start with random forest-based machine learning pixel classification (Ilastik, required by the Matlab pipeline) trained on manually annotated cell borders to assign pixels to nucleus, membrane, and cytoplasm; the full marker set should be used since dropping membrane markers or training on only NaKATPase+DAPI lowers agreement with manual annotations. Identify watershed-segmented cells containing internal aberrant membranes — flagged when internal membrane signal exceeds 10% of cell area or when the Mucin 2 channel shows intracellular expression. For flagged cells, compute internal membrane segment orientations and endpoints, then extend each segment ('connects the dots') until it hits the cell border or another extension, assigning unique integer IDs so separated cells stay distinct. Quantify each marker over the whole cell and in nucleus, membrane, and cytoplasm compartments, and optionally characterize cell shapes with an autoencoder neural network. Evaluate segmentation against manual annotations by dilating extracted borders with a 5-pixel square kernel and computing image-level and per-cell Dice (DSC) and Jaccard (JSC) similarity on 512x512 sub-images; MIRIAM's reported baseline on colorectal adenoma TMA cores is DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08.

## Related tools

- **MIRIAM** (Primary pipeline: pixel classification input, internal-membrane detection, 'connects the dots' resegmentation, and per-cell quantification) — github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Random-forest pixel classification of nucleus/membrane/cytoplasm compartments feeding the MIRIAM segmentation; explicitly required by the Matlab pipeline) — https://www.ilastik.org/download.html
- **Mesmer** (Deep-learning comparator segmentation method used to benchmark MIRIAM against manual annotations (DSC 0.59 ± 0.10 vs MIRIAM 0.65 ± 0.08))
- **Matlab** (Original implementation language of MIRIAM (Matlab R2018b); Matlab and Python implementations show highly concordant segmentations (DSC 0.84 ± 0.06, JSC 0.74 ± 0.09)) — github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative MIRIAM implementation (Python 3.7) validated as concordant with the Matlab implementation across 120 colon adenoma TMA cores) — github.com/Coffey-Lab/MIRIAM

## Evaluation signals

- Image-level DSC against manually annotated membranes should reach approximately 0.65 ± 0.08 and JSC 0.48 ± 0.08 on 240 colorectal adenoma sub-images from three TMA cores, exceeding Mesmer (DSC 0.59 ± 0.10) and Voronoi (DSC 0.32 ± 0.11)
- Matlab and Python MIRIAM implementations should agree with each other at image-level DSC 0.84 ± 0.06 and JSC 0.74 ± 0.09 over the 120 TMA cores
- All watershed-segmented cells with >10% internal membrane area (or intracellular Mucin 2 expression) should be resegmented, with internal membranes fully separated and every re-segmented cell holding a unique integer ID
- Ablation check: training pixel classification on the full marker set must outperform the no-membrane-marker and NaKATPase+DAPI-only permutations; single-marker exclusions should show minimal effect
- Per-cell DSC/JSC values and image-level aggregates (mean ± standard deviation) should be written to a metrics CSV covering all analyzed sub-images

## Limitations

- The Matlab implementation requires Ilastik for pixel classification, and example data excludes the large image stacks, which must be regenerated or downloaded separately (Dropbox link in README)
- No version numbers are stated for the Matlab or Python implementations beyond Matlab R2018b / Python 3.7 in the reproduction workflow
- MIRIAM's advantage over Mesmer is demonstrated on colorectal adenomas, to which Mesmer was not trained; performance on other tissues/platforms (e.g., breast cancer MIBI, Supplemental Figure 7) is shown qualitatively rather than benchmarked against annotations
- No accession or availability details are provided for the manually annotated ground-truth data or the breast cancer MIBI dataset

## Evidence
