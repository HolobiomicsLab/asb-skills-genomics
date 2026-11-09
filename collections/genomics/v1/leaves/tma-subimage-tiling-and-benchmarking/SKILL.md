---
name: tma-subimage-tiling-and-benchmarking
description: Use when manually annotated multiplexed tissue images (e.g., TMA cores)
  are available and you need to quantitatively compare cell-segmentation methods against
  ground-truth membranes using DSC/JSC.
when_to_use_negative:
- No manually annotated ground-truth membranes exist for the images — the skill requires
  annotated TMA cores to compute DSC/JSC (the article notes no accession or availability
  details for the manually annotated ground-truth data)
- The tissue type or segmentation goal differs from columnar epithelial membranes
  in multiplexed tissue images, e.g., Mesmer was not trained on colorectal adenomas
  (only mouse small intestine, normal human colon, and colorectal cancer), so benchmark
  transfer must be re-validated
- The input is already a per-cell feature/quantification table (nucleus, membrane,
  cytoplasm marker intensities) rather than image masks — segmentation benchmarking
  is then moot
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Primary segmentation pipeline under benchmark; run on each 512x512 sub-image
    to generate predicted cell masks for DSC/JSC scoring
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Mesmer
  role: Comparator deep-learning segmentation method scored against manual annotations
    alongside MIRIAM
- name: Voronoi
  role: Comparator tessellation-from-nuclei segmentation method; known to undersegment
    cells toward the lumen as it ignores cell membranes
- name: Matlab
  role: One of the two MIRIAM implementations used to produce predicted masks for
    the benchmark
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative MIRIAM implementation; Matlab and Python implementations show
    largely similar results
  repo: https://github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:aee94ee93e89edb51099dfa12b0d2f900e502d68ce93803bfd15222dd614b730
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
  - Mesmer
  - Python
  - Voronoi
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# tma-subimage-tiling-and-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A segmentation benchmarking skill that tiles manually annotated TMA cores into 512x512 sub-images, runs competing cell-segmentation pipelines (MIRIAM, Mesmer, Voronoi), and scores each against ground-truth membrane annotations with image-level and per-cell Dice (DSC) and Jaccard (JSC) similarity.

## When to use

Use when manually annotated multiplexed tissue images (e.g., TMA cores) are available and you need to quantitatively compare cell-segmentation methods against ground-truth membranes using DSC/JSC.

## When NOT to use

- No manually annotated ground-truth membranes exist for the images — the skill requires annotated TMA cores to compute DSC/JSC (the article notes no accession or availability details for the manually annotated ground-truth data)
- The tissue type or segmentation goal differs from columnar epithelial membranes in multiplexed tissue images, e.g., Mesmer was not trained on colorectal adenomas (only mouse small intestine, normal human colon, and colorectal cancer), so benchmark transfer must be re-validated
- The input is already a per-cell feature/quantification table (nucleus, membrane, cytoplasm marker intensities) rather than image masks — segmentation benchmarking is then moot

## Inputs

- Manually annotated TMA cores of precancerous colorectal adenomas (multiplexed immunofluorescence images with ground-truth membrane annotations)
- Predicted cell masks from MIRIAM (Matlab or Python implementation, github.com/Coffey-Lab/MIRIAM)
- Predicted cell masks from Mesmer deep-learning segmentation
- Predicted cell masks from Voronoi tessellation-from-nuclei segmentation

## Outputs

- Per-method image-level DSC and JSC aggregated as mean ± standard deviation across 240 sub-images (e.g., segmentation_similarity_metrics.csv)
- Per-cell DSC and JSC values for each segmentation method
- Dilated border masks (5-pixel square kernel) for predicted and ground-truth masks used in the similarity computation

## How to apply

Tile each manually annotated TMA core into 512x512 sub-images (240 sub-images were analyzed across the three annotated colorectal adenoma cores). Run each segmentation method on the identical sub-images — MIRIAM via its Matlab or Python implementation from github.com/Coffey-Lab/MIRIAM (which requires Ilastik for some tasks, e.g., random forest pixel classification), plus Mesmer deep-learning segmentation and Voronoi tessellation-from-nuclei as comparators. Extract cell borders from every predicted mask and dilate them with a 5-pixel square kernel, applying the identical border extraction and 5-pixel dilation to the manually annotated ground-truth masks so the comparison is symmetric. Compute DSC and JSC both at the whole sub-image level and per individual cell, then aggregate mean ± standard deviation across all 240 sub-images. Correctness is judged by reproducing the reported agreement (MIRIAM DSC 0.65±0.08 / JSC 0.48±0.08; Mesmer DSC 0.59±0.10 / JSC 0.42±0.08; Voronoi DSC 0.32±0.11 / JSC 0.20±0.07) and by confirming the ordering MIRIAM > Mesmer > Voronoi.

## Related tools

- **MIRIAM** (Primary segmentation pipeline under benchmark; run on each 512x512 sub-image to generate predicted cell masks for DSC/JSC scoring) — https://github.com/Coffey-Lab/MIRIAM
- **Mesmer** (Comparator deep-learning segmentation method scored against manual annotations alongside MIRIAM)
- **Voronoi** (Comparator tessellation-from-nuclei segmentation method; known to undersegment cells toward the lumen as it ignores cell membranes)
- **Matlab** (One of the two MIRIAM implementations used to produce predicted masks for the benchmark) — https://github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative MIRIAM implementation; Matlab and Python implementations show largely similar results) — https://github.com/Coffey-Lab/MIRIAM

## Evaluation signals

- Number of analyzed sub-images equals 240 across the three manually annotated TMA cores
- Image-level agreement reproduces reported values within one standard deviation: MIRIAM DSC 0.65±0.08 / JSC 0.48±0.08, Mesmer DSC 0.59±0.10 / JSC 0.42±0.08, Voronoi DSC 0.32±0.11 / JSC 0.20±0.07
- Method ordering invariant holds: MIRIAM exceeds both Mesmer and Voronoi on both DSC and JSC at image level
- Border dilation is symmetric: the same 5-pixel square kernel border extraction is applied to both predicted masks and manual ground-truth annotations
- Matlab and Python MIRIAM implementations yield largely similar segmentation results (per Supplemental Figure 6)

## Limitations

- Mesmer was trained on normal mouse small intestine, normal human colon, and colorectal cancer imaging data and was not trained on colorectal adenomas, which may bias the comparison in MIRIAM's favor
- Voronoi undersegments cells toward the lumen because it does not take cell membranes into account, so its low scores partly reflect a known structural limitation
- No accession ID or public availability statement is provided for the breast cancer MIBI dataset or the manually annotated ground-truth data, limiting reproducibility of the benchmark outside the original data
- No version numbers are stated for the Matlab or Python implementations, and no changelog exists for the software

## Evidence
