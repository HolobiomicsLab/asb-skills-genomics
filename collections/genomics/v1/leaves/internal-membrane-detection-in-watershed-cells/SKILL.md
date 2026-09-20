---
name: internal-membrane-detection-in-watershed-cells
description: Use when watershed segmentation of multiplexed tissue images merges adjacent
  columnar epithelial cells because incomplete internal membranes were not resolved,
  and per-cell membrane/cytoplasm/nucleus probability maps from ilastik are available.
when_to_use_negative:
- Cells have no internal membrane-class signal (<10% of cell area) and no internal
  Mucin 2 expression — the resegmentation step only acts on flagged cells
- The segmentation is being performed by a non-membrane-aware method (e.g., Voronoi
  tessellation) rather than MIRIAM's watershed labels — the extension algorithm operates
  on MIRIAM's own watershed output and probability maps
- Input lacks multi-marker multiplexed image data needed for ilastik pixel classification
  of membrane/cytoplasm/nucleus compartments
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Provides the watershed segmentation, the >10% internal-membrane flagging rule,
    and the 'connects the dots' endpoint-extension resegmentation algorithm; Matlab
    and Python implementations both save resegmented_labels.tif
  repo: github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: Random-forest pixel classification producing membrane/cytoplasm/nucleus probability
    maps that supply the internal-membrane signal used for flagging and extension
  repo: https://www.ilastik.org/download.html
- name: Matlab
  role: One of the two reference implementations of the MIRIAM resegmentation pipeline
    (results largely similar to Python per Supplemental Figure 6)
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative implementation of MIRIAM resegmentation for saving resegmented_labels.tif
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:cb75eb791916599704996592a02e12ede2c5df960497bbc9d6c75b2fecd13ff2
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
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# internal-membrane-detection-in-watershed-cells

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MIRIAM's resegmentation step detects watershed-segmented cells that contain aberrant internal membranes (membrane-class signal inside the cell exceeding 10% of cell area, or internal Mucin 2 expression) and splits them by extending internal membrane segments via a 'connects the dots' endpoint-extension algorithm. This recovers single-cell boundaries that standard watershed under-segments in columnar epithelial tissue.

## When to use

Use when watershed segmentation of multiplexed tissue images merges adjacent columnar epithelial cells because incomplete internal membranes were not resolved, and per-cell membrane/cytoplasm/nucleus probability maps from ilastik are available.

## When NOT to use

- Cells have no internal membrane-class signal (<10% of cell area) and no internal Mucin 2 expression — the resegmentation step only acts on flagged cells
- The segmentation is being performed by a non-membrane-aware method (e.g., Voronoi tessellation) rather than MIRIAM's watershed labels — the extension algorithm operates on MIRIAM's own watershed output and probability maps
- Input lacks multi-marker multiplexed image data needed for ilastik pixel classification of membrane/cytoplasm/nucleus compartments

## Inputs

- Multiplexed immunofluorescence or MIBI image stack (e.g., colorectal adenoma or breast carcinoma MIBI data)
- MIRIAM watershed segmentation label image
- ilastik random-forest pixel-classification probability maps for membrane, cytoplasm, and nucleus classes
- Mucin 2 marker channel image (used as an alternative internal-membrane trigger)

## Outputs

- resegmented_labels.tif — label image with cells containing internal aberrant membranes split into distinct cells
- Per-cell flag of internal membrane fraction (>10% threshold) and unique integer cell IDs post-resegmentation

## How to apply

Load the multiplexed image, MIRIAM watershed segmentation labels, and ilastik random-forest pixel-classification probability maps for membrane/cytoplasm/nucleus from github.com/Coffey-Lab/MIRIAM. For each watershed cell, compute the fraction of internal membrane-class signal within the cell area and flag it as aberrant if that fraction exceeds 10%, or if the Mucin 2 marker channel shows internal expression. For each flagged cell, compute the orientation angle of each internal membrane segment, locate its endpoints, and extend each segment along its angle until it intersects the cell border or another extension line — the 'connects the dots' algorithm. Assign a unique integer ID to every re-segmented cell so cells separated by extension remain distinct, then save the result as resegmented_labels.tif using the Matlab or Python MIRIAM implementation (both give largely similar results per Supplemental Figure 6).

## Related tools

- **MIRIAM** (Provides the watershed segmentation, the >10% internal-membrane flagging rule, and the 'connects the dots' endpoint-extension resegmentation algorithm; Matlab and Python implementations both save resegmented_labels.tif) — github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Random-forest pixel classification producing membrane/cytoplasm/nucleus probability maps that supply the internal-membrane signal used for flagging and extension) — https://www.ilastik.org/download.html
- **Matlab** (One of the two reference implementations of the MIRIAM resegmentation pipeline (results largely similar to Python per Supplemental Figure 6)) — github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative implementation of MIRIAM resegmentation for saving resegmented_labels.tif) — github.com/Coffey-Lab/MIRIAM

## Evaluation signals

- Every watershed cell with internal membrane fraction >10% of cell area (or internal Mucin 2 expression) is flagged and resegmented
- After extension, internal membranes within each formerly merged cell are fully separated from one another (no residual connecting regions)
- Every re-segmented cell carries a unique integer ID, with no duplicate labels among cells separated by the extension step
- The output resegmented_labels.tif has the same dimensions as the input watershed label image
- Matlab and Python implementations should yield largely similar resegmentation results (per Supplemental Figure 6 comparison)

## Limitations

- Example data in the repository omits the image stacks (too large); they must be regenerated or downloaded from the linked Dropbox share
- The pipeline requires ilastik for pixel-classification tasks, adding a dependency before resegmentation can run
- The 10% internal-membrane-area threshold and the Mucin 2 internal-expression trigger are the only flagging criteria; cells merged for other reasons will not be split
- MIRIAM was not evaluated on all platforms — it was applied to multiplexed immunofluorescence of colorectal adenomas and a MIBI breast carcinoma dataset; performance elsewhere is untested here

## Evidence

- [other] Cells containing greater than 10% internal membranes by area are resegmented using a novel 'connects the dots' algorithm that finds the endpoints of internal membranes and extends them (until intersection with the cell border or another extension).: [evidence span withheld — anchor status: not_found]
