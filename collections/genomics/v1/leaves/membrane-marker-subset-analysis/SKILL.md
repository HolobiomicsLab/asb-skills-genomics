---
name: membrane-marker-subset-analysis
description: Use when you need to determine which input markers actually drive multiplexed-image
  cell segmentation quality, e.g. testing whether DAPI or membrane markers (like NaKATPase)
  are necessary for MIRIAM border delineation.
when_to_use_negative:
- Input images lack manually annotated cell-border ground truth, making DSC/JSC computation
  impossible
- The imaging modality has no membrane or nuclear markers available to permute (e.g.
  marker sets without NaKATPase/DAPI analogues)
- The goal is general segmentation benchmarking against other methods (Mesmer, Voronoi)
  rather than marker-composition sensitivity analysis
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIRIAM
  role: Segmentation pipeline run on each pixel-classification output to generate
    cell segmentations for similarity scoring
  repo: github.com/Coffey-Lab/MIRIAM
- name: Python
  role: One of the two MIRIAM implementations usable for the segmentation step
- name: Matlab
  role: Primary MIRIAM implementation used for cell segmentation on multiplexed imaging
    data
  repo: github.com/Coffey-Lab/MIRIAM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:e91f84c2882b5ee6c4d776bba5c8b7a1a0528f671c3522a7395b2094b8fd9adf
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
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# membrane-marker-subset-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An ablation procedure that retrains the Ilastik random-forest pixel classifier on permuted marker subsets (full set, single-marker drops, membrane-marker combinations) and quantifies MIRIAM cell-border segmentation quality via DSC/JSC against manually annotated colorectal adenoma borders.

## When to use

Use when you need to determine which input markers actually drive multiplexed-image cell segmentation quality, e.g. testing whether DAPI or membrane markers (like NaKATPase) are necessary for MIRIAM border delineation.

## When NOT to use

- Input images lack manually annotated cell-border ground truth, making DSC/JSC computation impossible
- The imaging modality has no membrane or nuclear markers available to permute (e.g. marker sets without NaKATPase/DAPI analogues)
- The goal is general segmentation benchmarking against other methods (Mesmer, Voronoi) rather than marker-composition sensitivity analysis

## Inputs

- Multiplexed immunofluorescence image stacks of colorectal adenoma TMA cores (512x512 sub-images)
- Manually annotated cell-border ground-truth labels
- Marker-channel metadata defining the full marker set (including membrane markers such as NaKATPase and nuclear marker DAPI)

## Outputs

- Per-permutation cell segmentations from the MIRIAM pipeline
- ablation_similarity_scores.csv tabulating mean ± standard deviation DSC and JSC per marker permutation
- Comparison table against the full-marker-set baseline (DSC 0.65 ± 0.08, JSC 0.48 ± 0.08)

## How to apply

Partition the multiplexed immunofluorescence images and manually annotated cell-border labels into 512x512 sub-images (240 sub-images across the three annotated TMA cores in the reference study). For each marker permutation — full marker set baseline, each single marker dropped, membrane markers plus DAPI, no membrane markers, and NaKATPase+DAPI only — retrain the Ilastik random-forest pixel classifier on the identical ground-truth annotations, then run the MIRIAM segmentation pipeline (Matlab or Python) on each pixel-classification output. Compute image-level Dice (DSC) and Jaccard (JSC) similarity coefficients per permutation against the manual borders and tabulate mean ± standard deviation, benchmarking against the full-model baseline of DSC 0.65 ± 0.08 and JSC 0.48 ± 0.08. The rationale is that the study found omitting membrane markers and DAPI, or restricting to NaKATPase+DAPI, decreased border similarity while single-marker exclusions had minimal effect — so expected degradation concentrates on membrane-marker removal, not individual marker drops.

## Related tools

- **MIRIAM** (Segmentation pipeline run on each pixel-classification output to generate cell segmentations for similarity scoring) — github.com/Coffey-Lab/MIRIAM
- **Python** (One of the two MIRIAM implementations usable for the segmentation step)
- **Matlab** (Primary MIRIAM implementation used for cell segmentation on multiplexed imaging data) — github.com/Coffey-Lab/MIRIAM

## Examples

```
python run_miriam_ablation.py --markers full,single_drop,membrane+dapi,no_membrane,nakatpase+dapi --labels manual_borders/ --subimage-size 512 --out ablation_similarity_scores.csv
```

## Evaluation signals

- Full-marker-set baseline reproduces DSC ≈ 0.65 ± 0.08 and JSC ≈ 0.48 ± 0.08 against manual borders
- Permutations omitting membrane markers and DAPI, or using only NaKATPase+DAPI, show decreased DSC/JSC relative to baseline
- Single-marker exclusions produce minimal change in DSC/JSC versus the full-model baseline
- ablation_similarity_scores.csv contains one row per permutation with mean ± standard deviation for both DSC and JSC
- Matlab and Python MIRIAM implementations yield largely similar segmentation results across permutations

## Limitations

- Ground-truth similarity depends on manually annotated TMA cores; no accession or public availability details are provided for the annotated data
- Quantitative similarity scores are referenced via figure pointers (Fig 2H,I) rather than fully reported in text
- No version numbers stated for the Matlab or Python MIRIAM implementations
- MIRIAM was demonstrated on multiplexed immunofluorescence adenoma data and MIBI breast carcinoma data; behavior on other platforms/marker sets is not characterized

## Evidence
