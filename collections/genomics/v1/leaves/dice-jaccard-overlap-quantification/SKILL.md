---
name: dice-jaccard-overlap-quantification
description: Use when two cell/membrane segmentation masks over the same tissue image
  (e.g., MIRIAM vs manual annotation, MIRIAM vs Mesmer/Voronoi, or Matlab vs Python
  MIRIAM outputs) must be scored for pixel-level agreement with DSC and JSC.
when_to_use_negative:
- When masks have not been aligned/tiled consistently (e.g., predicted and ground-truth
  sub-images are not the same 512x512 grid), since DSC/JSC assume pixelwise co-registration
- When the goal is biological population discovery rather than segmentation-accuracy
  scoring — the article's t-SNE analyses of marker intensities or autoencoder latent
  vectors, not DSC/JSC, serve that purpose
- When evaluating compartment-level marker quantification (nucleus, membrane, cytoplasm
  intensities) rather than membrane-border overlap
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Source of predicted cell masks whose membrane borders are scored against manual
    annotations or a second implementation via DSC/JSC
  repo: github.com/Coffey-Lab/MIRIAM
- name: Mesmer
  role: Comparator deep-learning segmentation method scored against the same manual
    annotations with DSC/JSC
- name: Python
  role: One of the two MIRIAM implementations whose outputs are compared pairwise
    using DSC/JSC
- name: Matlab
  role: The other MIRIAM implementation (R2018b) whose outputs are compared pairwise
    using DSC/JSC
provenance:
  source_task_ids:
  - task_004
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:3c11dfc8a9685bbcedc161c73de90f0f2f6067e953f4b7d702ff31768c162a61
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
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# dice-jaccard-overlap-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies agreement between predicted and reference cell-membrane segmentations using image-level and per-cell Dice (DSC) and Jaccard (JSC) similarity coefficients, as used to benchmark MIRIAM against manual annotations, Mesmer, and Voronoi on colorectal adenoma TMA cores.

## When to use

Use when two cell/membrane segmentation masks over the same tissue image (e.g., MIRIAM vs manual annotation, MIRIAM vs Mesmer/Voronoi, or Matlab vs Python MIRIAM outputs) must be scored for pixel-level agreement with DSC and JSC.

## When NOT to use

- When masks have not been aligned/tiled consistently (e.g., predicted and ground-truth sub-images are not the same 512x512 grid), since DSC/JSC assume pixelwise co-registration
- When the goal is biological population discovery rather than segmentation-accuracy scoring — the article's t-SNE analyses of marker intensities or autoencoder latent vectors, not DSC/JSC, serve that purpose
- When evaluating compartment-level marker quantification (nucleus, membrane, cytoplasm intensities) rather than membrane-border overlap

## Inputs

- Binary predicted cell/membrane segmentation masks (e.g., from MIRIAM, Mesmer, or Voronoi)
- Manually annotated cell-border ground-truth masks (colorectal adenoma TMA cores tiled into 512x512 sub-images)
- Matlab- and Python-derived MIRIAM segmentation masks over 120 colon adenoma TMA cores (for cross-implementation comparison)

## Outputs

- Per-image DSC and JSC values
- Per-cell DSC and JSC values
- Mean ± standard deviation summary metrics (e.g., segmentation_similarity_metrics.csv, ablation_similarity_scores.csv, implementation_comparison_metrics.csv)

## How to apply

Tile the multiplexed immunofluorescence TMA cores into 512x512 sub-images (240 sub-images across three manually annotated cores in the reference study). Extract cell borders from each predicted mask and dilate them with a 5-pixel square kernel, applying identical border extraction and dilation to the manually annotated ground-truth masks so both are compared at the membrane-border level. Compute the Dice similarity coefficient (DSC = 2|A∩B|/(|A|+|B|)) and Jaccard similarity coefficient (JSC = |A∩B|/|A∪B|) at the whole sub-image level and per individual cell. Aggregate per-image values into mean ± standard deviation across all sub-images or cores, and write per-image and summary metrics to a CSV (e.g., segmentation_similarity_metrics.csv or implementation_comparison_metrics.csv). Validate against reported benchmarks: MIRIAM DSC 0.65 ± 0.08 / JSC 0.48 ± 0.08, Mesmer 0.59 ± 0.10 / 0.42 ± 0.08, Voronoi 0.32 ± 0.11 / 0.20 ± 0.07, and Matlab-vs-Python MIRIAM DSC 0.84 ± 0.06 / JSC 0.74 ± 0.09.

## Related tools

- **MIRIAM** (Source of predicted cell masks whose membrane borders are scored against manual annotations or a second implementation via DSC/JSC) — github.com/Coffey-Lab/MIRIAM
- **Mesmer** (Comparator deep-learning segmentation method scored against the same manual annotations with DSC/JSC)
- **Python** (One of the two MIRIAM implementations whose outputs are compared pairwise using DSC/JSC)
- **Matlab** (The other MIRIAM implementation (R2018b) whose outputs are compared pairwise using DSC/JSC)

## Examples

```
from sklearn.metrics import f1_score, jaccard_score; dsc = f1_score(gt_borders.ravel(), pred_borders.ravel()); jsc = jaccard_score(gt_borders.ravel(), pred_borders.ravel())
```

## Evaluation signals

- Reproduced image-level DSC for MIRIAM vs manual annotation falls within 0.65 ± 0.08 and JSC within 0.48 ± 0.08
- Mesmer values fall within DSC 0.59 ± 0.10 / JSC 0.42 ± 0.08 and Voronoi within DSC 0.32 ± 0.11 / JSC 0.20 ± 0.07, with MIRIAM exceeding both comparators
- Matlab-vs-Python MIRIAM cross-implementation agreement yields mean image-level DSC within 0.84 ± 0.06 and JSC within 0.74 ± 0.09 (Supplemental Figure 6)
- Identical border extraction and 5-pixel square-kernel dilation are applied to both predicted and ground-truth masks before scoring
- Sub-image count matches the reference design: 240 sub-images across the three manually annotated TMA cores

## Limitations

- Scores are membrane-border-based after dilation, so they measure border agreement rather than whole-cell volume agreement; Voronoi undersegments cells towards the lumen because it does not take cell membranes into account, which DSC/JSC will penalize
- No accession or availability details for the manually annotated ground-truth data are provided, limiting independent re-scoring
- In some sections the quantitative similarity scores are only referenced via figure pointers (Fig. 2H, I; Supplemental Figure 6) rather than reported in text

## Evidence
