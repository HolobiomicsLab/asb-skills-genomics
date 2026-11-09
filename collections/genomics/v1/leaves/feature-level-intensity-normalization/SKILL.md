---
name: feature-level-intensity-normalization
description: Use when integrating single-cell feature tables (e.g., .pkl or .h5ad
  from PRAD-CyCIF) across batches or platforms (CyCIF, ORION, COMET) whose marker
  intensity distributions are shifted by staining/imaging variation, and you need
  batch-corrected intensities that preserve biological signal.
when_to_use_negative:
- Input is multiplexed immunohistochemistry (mIHC) data — chromogenic detection produces
  non-linear signal intensities that linear normalization factors cannot correct
- Input is image/pixel-level data without nucleus/cell segmentation masks, or the
  goal is extracellular-region normalization — UniFORM is optimized for within-cell
  normalization only
- The MTI dataset or its segmentation masks are of compromised quality — UniFORM depends
  on input quality and cannot recover lost signal
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: UniFORM
  role: Non-parametric Python pipeline implementing the histogram -> registration
    -> normalization workflow for feature-level MTI data
  repo: https://github.com/kunlunW/UniFORM
- name: scikit-fda
  role: Converts marker histograms into functional data curves and performs automatic
    rigid-landmark registration
- name: scipy
  role: FFT-based maximum cross-correlation algorithm used to align each marker's
    distribution to a reference distribution
- name: numpy
  role: Computes histogram distributions of log-transformed cell intensities over
    a shared global range
- name: anndata
  role: Reads .h5ad single-cell feature-level inputs
- name: pandas
  role: Reads pickled (*.pkl) single-cell feature-level inputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:683a10fcc4e871d1056fc8ca133a55277b65d1690b4c8e1c330875a7a6e7f7fa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kunlunW/UniFORM
  tools:
  - Python
  - UniFORM
  - anndata
  - numpy
  - pandas
  - scikit-fda
  - scipy
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# feature-level-intensity-normalization

## Summary

UniFORM's three-step feature-level normalization of multiplex tissue imaging (MTI) single-cell data: histogram calculation of log-transformed intensities, automatic rigid-landmark functional data registration via maximum cross-correlation, and application of the estimated normalization factors to raw per-cell intensities.

## When to use

Use when integrating single-cell feature tables (e.g., .pkl or .h5ad from PRAD-CyCIF) across batches or platforms (CyCIF, ORION, COMET) whose marker intensity distributions are shifted by staining/imaging variation, and you need batch-corrected intensities that preserve biological signal.

## When NOT to use

- Input is multiplexed immunohistochemistry (mIHC) data — chromogenic detection produces non-linear signal intensities that linear normalization factors cannot correct
- Input is image/pixel-level data without nucleus/cell segmentation masks, or the goal is extracellular-region normalization — UniFORM is optimized for within-cell normalization only
- The MTI dataset or its segmentation masks are of compromised quality — UniFORM depends on input quality and cannot recover lost signal

## Inputs

- Feature-level single-cell MTI data (*.pkl readable with pandas, or *.h5ad readable with anndata), e.g., PRAD-CyCIF from Zenodo record 16383766
- Marker intensity columns (log-transformed cell mean intensities per marker per cell)
- Batch/sample labels per cell for per-batch factor estimation

## Outputs

- Normalized feature matrix (e.g., normalized_feature_data.pkl) with per-marker normalization factors applied
- Per-marker, per-batch rigid shift/scale normalization factors estimated from the rigid-landmark alignment
- Aligned marker intensity distributions matched to a reference distribution

## How to apply

Load the feature-level single-cell dataset (e.g., PRAD-CyCIF from Zenodo record 16383766 as *.pkl via pandas or *.h5ad via anndata) in a Python 3.10 environment with the UniFORM requirements installed. For each marker, compute the histogram distribution of log-transformed cell mean intensities over a shared global range with numpy so distributions from different batches are comparable. Convert each histogram into a functional data object with scikit-fda and perform automatic rigid-landmark registration, using the scipy FFT-based maximum cross-correlation algorithm to align each marker's distribution to an appropriate reference distribution; markers flagged by the Kullback-Leibler divergence-based metric as highly heterogeneous (e.g., CD45 in PRAD-CyCIF) should be routed to the guided fine-tuning mode instead of the automatic workflow. Extract the rigid shift/scale factors per marker and batch, apply them to the raw per-cell mean intensities, and save the result (e.g., normalized_feature_data.pkl). Rationale: aligning the biologically invariant negative population peaks removes technical variation while rigid (shape-preserving) alignment avoids warping that would distort positive-population proportions.

## Related tools

- **UniFORM** (Non-parametric Python pipeline implementing the histogram -> registration -> normalization workflow for feature-level MTI data) — https://github.com/kunlunW/UniFORM
- **scikit-fda** (Converts marker histograms into functional data curves and performs automatic rigid-landmark registration)
- **scipy** (FFT-based maximum cross-correlation algorithm used to align each marker's distribution to a reference distribution)
- **numpy** (Computes histogram distributions of log-transformed cell intensities over a shared global range)
- **anndata** (Reads .h5ad single-cell feature-level inputs)
- **pandas** (Reads pickled (*.pkl) single-cell feature-level inputs)

## Examples

```
python scripts/normalize.py --input PRAD-CyCIF_feature_data.h5ad --output normalized_feature_data.pkl
```

## Evaluation signals

- Aligned marker distributions match the reference at the biologically invariant negative-population peaks
- Overall distribution shape is preserved (no warping of curve shape) and positive-population proportions are maintained
- Mutually exclusive markers remain mutually exclusive after normalization
- kBET (k-nearest neighbor batch effect test) and silhouette scores improve relative to raw data and competing methods (Z score, ComBat, mean division, MxNorm registration)
- UMAP visualizations and Leiden clustering become more coherent across batches

## Limitations

- Linear normalization factors are unsuitable for mIHC because chromogenic detection produces non-linear signal intensities from enzyme-catalyzed colorimetric amplification; a warping function for non-linear mIHC normalization is future work
- Highly heterogeneous/irregular marker distributions (e.g., CD45 in PRAD-CyCIF) violate the automatic workflow's assumptions and require guided fine-tuning; such markers are flagged with a Kullback-Leibler (KL) divergence-based metric
- Effectiveness depends on the quality of the MTI dataset and the cell/nuclei segmentation masks used to extract signals; compromised signals cannot be recovered

## Evidence
