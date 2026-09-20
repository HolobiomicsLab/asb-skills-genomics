---
name: unique-cell-id-labeling-after-resegmentation
description: Use when a watershed-segmented cell in multiplexed imaging data has been
  flagged (>10% internal membrane area, or internal Mucin 2 expression) and split
  by membrane extension, and each resulting fragment must carry a distinct integer
  label.
when_to_use_negative:
- Cells that do not exceed the 10% internal membrane area threshold and show no internal
  Mucin 2 expression — they were not flagged for resegmentation and should keep their
  original watershed label.
- Segmentation tasks where cells have regular morphologies and no incomplete/internal
  membranes — simpler dilation-from-nuclei or Voronoi approaches may suffice, per
  the paper's framing of MIRIAM for irregular columnar epithelial cells.
- When the input is already a per-cell quantification or feature table rather than
  a label image.
tools:
- name: MIRIAM
  role: Pipeline whose novel 'connects the dots' resegmentation step splits flagged
    cells and requires unique integer IDs for the resulting cells
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Implementation in which the resegmented label image with unique cell IDs is
    generated and saved
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative implementation for producing the re-segmented unique-ID label
    image
- name: Ilastik
  role: Provides the random-forest pixel-classification probability maps (membrane/cytoplasm/nucleus)
    that feed the watershed segmentation prior to resegmentation
  repo: https://www.ilastik.org/download.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:ec00eb537db2ad09cb2dc5775e759b692fe4b340b052bcc5e178221f1ff5b457
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# Unique cell ID labeling after resegmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns a unique integer ID to every MIRIAM-resegmented cell so that watershed-segmented cells split by the 'connects the dots' internal-membrane extension step remain distinct objects in the final label image (resegmented_labels.tif).

## When to use

Use when a watershed-segmented cell in multiplexed imaging data has been flagged (>10% internal membrane area, or internal Mucin 2 expression) and split by membrane extension, and each resulting fragment must carry a distinct integer label.

## When NOT to use

- Cells that do not exceed the 10% internal membrane area threshold and show no internal Mucin 2 expression — they were not flagged for resegmentation and should keep their original watershed label.
- Segmentation tasks where cells have regular morphologies and no incomplete/internal membranes — simpler dilation-from-nuclei or Voronoi approaches may suffice, per the paper's framing of MIRIAM for irregular columnar epithelial cells.
- When the input is already a per-cell quantification or feature table rather than a label image.

## Inputs

- Multiplexed image data (e.g., multiplexed immunofluorescence or MIBI image stacks)
- MIRIAM watershed segmentation label image
- ilastik random-forest pixel-classification probability maps for membrane, cytoplasm, and nucleus
- List of cells flagged as containing aberrant internal membranes (>10% internal membrane area, or internal Mucin 2 expression)

## Outputs

- resegmented_labels.tif — re-segmented label image with a unique integer ID per separated cell
- Verification that internal membranes are fully separated from one another

## How to apply

After MIRIAM's pixel-classification (ilastik probability maps for membrane/cytoplasm/nucleus) and watershed segmentation, compute for each cell the fraction of internal membrane signal and flag cells exceeding 10% internal membrane area, or showing Mucin 2 expression inside the cell. For each flagged cell, compute orientation angles of internal membrane segments, locate their endpoints, and extend them along those angles until they intersect the cell border or another extended segment ('connects the dots'). Then assign a unique integer ID to every re-segmented cell so fragments separated by the extension step remain distinct, and verify internal membranes are fully separated from one another before saving the label image as resegmented_labels.tif with the Python or Matlab MIRIAM implementation.

## Related tools

- **MIRIAM** (Pipeline whose novel 'connects the dots' resegmentation step splits flagged cells and requires unique integer IDs for the resulting cells) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Implementation in which the resegmented label image with unique cell IDs is generated and saved) — github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative implementation for producing the re-segmented unique-ID label image)
- **Ilastik** (Provides the random-forest pixel-classification probability maps (membrane/cytoplasm/nucleus) that feed the watershed segmentation prior to resegmentation) — https://www.ilastik.org/download.html

## Evaluation signals

- Every re-segmented cell in resegmented_labels.tif carries a unique integer ID; no two separated fragments share a label.
- Cells originally flagged (>10% internal membrane area or internal Mucin 2 expression) are split into distinct labels at each internal membrane.
- Internal membranes are fully separated from one another after the extension step.
- Extended membrane segments terminate only at the cell border or at intersections with other extended segments, not mid-cytoplasm.
- Matlab and Python implementations yield largely similar results (per Supplemental Figure 6 comparison).

## Limitations

- The flagging threshold is empirical (>10% internal membrane by area, or Mucin 2 expression inside the cell); cells just below the cutoff with aberrant membranes may be missed.
- Extension stops at intersections with other extended segments, so ambiguous crossing membranes can produce arbitrary splits.
- Example data in the repository omits image stacks (too large), which must be regenerated by the user from the Dropbox-hosted files.
- No version numbers are stated for the Matlab or Python implementations.

## Evidence

- [full_text] Cells containing greater than 10% internal membranes by area are resegmented using a novel 'connects the dots' algorithm that finds the endpoints of internal membranes and extends them (until intersection with the cell border or another extension).: [evidence span withheld — anchor status: not_found]
- [full_text] Assign a unique integer ID to every re-segmented cell so that cells separated by the extension step remain distinct, and verify that internal membranes are fully separated from one another.: [evidence span withheld — anchor status: not_found]
