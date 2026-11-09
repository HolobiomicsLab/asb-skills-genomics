---
name: cross-implementation-reproducibility-analysis
description: Use when the same segmentation/pipeline exists in two implementations
  (e.g., Matlab and Python) and you must verify they yield consistent outputs on an
  identical image set before adoption or publication.
when_to_use_negative:
- When comparing MIRIAM against a different segmentation method (e.g., Mesmer or Voronoi)
  rather than a second implementation of MIRIAM itself — that is a method-benchmarking
  task with different reference metrics.
- When input images are not identically paired across the two implementations (same
  120 cores, same settings), since cross-implementation agreement is meaningless on
  mismatched inputs.
- When the pipeline has no officially maintained second implementation, since there
  would be no independent implementation to compare against.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: The segmentation pipeline under test; both its Matlab and Python implementations
    are executed on the same TMA cores to produce comparable cell membrane masks.
  repo: https://github.com/Coffey-Lab/MIRIAM
- name: Matlab
  role: Reference implementation environment (R2018b) used to generate the Matlab-derived
    segmentation masks.
- name: Python
  role: Ported implementation environment (3.7) used to generate the Python-derived
    segmentation masks for agreement computation.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:c38f09d73226276b90396d212d3865b7ede9c4aa8ef84830e69c14851df54ad1
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# cross-implementation-reproducibility-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies agreement between two independent language implementations of the same pipeline — here Matlab (R2018b) vs Python (3.7) builds of MIRIAM — by computing per-image segmentation overlap metrics (DSC, JSC) across a matched image set and aggregating to mean ± standard deviation.

## When to use

Use when the same segmentation/pipeline exists in two implementations (e.g., Matlab and Python) and you must verify they yield consistent outputs on an identical image set before adoption or publication.

## When NOT to use

- When comparing MIRIAM against a different segmentation method (e.g., Mesmer or Voronoi) rather than a second implementation of MIRIAM itself — that is a method-benchmarking task with different reference metrics.
- When input images are not identically paired across the two implementations (same 120 cores, same settings), since cross-implementation agreement is meaningless on mismatched inputs.
- When the pipeline has no officially maintained second implementation, since there would be no independent implementation to compare against.

## Inputs

- 120 colon adenoma TMA core multiplexed immunofluorescence images
- MIRIAM Matlab implementation (R2018b) from Coffey-Lab GitHub repository
- MIRIAM Python implementation (3.7) with identical pipeline settings
- Reported reference agreement metrics (DSC 0.84 ± 0.06, JSC 0.74 ± 0.09, Supplemental Figure 6)

## Outputs

- Per-image DSC and JSC values for each of the 120 cores
- Summary mean ± standard deviation of DSC and JSC across cores
- implementation_comparison_metrics.csv

## How to apply

Run both implementations of MIRIAM (random-forest pixel classification followed by internal-membrane-based cell separation) over the identical 120 colon adenoma TMA cores using Matlab R2018b and Python 3.7, obtaining cell/membrane segmentation masks from each. For every image, compute the image-level cell-membrane Dice similarity coefficient (DSC) and Jaccard similarity coefficient (JSC) between the Matlab-derived and Python-derived masks. Aggregate per-image values into mean ± standard deviation across the 120 cores and export per-image and summary rows (e.g., to implementation_comparison_metrics.csv). Accept the reproduction if mean DSC falls within 0.84 ± 0.06 and mean JSC within 0.74 ± 0.09, matching the cross-implementation agreement reported in Supplemental Figure 6. The rationale is that the paper explicitly ships and supports both implementations, so a large divergence would indicate a porting defect rather than method variance.

## Related tools

- **MIRIAM** (The segmentation pipeline under test; both its Matlab and Python implementations are executed on the same TMA cores to produce comparable cell membrane masks.) — https://github.com/Coffey-Lab/MIRIAM
- **Matlab** (Reference implementation environment (R2018b) used to generate the Matlab-derived segmentation masks.)
- **Python** (Ported implementation environment (3.7) used to generate the Python-derived segmentation masks for agreement computation.)

## Evaluation signals

- Mean image-level DSC across the 120 cores falls within 0.84 ± 0.06.
- Mean image-level JSC across the 120 cores falls within 0.74 ± 0.09.
- Per-image DSC and JSC computed for all 120 cores with no missing pairs (one Matlab mask and one Python mask per image).
- Per-image and summary metrics written to implementation_comparison_metrics.csv and summary values agree with Supplemental Figure 6.
- Both implementations ran with identical pipeline settings (random-forest pixel classification plus internal-membrane-based cell separation) on the same input stacks.

## Limitations

- Agreement metrics are sensitive to image set choice; the reported 0.84 ± 0.06 DSC and 0.74 ± 0.09 JSC were established on colon adenoma TMA cores and may not transfer to other tissue types such as the breast carcinoma MIBI dataset.
- The article provides no version numbers for the Python implementation or changelog, so exact build parity may be unverifiable from metadata alone.
- Matlab pipeline requires external dependencies (e.g., Ilastik) that must be installed identically for both environments to attribute residual differences to implementation rather than setup.

## Evidence
