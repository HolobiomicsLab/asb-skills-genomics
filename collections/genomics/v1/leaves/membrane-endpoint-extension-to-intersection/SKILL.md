---
name: membrane-endpoint-extension-to-intersection
description: Use when watershed-segmented cells in multiplexed tissue images contain
  internal aberrant membranes (>10% of cell area) or internal Mucin 2 marker expression
  that causes multiple cells to be merged into one label.
when_to_use_negative:
- Cells without aberrant internal membranes (<10% internal membrane area and no internal
  Mucin 2 signal) — no resegmentation is needed
- When a fully deep-learning segmentation (e.g., Mesmer) is preferred instead of MIRIAM's
  random-forest plus resegmentation approach
- When input is already a per-cell quantification table rather than image data requiring
  segmentation
tools:
- name: MIRIAM
  role: Implements the 'connects the dots' internal membrane endpoint-extension resegmentation
    algorithm
  repo: github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: Random-forest pixel classification producing membrane/cytoplasm/nucleus probability
    maps that feed the resegmentation step
  repo: https://www.ilastik.org/download.html
- name: Matlab
  role: Primary implementation language for running MIRIAM cell segmentation on multiplexed
    imaging data
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative MIRIAM implementation with largely similar resegmentation results
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:6cce1e779f6ec72f49bd83fdd4cd80917dec47611d992587082197df750990d1
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# membrane-endpoint-extension-to-intersection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MIRIAM's 'connects the dots' resegmentation step: cells from watershed segmentation that contain internal aberrant membranes (>10% internal membrane by area, or internal Mucin 2 signal) are split by locating internal membrane segment endpoints and extending them along their orientation angles until they intersect the cell border or another extension.

## When to use

Use when watershed-segmented cells in multiplexed tissue images contain internal aberrant membranes (>10% of cell area) or internal Mucin 2 marker expression that causes multiple cells to be merged into one label.

## When NOT to use

- Cells without aberrant internal membranes (<10% internal membrane area and no internal Mucin 2 signal) — no resegmentation is needed
- When a fully deep-learning segmentation (e.g., Mesmer) is preferred instead of MIRIAM's random-forest plus resegmentation approach
- When input is already a per-cell quantification table rather than image data requiring segmentation

## Inputs

- Multiplexed immunofluorescence or MIBI image data (e.g., colorectal adenoma or breast carcinoma MIBI stacks)
- Watershed segmentation label image from the MIRIAM pipeline
- Random-forest pixel-classification probability maps for membrane, cytoplasm, and nucleus (Ilastik)
- Marker channels such as Mucin 2 used to detect internal membrane expression

## Outputs

- Resegmented label image (resegmented_labels.tif) with internal-membrane-containing cells split into distinct cells
- Per-cell integer IDs distinguishing cells separated by the extension step
- Verified separation of internal membranes from one another

## How to apply

First classify pixels with random-forest machine learning (e.g., Ilastik) into membrane/cytoplasm/nucleus probability maps, run watershed segmentation, then for each cell compute the fraction of internal membrane signal within the cell and flag it if that fraction exceeds 10% or if the Mucin 2 marker shows expression inside the cell. For each flagged cell, compute the orientation angles of the internal membrane segments and locate their endpoints. Extend each segment from its endpoints along its angle until it intersects either the cell border or another extended membrane segment — this is the 'connects the dots' extension that cuts merged epithelial cells apart. Assign a unique integer ID to every re-segmented cell so cells separated by the extension step remain distinct, and save the re-segmented label image as resegmented_labels.tif using the Matlab or Python MIRIAM implementation (results are largely similar between the two).

## Related tools

- **MIRIAM** (Implements the 'connects the dots' internal membrane endpoint-extension resegmentation algorithm) — github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Random-forest pixel classification producing membrane/cytoplasm/nucleus probability maps that feed the resegmentation step) — https://www.ilastik.org/download.html
- **Matlab** (Primary implementation language for running MIRIAM cell segmentation on multiplexed imaging data) — github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative MIRIAM implementation with largely similar resegmentation results) — github.com/Coffey-Lab/MIRIAM

## Evaluation signals

- All cells flagged with >10% internal membrane area (or internal Mucin 2 signal) undergo resegmentation — none are skipped
- Every extended membrane terminates at a cell border or another extension; no extension runs indefinitely
- Each re-segmented cell carries a unique integer ID and internal membranes are fully separated from one another
- Matlab and Python implementations of the extension produce largely similar segmentation results (Supplemental Figure 6)
- Output file resegmented_labels.tif is written with valid label geometry

## Limitations

- The pipeline requires Ilastik for pixel-classification tasks, adding an external dependency
- Example data omits the large image stacks, which must be regenerated by the user from the Dropbox link
- No version numbers are stated for the Matlab or Python implementations, complicating reproducibility

## Evidence
