---
name: functional-data-registration-of-intensity-histograms
description: Use when integrating multiplex tissue imaging (MTI) data across samples/batches
  (CyCIF, ORION, COMET) where staining-intensity variation must be removed from right-skewed
  marker distributions that violate Z-score/ComBat assumptions.
when_to_use_negative:
- 'Data is mIHC (multiplex immunohistochemistry): chromogenic detection produces non-linear,
  enzyme-amplified signal intensities, so linear normalization factors are unsuitable'
- 'The signal of interest lies in extracellular regions: UniFORM is currently optimized
  only for within-cell normalization'
- The input is already a normalized feature table or a compressed/lossy representation
  of raw intensities whose negative populations can no longer be aligned
tools:
- name: UniFORM
  role: Non-parametric Python pipeline implementing histogram calculation, automatic
    rigid-landmark functional registration, and normalization factor application
  repo: https://github.com/kunlunW/UniFORM
- name: Python
  role: Implementation language of the UniFORM normalization pipeline
  repo: https://github.com/kunlunW/UniFORM
- name: scikit-fda
  role: Provides the automatic rigid-landmark functional data registration used to
    align intensity-distribution curves via maximum cross-correlation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:1f9199c8abeaef2168ca097005f3a63f5738faaca27fb36407bcb107e1ea456c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kunlunW/UniFORM
  tools:
  - Python
  - UniFORM
  - scikit-fda
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# functional-data-registration-of-intensity-histograms

## Summary

UniFORM's core alignment method: log-transformed marker intensity histograms from MTI samples (cell-level features or masked pixels) are converted to functional data curves and aligned to a reference distribution via automatic rigid-landmark registration (scikit-fda) using maximum cross-correlation, yielding normalization factors that align biologically invariant negative populations while preserving positive-population shapes.

## When to use

Use when integrating multiplex tissue imaging (MTI) data across samples/batches (CyCIF, ORION, COMET) where staining-intensity variation must be removed from right-skewed marker distributions that violate Z-score/ComBat assumptions.

## When NOT to use

- Data is mIHC (multiplex immunohistochemistry): chromogenic detection produces non-linear, enzyme-amplified signal intensities, so linear normalization factors are unsuitable
- The signal of interest lies in extracellular regions: UniFORM is currently optimized only for within-cell normalization
- The input is already a normalized feature table or a compressed/lossy representation of raw intensities whose negative populations can no longer be aligned

## Inputs

- AnnData .h5ad feature table of per-cell log-transformed marker intensities (feature-level workflow)
- OME-TIFF whole-slide image stack plus nucleus/cell segmentation masks (pixel-level workflow, Dask-backed reading)

## Outputs

- Per-marker normalization factors estimated from the rigid-landmark functional registration
- Normalized feature table with aligned negative populations and preserved distribution shapes
- normalized_pixel_stack.ome.tiff (pixel-level workflow, uint16 dynamic range)

## How to apply

For each marker, calculate the histogram distribution of log-transformed intensities — cell-level feature intensities from .h5ad/AnnData inputs, or pixel intensities restricted to nucleus/cell segmentation masks for OME-TIFF whole-slide stacks. Convert the histograms into functional data curves and align each sample to an appropriate reference distribution using an automated rigid-landmark functional registration (scikit-fda) that maximizes cross-correlation. Estimate linear normalization factors from the alignment of the biologically invariant negative-population peaks, then apply them to the raw intensities. For pixel-level uint16 data, additionally apply a stretching factor derived from the most-compressed sample before rescaling to 16-bit, preventing rounding-spike artifacts from normalization factors <1. Markers whose distributions violate automatic-workflow assumptions are flagged via a Kullback-Leibler divergence-based metric and routed to the optional guided fine-tuning mode.

## Related tools

- **UniFORM** (Non-parametric Python pipeline implementing histogram calculation, automatic rigid-landmark functional registration, and normalization factor application) — https://github.com/kunlunW/UniFORM
- **Python** (Implementation language of the UniFORM normalization pipeline) — https://github.com/kunlunW/UniFORM
- **scikit-fda** (Provides the automatic rigid-landmark functional data registration used to align intensity-distribution curves via maximum cross-correlation)

## Examples

```
git clone https://github.com/kunlunW/UniFORM.git
```

## Evaluation signals

- Alignment of negative-population peaks across samples/batches in the normalized intensity distributions (visual/quantitative check of marker histograms)
- Downstream metric improvements: kBET (k-nearest neighbor batch effect test) and silhouette scores increase after normalization
- Positive-population preservation: overall distribution shape and positive-population proportions are retained (positive population change metric)
- More coherent UMAP and Leiden clustering relative to MxNorm registration, Z score, ComBat, and mean division baselines
- Pixel-level check: the normalized OME-TIFF is readable and its pixel values stay within uint16 dynamic range without rounding-spike artifacts at low intensities

## Limitations

- Highly heterogeneous/irregular marker distributions (e.g., CD45 in PRAD-CyCIF) violate the automatic workflow assumptions and are flagged via a KL divergence-based metric, requiring the guided fine-tuning mode
- Effectiveness depends on MTI dataset quality and the quality of cell/nuclei segmentation masks; compromised signals cannot be recovered
- Linear normalization factors cannot handle mIHC's non-linear chromogenic amplification; a non-linear warping function is unimplemented future work
- The Pixel-Level (OME-TIFF) example dataset and tutorial were listed as still being assembled in the methods

## Evidence
