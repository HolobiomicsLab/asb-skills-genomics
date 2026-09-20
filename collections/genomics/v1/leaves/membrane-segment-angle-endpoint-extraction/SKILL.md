---
name: membrane-segment-angle-endpoint-extraction
description: Use when processing watershed-segmented multiplexed tissue images where
  cells may be erroneously merged because internal aberrant membranes were not used
  as segmentation boundaries (e.g., columnar epithelial cells in colorectal adenoma
  images).
when_to_use_negative:
- Cells whose membranes are already complete and contiguous — no internal membranes
  will exceed the 10% area threshold, so resegmentation is unnecessary
- When the goal is whole-cell segmentation from nuclei alone without membrane information;
  Voronoi/dilation approaches, not membrane extension, apply to that setting
- Input is a feature/quantification table rather than a segmented multiplexed image;
  this skill operates on label images and pixel-classification probability maps
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Pipeline providing the watershed segmentation and the 'connects the dots'
    internal-membrane resegmentation algorithm implemented here
  repo: github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: Random-forest pixel classification producing membrane/cytoplasm/nucleus probability
    maps used to quantify internal membrane fraction
  repo: https://www.ilastik.org/download.html
- name: Matlab
  role: Primary implementation language for the MIRIAM resegmentation code
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative implementation of MIRIAM with largely similar results
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:c2ee793e0d914967aa71d7a8e8932d983f51f121e6e3b675612a87ba58171393
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
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# membrane-segment-angle-endpoint-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements MIRIAM's 'connects the dots' internal-membrane resegmentation step, which detects watershed-segmented cells containing aberrant internal membranes, extracts each membrane segment's orientation angle and endpoints, and extends segments until they intersect the cell border or another extension to split incorrectly merged cells.

## When to use

Use when processing watershed-segmented multiplexed tissue images where cells may be erroneously merged because internal aberrant membranes were not used as segmentation boundaries (e.g., columnar epithelial cells in colorectal adenoma images).

## When NOT to use

- Cells whose membranes are already complete and contiguous — no internal membranes will exceed the 10% area threshold, so resegmentation is unnecessary
- When the goal is whole-cell segmentation from nuclei alone without membrane information; Voronoi/dilation approaches, not membrane extension, apply to that setting
- Input is a feature/quantification table rather than a segmented multiplexed image; this skill operates on label images and pixel-classification probability maps

## Inputs

- Watershed segmentation label image of cells (MIRIAM intermediate output)
- Random-forest (Ilastik) pixel-classification probability maps for membrane/cytoplasm/nucleus
- Multiplexed immunofluorescence image stack (e.g., colorectal adenoma or MIBI breast carcinoma channels), including the Mucin 2 marker channel

## Outputs

- resegmented_labels.tif — label image with internal-membrane-containing cells split into distinct cells with unique integer IDs
- Per-cell flag (fraction of internal membrane area > 10% or Mucin 2 expression) identifying aberrant-membrane cells
- Per-segment orientation angles and endpoint coordinates used for membrane extension

## How to apply

Start from the MIRIAM watershed segmentation labels plus random-forest pixel-classification probability maps for membrane, cytoplasm, and nucleus compartments. For each watershed-segmented cell, compute the fraction of internal membrane signal by area and flag the cell if that fraction exceeds 10%, or if the Mucin 2 marker channel shows expression inside the cell. For each flagged cell, compute the orientation angles of the internal membrane segments and locate their endpoints, then extend each segment from its endpoints along its angle until it intersects either the cell border or another extended segment — this 'connects the dots' step converts incomplete internal membranes into complete separating boundaries. Assign a unique integer ID to every re-segmented cell so cells split by the extension remain distinct. The rationale is that dilation/Voronoi-style methods ignore membranes and undersegment cells toward the lumen, while extending membrane segments recovers true epithelial cell boundaries. Save the result as resegmented_labels.tif using the Matlab or Python MIRIAM implementation (the Matlab pipeline additionally requires Ilastik for pixel classification).

## Related tools

- **MIRIAM** (Pipeline providing the watershed segmentation and the 'connects the dots' internal-membrane resegmentation algorithm implemented here) — github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Random-forest pixel classification producing membrane/cytoplasm/nucleus probability maps used to quantify internal membrane fraction) — https://www.ilastik.org/download.html
- **Matlab** (Primary implementation language for the MIRIAM resegmentation code) — github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative implementation of MIRIAM with largely similar results)

## Evaluation signals

- Every re-segmented cell in resegmented_labels.tif carries a unique integer ID and cells separated by the extension step remain distinct
- After extension, internal membranes are fully separated from one another (no unextended endpoints left inside a cell)
- Only cells with >10% internal membrane area (or Mucin 2 expression inside the cell) are resegmented; all others pass through unchanged
- Extended membrane segments terminate exactly at the cell border or at an intersection with another extended segment
- Matlab and Python implementations should yield largely similar results (per Supplemental Figure 6 comparison)

## Limitations

- The 10% internal-membrane area threshold and Mucin 2 trigger were developed/evaluated on colorectal adenoma and MIBI breast carcinoma data; performance on other tissue types or platforms is not guaranteed
- The extension heuristic assumes internal membrane segments can be completed by straight-line extension along their orientation angle; highly curved or branched membranes may not connect correctly
- No version numbers are stated for the Matlab or Python implementations, and no changelog is provided in the repository

## Evidence

- [other] Cells containing greater than 10% internal membranes by area are resegmented using a novel 'connects the dots' algorithm that finds the endpoints of internal membranes and extends them (until intersection with the cell border or another extension).: [evidence span withheld — anchor status: not_found]
