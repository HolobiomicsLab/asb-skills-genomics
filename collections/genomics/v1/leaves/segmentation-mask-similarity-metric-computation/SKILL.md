---
name: segmentation-mask-similarity-metric-computation
description: Use when you have two independent segmentation runs of the same pipeline
  (e.g., Matlab vs Python implementations of MIRIAM) over the same images and need
  image-level DSC/JSC agreement metrics per mask and aggregated as mean ± standard
  deviation.
when_to_use_negative:
- Comparing a single segmentation output against manually annotated ground truth (e.g.,
  MIRIAM vs manual annotation with DSC 0.65 ± 0.08 / JSC 0.48 ± 0.08, or Mesmer/Voronoi
  comparisons) — that is a benchmarking task, not cross-implementation equivalence
  testing
- Quantifying per-cell marker intensities or cell-shape features after segmentation
  is already complete
- Input masks come from one implementation only, with no second segmentation to compare
  against
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: The segmentation pipeline whose Matlab and Python implementations generate
    the paired cell-membrane masks being compared
  repo: github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Reference implementation (R2018b) producing one side of the mask comparison
- name: Python
  role: Second implementation (3.7) producing the other side of the mask comparison;
    also used to compute DSC/JSC metrics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:a1e41e83b2d50e5c301e6a514c170664d11b1cf4ee12a48773a2910261412715
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# segmentation-mask-similarity-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes image-level Dice similarity coefficient (DSC) and Jaccard similarity coefficient (JSC) between two sets of cell-membrane segmentation masks to quantify cross-implementation agreement, as done for the Matlab (R2018b) and Python (3.7) MIRIAM implementations over 120 colon adenoma TMA cores.

## When to use

Use when you have two independent segmentation runs of the same pipeline (e.g., Matlab vs Python implementations of MIRIAM) over the same images and need image-level DSC/JSC agreement metrics per mask and aggregated as mean ± standard deviation.

## When NOT to use

- Comparing a single segmentation output against manually annotated ground truth (e.g., MIRIAM vs manual annotation with DSC 0.65 ± 0.08 / JSC 0.48 ± 0.08, or Mesmer/Voronoi comparisons) — that is a benchmarking task, not cross-implementation equivalence testing
- Quantifying per-cell marker intensities or cell-shape features after segmentation is already complete
- Input masks come from one implementation only, with no second segmentation to compare against

## Inputs

- Matlab (R2018b) MIRIAM cell/membrane segmentation masks for each of the 120 colon adenoma TMA cores
- Python (3.7) MIRIAM cell/membrane segmentation masks for the same 120 cores, generated with identical pipeline settings
- Binary label images (2D masks) aligned on the same image grid per core

## Outputs

- Per-image DSC and JSC values between Matlab- and Python-derived masks
- Mean ± standard deviation DSC and JSC across the 120 cores (expected ~0.84 ± 0.06 and ~0.74 ± 0.09)
- implementation_comparison_metrics.csv containing per-image and summary metrics

## How to apply

Run both pipeline implementations (here: MIRIAM's random-forest pixel classification plus internal-membrane-based cell separation, in Matlab R2018b and Python 3.7) on each image — 120 colon adenoma TMA cores in the reference study — producing paired cell/membrane segmentation masks. For every image, compute the image-level cell-membrane DSC (2|A∩B|/(|A|+|B|)) and JSC (|A∩B|/|A∪B|) between the Matlab-derived and Python-derived masks. Aggregate the per-image DSC and JSC into mean ± standard deviation across all cores and write per-image and summary rows to implementation_comparison_metrics.csv. Accept the comparison as reproduced if mean DSC falls within 0.84 ± 0.06 and mean JSC within 0.74 ± 0.09, matching the cross-implementation agreement reported in Supplemental Figure 6. Note that JSC must always be ≤ DSC, so use that invariant to sanity-check the metric code.

## Related tools

- **MIRIAM** (The segmentation pipeline whose Matlab and Python implementations generate the paired cell-membrane masks being compared) — github.com/Coffey-Lab/MIRIAM
- **Matlab** (Reference implementation (R2018b) producing one side of the mask comparison)
- **Python** (Second implementation (3.7) producing the other side of the mask comparison; also used to compute DSC/JSC metrics)

## Examples

```
python compute_mask_agreement.py --matlab_masks masks_matlab/ --python_masks masks_python/ --metric dsc jsc --out implementation_comparison_metrics.csv
```

## Evaluation signals

- Mean image-level DSC across the 120 cores falls within 0.84 ± 0.06, matching Supplemental Figure 6
- Mean image-level JSC falls within 0.74 ± 0.09, matching Supplemental Figure 6
- Per-image check that JSC ≤ DSC for every core (mathematical invariant of the two metrics)
- implementation_comparison_metrics.csv contains exactly 120 per-image rows plus summary rows
- Both mask sets were generated with the same pipeline settings (random-forest pixel classification followed by internal-membrane-based cell separation) on identical input images

## Limitations

- The paper only points to the quantitative agreement via Supplemental Figure 6 rather than reporting the values in the main text; the 0.84 ± 0.06 / 0.74 ± 0.09 thresholds come from the source task card
- No version numbers for the Matlab or Python implementations are stated in the paper itself (R2018b and 3.7 come from the task card), so exact environment reproduction may require additional pinning
- The repository README notes that image stacks are too large to ship with example data and must be regenerated by the user, and that Ilastik must be installed as a dependency

## Evidence
