---
name: tissue-microarray-image-batch-processing
description: Use when you need to execute the MIRIAM cell-segmentation pipeline over
  many TMA cores across two software implementations and quantify cross-implementation
  segmentation agreement per image.
when_to_use_negative:
- You are comparing MIRIAM against alternative segmentation methods (Mesmer, Voronoi)
  against manual annotation — that benchmark uses 240 sub-images across three manually
  annotated TMA cores with different reported DSC/JSC values (MIRIAM DSC 0.65 ± 0.08),
  not the Matlab-vs-Python comparison.
- Your images are not multiplexed tissue images (e.g., single-channel microscopy or
  non-TMA whole-slide data) — the pipeline and its agreement thresholds were established
  on colon adenoma TMA cores and a breast carcinoma MIBI dataset.
- You only need marker quantification or t-SNE shape analysis downstream and do not
  intend to verify cross-implementation segmentation consistency.
edam_operation: http://edamontology.org/operation_3553
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Cell segmentation and quantification pipeline (random-forest pixel classification
    plus internal-membrane-based cell separation) executed per TMA core in both implementations
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: R2018b execution environment for the original MIRIAM implementation producing
    reference segmentation masks
- name: Python
  role: Python 3.7 execution environment for the second MIRIAM implementation producing
    comparison segmentation masks
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:6194d7e69fed2db6f3a00fa64245608b4b616d92f1c3f2c5e2b374353983ba51
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
  edam_operation: http://edamontology.org/operation_3553
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# tissue-microarray-image-batch-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Batch-runs the MIRIAM segmentation pipeline (random-forest pixel classification plus internal-membrane-based cell separation) over tissue microarray cores in both Matlab (R2018b) and Python (3.7) environments, then compares the two implementations per image using cell-membrane Dice (DSC) and Jaccard (JSC) similarity coefficients.

## When to use

Use when you need to execute the MIRIAM cell-segmentation pipeline over many TMA cores across two software implementations and quantify cross-implementation segmentation agreement per image.

## When NOT to use

- You are comparing MIRIAM against alternative segmentation methods (Mesmer, Voronoi) against manual annotation — that benchmark uses 240 sub-images across three manually annotated TMA cores with different reported DSC/JSC values (MIRIAM DSC 0.65 ± 0.08), not the Matlab-vs-Python comparison.
- Your images are not multiplexed tissue images (e.g., single-channel microscopy or non-TMA whole-slide data) — the pipeline and its agreement thresholds were established on colon adenoma TMA cores and a breast carcinoma MIBI dataset.
- You only need marker quantification or t-SNE shape analysis downstream and do not intend to verify cross-implementation segmentation consistency.

## Inputs

- 120 colon adenoma TMA core multiplexed immunofluorescence image stacks
- MIRIAM pipeline from github.com/Coffey-Lab/MIRIAM (Matlab and Python implementations)
- Ilastik installation required by the Matlab pipeline for classification tasks

## Outputs

- Per-core cell/membrane segmentation masks from the Matlab implementation
- Per-core cell/membrane segmentation masks from the Python implementation
- Per-image DSC and JSC cross-implementation metrics
- implementation_comparison_metrics.csv with per-image and mean ± standard deviation summary metrics

## How to apply

Obtain MIRIAM from the Coffey-Lab GitHub repository and set up both the Matlab (R2018b) and Python (3.7) execution environments; the Matlab pipeline additionally requires Ilastik for pixel-classification tasks, as stated in the README. Run the Matlab implementation of MIRIAM (random-forest pixel classification followed by internal-membrane-based cell separation) on each of the 120 colon adenoma TMA cores, then run the Python implementation with identical pipeline settings on the same cores to produce paired cell/membrane segmentation masks. For every image, compute the image-level cell-membrane Dice similarity coefficient and Jaccard similarity coefficient between the Matlab-derived and Python-derived masks. Aggregate per-image DSC and JSC into mean ± standard deviation across the 120 cores and write per-image and summary metrics to implementation_comparison_metrics.csv. Accept the reproduction if mean image-level DSC falls within 0.84 ± 0.06 and mean JSC within 0.74 ± 0.09, matching the cross-implementation agreement reported in Supplemental Figure 6.

## Related tools

- **MIRIAM** (Cell segmentation and quantification pipeline (random-forest pixel classification plus internal-membrane-based cell separation) executed per TMA core in both implementations) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (R2018b execution environment for the original MIRIAM implementation producing reference segmentation masks)
- **Python** (Python 3.7 execution environment for the second MIRIAM implementation producing comparison segmentation masks)

## Evaluation signals

- Mean image-level DSC across the 120 cores falls within 0.84 ± 0.06
- Mean image-level JSC across the 120 cores falls within 0.74 ± 0.09
- Per-image DSC and JSC are computed for all 120 cores with both Matlab and Python masks present for each image
- implementation_comparison_metrics.csv contains both per-image rows and a mean ± standard deviation summary
- Both runs used identical MIRIAM pipeline settings (random-forest pixel classification parameters unchanged between implementations)

## Limitations

- Reported DSC/JSC values correspond to agreement between the two MIRIAM implementations, not to accuracy against manual ground truth.
- Image stacks are not included in the repository example data (too large) and must be regenerated by the user.
- The Matlab pipeline requires Ilastik to be installed separately for some tasks; version numbers for the Matlab or Python implementations are not stated in the article.
- Agreement thresholds were established on colon adenoma TMA cores; other tissue types (e.g., the breast carcinoma MIBI dataset) may behave differently.

## Evidence
