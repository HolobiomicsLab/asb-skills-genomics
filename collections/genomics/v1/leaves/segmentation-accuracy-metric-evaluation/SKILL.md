---
name: segmentation-accuracy-metric-evaluation
description: Use when predicted single-cell segmentation masks from multiplexed tissue
  imaging need to be scored against manually annotated ground truth, or when benchmarking
  competing segmentation methods (e.g., MIRIAM vs. Mesmer vs. Voronoi).
when_to_use_negative:
- Input is a per-cell marker intensity/quantification table with no segmentation masks
  — metric evaluation cannot be computed without pixel-level masks
- No manually annotated ground truth is available; the article's quantitative DSC/JSC
  comparison specifically requires manually annotated data (Fig. 2H,I)
- Goal is downstream phenotype analysis (e.g., t-SNE of marker intensities or autoencoder
  shape vectors) rather than judging segmentation quality itself
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Segmentation method being evaluated; produces cell masks scored by DSC/JSC
    against manual annotations
  repo: github.com/Coffey-Lab/MIRIAM
- name: Mesmer
  role: Deep-learning segmentation baseline compared against MIRIAM at the image level
- name: Voronoi
  role: Nucleus-dilation/Voronoi tessellation baseline showing undersegmentation of
    cells towards the lumen
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:0d9d50863fb3a31bb07b7d32706a443bbbfc805766422f607311239d9ef8e288
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
  - Voronoi
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# segmentation-accuracy-metric-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies single-cell segmentation accuracy by comparing predicted masks against manual annotations using image-level Dice similarity coefficient (DSC) and Jaccard similarity coefficient (JSC), as done to benchmark MIRIAM against Mesmer and Voronoi on manually annotated colorectal adenoma TMA cores.

## When to use

Use when predicted single-cell segmentation masks from multiplexed tissue imaging need to be scored against manually annotated ground truth, or when benchmarking competing segmentation methods (e.g., MIRIAM vs. Mesmer vs. Voronoi).

## When NOT to use

- Input is a per-cell marker intensity/quantification table with no segmentation masks — metric evaluation cannot be computed without pixel-level masks
- No manually annotated ground truth is available; the article's quantitative DSC/JSC comparison specifically requires manually annotated data (Fig. 2H,I)
- Goal is downstream phenotype analysis (e.g., t-SNE of marker intensities or autoencoder shape vectors) rather than judging segmentation quality itself

## Inputs

- Predicted single-cell segmentation masks from each candidate method (e.g., MIRIAM, Mesmer, Voronoi outputs)
- Manually annotated whole-cell ground-truth masks for a defined set of sub-images (e.g., 240 sub-images across three TMA cores)
- The underlying multiplexed tissue image stack (e.g., colorectal adenoma MxIF or MIBI breast carcinoma TMA, 2048x2048 at 0.5 um/pixel)

## Outputs

- Per-sub-image DSC and JSC values for each segmentation method
- Aggregate image-level accuracy summary (mean ± standard deviation of DSC and JSC per method)
- Qualitative comparison of segmentation overlays against manual annotations

## How to apply

Collect manual annotations of whole cells for a set of sub-images (in the MIRIAM study, 240 sub-images were analyzed across three manually annotated TMA cores of precancerous colorectal adenoma multiplexed immunofluorescence images). For each sub-image, compute the image-level Dice similarity coefficient (DSC) and Jaccard similarity coefficient (JSC) between the predicted cell mask and the annotated mask, then report the mean and standard deviation across sub-images. Benchmark multiple segmenters on the identical annotated set so their scores are directly comparable; in the article, MIRIAM achieved DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08, Mesmer DSC 0.59 ± 0.10 / JSC 0.42 ± 0.08, and Voronoi DSC 0.32 ± 0.11 / JSC 0.20 ± 0.07. Supplement quantitative scores with qualitative mask overlays to identify systematic errors such as undersegmentation of columnar epithelial cells towards the lumen (a known Voronoi failure mode).

## Related tools

- **MIRIAM** (Segmentation method being evaluated; produces cell masks scored by DSC/JSC against manual annotations) — github.com/Coffey-Lab/MIRIAM
- **Mesmer** (Deep-learning segmentation baseline compared against MIRIAM at the image level)
- **Voronoi** (Nucleus-dilation/Voronoi tessellation baseline showing undersegmentation of cells towards the lumen)

## Evaluation signals

- Per-sub-image DSC and JSC values are finite and bounded in [0, 1]; aggregate reported as mean ± standard deviation across the annotated sub-image set
- MIRIAM's published reference range: image-level DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08 on the colorectal adenoma TMA annotations; results deviating drastically suggest a mask-alignment or label-matching error
- Relative ordering expectation: MIRIAM > Mesmer > Voronoi for both DSC and JSC; Voronoi scores near DSC 0.32 / JSC 0.20 indicate the metric pipeline reproduces the benchmark
- Number of evaluated sub-images matches the annotated corpus (240 sub-images across three manually annotated TMA cores in the article)

## Limitations

- Quantitative similarity scores are only available against manually annotated colorectal adenoma data; the MIBI breast carcinoma results (Supplemental Figure 7) are qualitative/representative without published numeric scores
- Voronoi undersegments cells towards the lumen because it does not take cell membranes into account, which depresses its DSC/JSC but reflects a known baseline failure mode rather than metric noise
- Metric values are reported at the image level; per-cell-level accuracy statistics are not provided in the article text

## Evidence
