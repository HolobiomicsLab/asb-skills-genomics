---
name: segmentation-mask-border-extraction-and-dilation
description: Use when comparing predicted cell segmentation masks (e.g., MIRIAM, Mesmer,
  Voronoi outputs) against manually annotated membranes on multiplexed tissue images
  and you need a border-based DSC/JSC agreement metric rather than whole-mask overlap.
when_to_use_negative:
- The input is raw multiplexed image data and no segmentation masks exist yet — run
  a segmentation pipeline (e.g., MIRIAM) first.
- You need whole-cell overlap rather than membrane-border placement fidelity — border
  dilation with a 5-pixel kernel specifically scores boundary agreement.
- Segmentation masks come from different voxel/pixel scales or are not co-registered
  to the ground-truth annotation — DSC/JSC on dilated borders is only meaningful on
  aligned masks.
tools:
- name: MIRIAM
  role: Source of predicted cell masks whose borders are compared against manual annotations;
    has Matlab and Python implementations
  repo: github.com/Coffey-Lab/MIRIAM
- name: Mesmer
  role: Deep-learning comparator segmentation whose output masks undergo the same
    border extraction and dilation before DSC/JSC scoring
- name: Matlab
  role: Implementation language of the MIRIAM pipeline used to generate and process
    the masks
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative MIRIAM implementation producing largely similar segmentation results
    that can feed this comparison workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:0df1556264dcfdfc44a75c69d4d8703914de23825cae6d244044739b710c21d4
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
  - Mesmer
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# segmentation-mask-border-extraction-and-dilation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A mask-comparison preprocessing skill from the MIRIAM paper: extract cell borders from predicted and ground-truth segmentation masks, dilate both with the same 5-pixel square kernel, and compute image-level Dice (DSC) and Jaccard (JSC) similarity to quantify agreement between segmentation methods and manually annotated membranes.

## When to use

Use when comparing predicted cell segmentation masks (e.g., MIRIAM, Mesmer, Voronoi outputs) against manually annotated membranes on multiplexed tissue images and you need a border-based DSC/JSC agreement metric rather than whole-mask overlap.

## When NOT to use

- The input is raw multiplexed image data and no segmentation masks exist yet — run a segmentation pipeline (e.g., MIRIAM) first.
- You need whole-cell overlap rather than membrane-border placement fidelity — border dilation with a 5-pixel kernel specifically scores boundary agreement.
- Segmentation masks come from different voxel/pixel scales or are not co-registered to the ground-truth annotation — DSC/JSC on dilated borders is only meaningful on aligned masks.

## Inputs

- Predicted cell segmentation masks (MIRIAM, Mesmer, or Voronoi outputs) per sub-image
- Manually annotated ground-truth membrane masks
- 512x512 tiled sub-images from manually annotated TMA cores

## Outputs

- Image-level DSC and JSC per sub-image and segmentation method
- Per-cell DSC and JSC values
- Aggregated means and standard deviations across sub-images (e.g., a segmentation_similarity_metrics.csv)

## How to apply

Take each predicted cell mask (from MIRIAM's Matlab or Python implementation, Mesmer, or Voronoi tessellation-from-nuclei) and the manually annotated ground-truth mask, and extract cell borders from both. Dilate every extracted border with an identical 5-pixel square kernel so that slight boundary misalignments are not over-penalized — the same extraction and dilation must be applied to predicted and ground-truth masks symmetrically. Compute the Dice similarity coefficient (DSC) and Jaccard similarity coefficient (JSC) on the dilated borders at the whole sub-image level and per individual cell. The paper applies this on 512x512 sub-images tiled from three manually annotated colorectal adenoma TMA cores (240 sub-images total), then aggregates means and standard deviations across sub-images (e.g., MIRIAM image-level DSC 0.65 ± 0.08, JSC 0.48 ± 0.08). The rationale is that border dilation measures whether methods place membranes in the right locations, which is the discriminating failure mode for Voronoi, which undersegments cells toward the lumen because it ignores cell membranes.

## Related tools

- **MIRIAM** (Source of predicted cell masks whose borders are compared against manual annotations; has Matlab and Python implementations) — github.com/Coffey-Lab/MIRIAM
- **Mesmer** (Deep-learning comparator segmentation whose output masks undergo the same border extraction and dilation before DSC/JSC scoring)
- **Matlab** (Implementation language of the MIRIAM pipeline used to generate and process the masks) — github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative MIRIAM implementation producing largely similar segmentation results that can feed this comparison workflow)

## Evaluation signals

- The same border-extraction and 5-pixel square-kernel dilation is applied identically to both predicted and ground-truth masks (symmetric preprocessing).
- Recomputed image-level agreement falls within the reported ranges: MIRIAM DSC 0.65 ± 0.08 / JSC 0.48 ± 0.08, Mesmer DSC 0.59 ± 0.10 / JSC 0.42 ± 0.08, Voronoi DSC 0.32 ± 0.11 / JSC 0.20 ± 0.07.
- MIRIAM's DSC and JSC exceed both Mesmer's and Voronoi's on the same sub-images.
- The analysis covers all 240 sub-images tiled from the three manually annotated TMA cores.
- Both image-level and per-cell DSC/JSC values are produced, not just one granularity.

## Limitations

- Voronoi segmentation undersegments cells toward the lumen because it does not take into account cell membranes, so border-based scores will be low for such methods by design.
- Quantitative similarity scores are referenced mainly via figure pointers; no accession or availability details for the manually annotated ground-truth data are provided, complicating reproduction.
- The MIBI breast carcinoma dataset used elsewhere in the paper has a different marker set and lower image quality than the colorectal adenoma data, so thresholds tuned on adenoma TMA cores may not transfer.

## Evidence
