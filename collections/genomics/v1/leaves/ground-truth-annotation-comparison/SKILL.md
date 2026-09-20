---
name: ground-truth-annotation-comparison
description: Use when you need to quantitatively compare predicted cell segmentation
  masks against manually annotated ground-truth membranes, especially for tissue segmentation
  methods on multiplexed imaging (TMA cores, MIBI, IMC).
when_to_use_negative:
- When ground-truth manual annotations do not exist — DSC/JSC against manual membranes
  cannot be computed and only qualitative comparison (e.g., Fig. 2G-style overlay
  inspection) is possible
- When the target cells are round and non-epithelial — the paper shows Voronoi-style
  dilation-from-nuclei is inadequate mainly for columnar epithelial cells with elongated,
  irregular morphologies, so a full border-dilation benchmark may add little
- When the segmentation method was trained on the annotated tissue itself — the comparison
  would be circular; note Mesmer was trained on normal mouse small intestine, normal
  human colon, and colorectal cancer, but not colorectal adenomas
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Primary segmentation method under evaluation; provides predicted cell masks
    whose borders are compared to manual annotations
  repo: github.com/Coffey-Lab/MIRIAM
- name: Mesmer
  role: Deep-learning comparator segmentation method benchmarked against MIRIAM using
    DSC/JSC
- name: Python
  role: Implementation language for the MIRIAM pipeline and metric computation; results
    largely similar to Matlab implementation
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Reference implementation language for running the MIRIAM pipeline on sub-images
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:1b54c7c855a5a94e6fb283a4ed4c8de050e09afcdb104f3e5db9fe38f1d9cfe8
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
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ground-truth-annotation-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks segmentation outputs (MIRIAM, Mesmer, Voronoi) against manually annotated cell membranes by extracting cell borders, dilating them with a 5-pixel square kernel, and computing image-level and per-cell Dice (DSC) and Jaccard (JSC) similarity coefficients.

## When to use

Use when you need to quantitatively compare predicted cell segmentation masks against manually annotated ground-truth membranes, especially for tissue segmentation methods on multiplexed imaging (TMA cores, MIBI, IMC).

## When NOT to use

- When ground-truth manual annotations do not exist — DSC/JSC against manual membranes cannot be computed and only qualitative comparison (e.g., Fig. 2G-style overlay inspection) is possible
- When the target cells are round and non-epithelial — the paper shows Voronoi-style dilation-from-nuclei is inadequate mainly for columnar epithelial cells with elongated, irregular morphologies, so a full border-dilation benchmark may add little
- When the segmentation method was trained on the annotated tissue itself — the comparison would be circular; note Mesmer was trained on normal mouse small intestine, normal human colon, and colorectal cancer, but not colorectal adenomas

## Inputs

- Manually annotated cell membrane ground-truth masks for TMA cores (e.g., three annotated colorectal adenoma TMA cores)
- Predicted cell segmentation masks from each comparator method (MIRIAM, Mesmer, Voronoi), generated per 512x512 sub-image
- Tiled 512x512 sub-images of the original multiplexed immunofluorescence tissue images

## Outputs

- Image-level DSC and JSC values (mean ± standard deviation) per segmentation method, e.g., segmentation_similarity_metrics.csv
- Per-cell DSC and JSC values for each method across the analyzed sub-images
- Method ranking against ground truth (e.g., MIRIAM > Mesmer > Voronoi)

## How to apply

Tile each manually annotated TMA core into 512x512 sub-images (the MIRIAM paper analyzed 240 such sub-images across three cores) and run each segmentation method (e.g., MIRIAM from github.com/Coffey-Lab/MIRIAM, Mesmer, Voronoi tessellation-from-nuclei) on the same sub-images. Extract cell borders from each predicted mask and from the manual ground-truth masks, then dilate all borders with a 5-pixel square kernel so thin membrane disagreements do not dominate the score. Compute Dice similarity coefficient (DSC) and Jaccard similarity coefficient (JSC) at the whole sub-image level and per individual cell, then aggregate mean ± standard deviation across sub-images. Judge success by comparing to reference values: for colorectal adenoma TMAs, MIRIAM DSC 0.65±0.08 / JSC 0.48±0.08 should exceed Mesmer (0.59±0.10 / 0.42±0.08) and Voronoi (0.32±0.11 / 0.20±0.07). Rationale: dilation-based border comparison fairly scores methods like MIRIAM that extend incomplete membranes, whereas direct pixel agreement penalizes columnar epithelial cells with elongated morphologies.

## Related tools

- **MIRIAM** (Primary segmentation method under evaluation; provides predicted cell masks whose borders are compared to manual annotations) — github.com/Coffey-Lab/MIRIAM
- **Mesmer** (Deep-learning comparator segmentation method benchmarked against MIRIAM using DSC/JSC)
- **Python** (Implementation language for the MIRIAM pipeline and metric computation; results largely similar to Matlab implementation) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Reference implementation language for running the MIRIAM pipeline on sub-images) — github.com/Coffey-Lab/MIRIAM

## Examples

```
matlab -r "run_miriam_segmentation(subimages_512x512, 'core1','core2','core3'); compute_dsc_jsc(pred_masks, manual_masks, 'dilation_kernel',5, 'out','segmentation_similarity_metrics.csv')"
```

## Evaluation signals

- Image-level DSC for the method under test falls within the reported mean ± SD (e.g., MIRIAM 0.65±0.08, Mesmer 0.59±0.10, Voronoi 0.32±0.11 on colorectal adenoma TMAs)
- JSC values are consistently lower than DSC (JSC = DSC/(2−DSC)); e.g., MIRIAM JSC 0.48±0.08 vs DSC 0.65±0.08
- The analyzed sub-image count matches the expected tiling (240 sub-images across three manually annotated TMA cores)
- All methods are scored with identical border extraction and the same 5-pixel square dilation kernel applied to both predicted and ground-truth masks
- Per-cell DSC/JSC values are computed and aggregated in addition to whole-image scores, written to a metrics CSV

## Limitations

- Voronoi undersegments cells towards the lumen because it does not take into account cell membranes, so its low scores partly reflect a known method limitation rather than annotation difficulty
- Mesmer was trained on normal mouse small intestine, normal human colon, and colorectal cancer imaging data, but not colorectal adenomas — comparator performance depends on training-domain overlap
- MIRIAM requires Ilastik (for pixel classification), and example data omits the large image stacks which must be regenerated by the user
- Quantitative similarity results were only referenced via figure pointers in parts of the paper, and no accession or availability details are given for the manually annotated ground-truth data

## Evidence
