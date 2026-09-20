---
name: marker-ablation-segmentation-evaluation
description: Use when you need to determine which multiplexed imaging markers actually
  drive segmentation quality, e.g. testing whether dropping DAPI or membrane markers
  like NaKATPase degrades MIRIAM's cell-border segmentation against a manually annotated
  gold standard.
when_to_use_negative:
- You lack manually annotated cell-border ground truth — DSC/JSC cannot be computed
  and the ablation comparison is meaningless
- Your goal is marker-panel optimization for downstream cell typing (t-SNE, quantification)
  rather than segmentation-quality attribution
- The imaging modality lacks distinct membrane markers (e.g. no NaKATPase-like channel),
  since the paper shows membrane-marker omission is precisely what drives the large
  quality drop
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: MIRIAM
  role: Segmentation pipeline run on each permutation's pixel-classification output
    to produce cell segmentations for DSC/JSC scoring
  repo: github.com/Coffey-Lab/MIRIAM
- name: Ilastik
  role: Random-forest pixel classifier retrained per marker permutation on the same
    ground-truth annotations; required dependency of the MIRIAM Matlab code
  repo: https://www.ilastik.org/download.html
- name: Matlab
  role: One of the two MIRIAM implementations used to run the segmentation ablation;
    results largely similar to Python
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: Alternative MIRIAM implementation for running the segmentation ablation; results
    largely similar to Matlab
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:2993782f830fcc35b853a1aa574a5bc9c69615749ef3718da134bb203e02dfbe
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
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# marker-ablation-segmentation-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A controlled ablation protocol that retrains MIRIAM's Ilastik random-forest pixel classifier under different marker permutations (single-marker drops, membrane-marker omissions, NaKATPase+DAPI only) and quantifies the resulting segmentation quality against manually annotated cell borders using image-level Dice (DSC) and Jaccard (JSC) similarity coefficients.

## When to use

Use when you need to determine which multiplexed imaging markers actually drive segmentation quality, e.g. testing whether dropping DAPI or membrane markers like NaKATPase degrades MIRIAM's cell-border segmentation against a manually annotated gold standard.

## When NOT to use

- You lack manually annotated cell-border ground truth — DSC/JSC cannot be computed and the ablation comparison is meaningless
- Your goal is marker-panel optimization for downstream cell typing (t-SNE, quantification) rather than segmentation-quality attribution
- The imaging modality lacks distinct membrane markers (e.g. no NaKATPase-like channel), since the paper shows membrane-marker omission is precisely what drives the large quality drop

## Inputs

- Multiplexed immunofluorescence colorectal adenoma image stacks (multi-channel per marker)
- Manually annotated cell-border ground-truth labels, partitioned into 512x512 sub-images (240 sub-images across three annotated TMA cores)
- Marker channel metadata defining each permutation (full set, single-drop, membrane+DAPI, no-membrane, NaKATPase+DAPI)
- Trained Ilastik random-forest pixel classifier project files per permutation

## Outputs

- Per-permutation cell segmentation masks from the MIRIAM pipeline
- Image-level DSC and JSC similarity scores per marker permutation
- Summary table (e.g. ablation_similarity_scores.csv) with mean ± standard deviation DSC and JSC per permutation
- Comparison against the full-marker-set baseline (DSC 0.65 ± 0.08, JSC 0.48 ± 0.08)

## How to apply

Partition the multiplexed immunofluorescence colorectal adenoma images and manually annotated cell-border labels into 512x512 sub-images (240 sub-images across the three annotated TMA cores). For each marker permutation — full marker set baseline, each single marker dropped, membrane-marker subset plus DAPI, no membrane markers, and NaKATPase+DAPI only — retrain the Ilastik random-forest pixel classifier on the identical ground-truth annotations so the only variable is the marker input. Feed each permutation's pixel-classification probability output into the MIRIAM segmentation pipeline (Matlab or Python implementation from github.com/Coffey-Lab/MIRIAM, which requires Ilastik) to generate cell segmentations. Compute image-level Dice similarity coefficient (DSC) and Jaccard similarity coefficient (JSC) per sub-image against the manual annotations, then tabulate mean ± standard deviation per permutation (e.g. ablation_similarity_scores.csv). Compare each permutation against the full-model baseline of DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08; the paper's finding is that omitting membrane markers and DAPI, or using only NaKATPase and DAPI, decreased similarity, while single-marker exclusion had minimal effect.

## Related tools

- **MIRIAM** (Segmentation pipeline run on each permutation's pixel-classification output to produce cell segmentations for DSC/JSC scoring) — github.com/Coffey-Lab/MIRIAM
- **Ilastik** (Random-forest pixel classifier retrained per marker permutation on the same ground-truth annotations; required dependency of the MIRIAM Matlab code) — https://www.ilastik.org/download.html
- **Matlab** (One of the two MIRIAM implementations used to run the segmentation ablation; results largely similar to Python) — github.com/Coffey-Lab/MIRIAM
- **Python** (Alternative MIRIAM implementation for running the segmentation ablation; results largely similar to Matlab) — github.com/Coffey-Lab/MIRIAM

## Evaluation signals

- Full-marker-set baseline reproduces DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08 at image level across the 240 sub-images
- Permutations dropping membrane markers + DAPI, or NaKATPase+DAPI only, show measurably decreased DSC/JSC relative to baseline
- Single-marker exclusions show minimal deviation from baseline DSC/JSC
- All permutations trained on the identical 512x512 sub-image ground-truth annotations (240 sub-images, three TMA cores) so scores are directly comparable
- Matlab and Python MIRIAM implementations yield largely similar segmentation results for the same permutation (per Supplemental Figure 6)

## Limitations

- Ground truth was manually annotated only for three colorectal adenoma TMA cores (240 sub-images), so ablation conclusions are tissue- and marker-set-specific
- Segmentation quality is benchmarked against manual border annotations via DSC/JSC; MIRIAM's superiority was demonstrated specifically for columnar epithelial cells with internal membranes and may not transfer to other morphologies
- No version numbers are stated for the Matlab or Python implementations, complicating exact reproduction
- No public accession or availability statement is provided for the manually annotated ground-truth data

## Evidence
