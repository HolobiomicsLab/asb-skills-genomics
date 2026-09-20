---
name: mucin2-marker-based-goblet-cell-identification
description: Use when segmenting multiplexed tissue images containing goblet/mucin-producing
  epithelial cells whose internal mucin membranes break the single-cell segmentation,
  and Mucin 2 is one of the marker channels.
when_to_use_negative:
- The dataset lacks a Mucin 2 (or equivalent mucin) marker channel, so intracellular
  goblet-cell expression cannot be assessed — fall back on the >10% internal-membrane-area
  rule alone
- Cells are morphologically regular with complete membranes and no internal membrane
  signal, so no resegmentation trigger is needed
edam_operation: http://edamontology.org/operation_3443
tools:
- name: MIRIAM
  role: Provides the pipeline in which the Mucin 2 internal-expression check flags
    cells for the 'connects the dots' membrane-extension resegmentation
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: 'Matlab implementation used to run the MIRIAM segmentation and resegmentation
    code (README: ''Matlab code to run cell segmentation on multiplexed imaging data'')'
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative implementation of the MIRIAM pipeline yielding largely similar
    resegmentation results
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:20dd4dd2d72c870c9c645a1cb948b2b3e085fcb4066961295bac384351fd05db
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mucin2-marker-based-goblet-cell-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A MIRIAM-specific rule for flagging cells (e.g., mucin-containing goblet cells) as harboring internal aberrant membranes when the Mucin 2 marker channel shows intracellular expression, triggering the 'connects the dots' membrane-extension resegmentation step.

## When to use

Use when segmenting multiplexed tissue images containing goblet/mucin-producing epithelial cells whose internal mucin membranes break the single-cell segmentation, and Mucin 2 is one of the marker channels.

## When NOT to use

- The dataset lacks a Mucin 2 (or equivalent mucin) marker channel, so intracellular goblet-cell expression cannot be assessed — fall back on the >10% internal-membrane-area rule alone
- Cells are morphologically regular with complete membranes and no internal membrane signal, so no resegmentation trigger is needed

## Inputs

- MIRIAM watershed segmentation label image for the multiplexed image
- Random-forest (ilastik) pixel-classification probability maps for membrane/cytoplasm/nucleus
- Multiplexed immunofluorescence image stack including the Mucin 2 marker channel

## Outputs

- Set of cells flagged as containing internal aberrant membranes (via >10% internal membrane area or intracellular Mucin 2 expression)
- Resegmented label image with internal membranes separated (e.g., resegmented_labels.tif) and unique integer IDs per re-segmented cell

## How to apply

After MIRIAM's random-forest pixel classification produces membrane/cytoplasm/nucleus probability maps and watershed segmentation labels are generated, compute the fraction of internal membrane signal within each watershed-segmented cell. Flag a cell as containing an aberrant internal membrane if that fraction exceeds 10% of the cell area OR if the Mucin 2 marker channel shows expression inside the cell. The Mucin 2 rule captures goblet cells whose mucin granules register as internal membranes even when the area fraction is below threshold. Each flagged cell is then processed by the 'connects the dots' resegmentation: internal membrane segment endpoints are located and extended along their orientation angles until they intersect the cell border or another extension, assigning unique integer IDs to the separated daughter cells.

## Related tools

- **MIRIAM** (Provides the pipeline in which the Mucin 2 internal-expression check flags cells for the 'connects the dots' membrane-extension resegmentation) — https://github.com/Coffey-Lab/MIRIAM
- **Matlab** (Matlab implementation used to run the MIRIAM segmentation and resegmentation code (README: 'Matlab code to run cell segmentation on multiplexed imaging data')) — https://github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative implementation of the MIRIAM pipeline yielding largely similar resegmentation results)

## Evaluation signals

- Every cell with intracellular Mucin 2 expression is included in the flagged set for resegmentation
- All cells exceeding 10% internal membrane area are flagged, and the two criteria (area fraction, Mucin 2) are applied as an OR
- After extension, internal membranes within each flagged cell are fully separated from one another and no two re-segmented cells share an integer ID
- Matlab and Python implementations produce largely similar resegmentation results (per Supplemental Figure 6)

## Limitations

- The Mucin 2 rule is marker-dependent; datasets without a mucin channel or with non-goblet internal membranes must rely solely on the 10% internal-membrane-area threshold
- Example data omits the raw image stacks (too large), so users must regenerate/download them separately from the Dropbox link in the README
- The pipeline requires ilastik for pixel-classification tasks, adding a dependency

## Evidence

- [other] Internal-membrane flagging rule: [evidence span withheld — anchor status: not_found]
- [other] Resegmentation method: [evidence span withheld — anchor status: not_found]
