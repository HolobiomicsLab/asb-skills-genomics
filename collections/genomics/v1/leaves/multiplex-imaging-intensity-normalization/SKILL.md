---
name: multiplex-imaging-intensity-normalization
description: Use when integrating single-cell or pixel-level MTI data (CyCIF, ORION,
  COMET platforms) across batches with right-skewed, heterogeneous marker distributions
  that violate Z score/ComBat assumptions.
when_to_use_negative:
- 'Data are mIHC (multiplex immunohistochemistry) samples: chromogenic detection produces
  non-linear signal intensities due to enzyme-catalyzed colorimetric amplification,
  for which linear normalization factors are unsuitable'
- Input is already normalized, or the goal is analysis of normalized data without
  cross-batch intensity alignment (e.g., pure downstream clustering on a single batch)
- Signals come from extracellular regions or compromised datasets/segmentation masks
  — UniFORM is optimized for within-cell normalization and cannot recover compromised
  signals
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_2640
tools:
- name: UniFORM
  role: Primary Python pipeline performing histogram calculation, functional data
    registration, and normalization at feature and pixel levels
  repo: https://github.com/kunlunW/UniFORM
- name: MxNorm
  role: Benchmark R package providing the registration (B-spline) and mean-division
    normalization baselines
  repo: https://github.com/ColemanRHarris/mxnorm
- name: pyComBat
  role: Benchmark ComBat batch-correction baseline applied to the same distributions
  repo: https://epigenelabs.github.io/pyComBat/
- name: Python
  role: Runtime environment (Python 3.10 conda environment with UniFORM requirements.txt)
    for running the pipeline
  repo: https://github.com/kunlunW/UniFORM
provenance:
  source_task_ids:
  - task_004
  - task_002
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:87738c210f304d873daeccb583892ed9ce531fca9db44865ad4deb5149bfc1ef
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kunlunW/UniFORM
  tools:
  - MxNorm
  - Python
  - UniFORM
  - pyComBat
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multiplex-imaging-intensity-normalization

## Summary

Normalizes multiplex tissue imaging (MTI) marker intensities across samples and batches at both feature (single-cell) and pixel levels by aligning biologically invariant negative populations with automatic rigid-landmark functional registration, preserving distribution shape and positive-population proportions.

## When to use

Use when integrating single-cell or pixel-level MTI data (CyCIF, ORION, COMET platforms) across batches with right-skewed, heterogeneous marker distributions that violate Z score/ComBat assumptions.

## When NOT to use

- Data are mIHC (multiplex immunohistochemistry) samples: chromogenic detection produces non-linear signal intensities due to enzyme-catalyzed colorimetric amplification, for which linear normalization factors are unsuitable
- Input is already normalized, or the goal is analysis of normalized data without cross-batch intensity alignment (e.g., pure downstream clustering on a single batch)
- Signals come from extracellular regions or compromised datasets/segmentation masks — UniFORM is optimized for within-cell normalization and cannot recover compromised signals

## Inputs

- Single-cell feature-level data as AnnData .h5ad or Pickle .pkl files (e.g., PRAD-CyCIF from Zenodo record 16383766; CRC-ORION from HTAN participant HTA7_926)
- OME-TIFF whole-slide image stacks with nucleus/cell segmentation masks (pixel-level pipeline)
- Marker intensity distributions per sample/batch (log-transformed)

## Outputs

- Normalized single-cell feature table with aligned marker intensity distributions
- Normalized pixel-level image stack as OME-TIFF (normalized_pixel_stack.ome.tiff) within uint16 dynamic range
- Estimated per-sample normalization factors and benchmark figures (e.g., Figure 2A-style before/after distribution plots)

## How to apply

Load single-cell feature data (AnnData .h5ad or Pickle .pkl) and compute the histogram distribution of log-transformed cell intensities per marker. Convert histograms into functional data curves and align them to a reference distribution via maximum cross-correlation using automatic rigid-landmark functional registration (scikit-fda), then apply the estimated normalization factors to the raw feature data. Aligning the biologically invariant negative population peaks removes technical variation while preserving tissue-specific expression in positive populations. For pixel-level data, restrict histograms to pixels within nucleus/cell segmentation masks, apply the same registration to estimate factors, and use a uint16 stretching factor from the most-compressed sample to avoid range-compression artifacts when writing OME-TIFF output. For markers with high noise or heterogeneous/irregular distributions flagged by the KL divergence-based metric (e.g., CD45 in PRAD-CyCIF), use the guided fine-tuning mode instead of the automatic workflow.

## Related tools

- **UniFORM** (Primary Python pipeline performing histogram calculation, functional data registration, and normalization at feature and pixel levels) — https://github.com/kunlunW/UniFORM
- **MxNorm** (Benchmark R package providing the registration (B-spline) and mean-division normalization baselines) — https://github.com/ColemanRHarris/mxnorm
- **pyComBat** (Benchmark ComBat batch-correction baseline applied to the same distributions) — https://epigenelabs.github.io/pyComBat/
- **Python** (Runtime environment (Python 3.10 conda environment with UniFORM requirements.txt) for running the pipeline) — https://github.com/kunlunW/UniFORM

## Examples

```
git clone https://github.com/kunlunW/UniFORM.git && conda create -n uniform python=3.10 && pip install -r requirements.txt
```

## Evaluation signals

- Before/after intensity distribution plots show negative population peaks aligned while the bimodal ECAD (PRAD-CyCIF) and unimodal PD-L1 (CRC-ORION) distribution shapes are preserved, matching the Figure 2A layout
- kBET and silhouette scores improve after normalization relative to raw data and benchmark methods (MxNorm registration, Z score, ComBat, mean division)
- Runtime on PRAD-CyCIF is consistent with ~4-fold speedup over MxNorm (UniFORM 30.58 ± 0.30 s vs 114.75 ± 4.35 s) via a value_in_range tolerance check
- Normalized OME-TIFF pixel stack is readable, pixel values stay within uint16 dynamic range, and no rounding-spike artifacts appear at low intensities
- Positive-population proportions and mutual exclusivity of mutually exclusive markers are maintained after normalization

## Limitations

- Linear normalization factors are unsuitable for mIHC due to non-linear chromogenic signal amplification; a warping function for non-linear mIHC normalization is not implemented (future work)
- Highly heterogeneous or irregular marker distributions (e.g., CD45 in PRAD-CyCIF) violate automatic workflow assumptions and are flagged via a Kullback-Leibler (KL) divergence-based metric, requiring manual fine-tuning
- Effectiveness depends on MTI dataset quality and cell/nuclei segmentation mask quality; compromised signals cannot be recovered
- Currently optimized for within-cell normalization only; extracellular regions are not supported
- TMA-Lunaphore dataset is only available from the authors upon request, and the pixel-level (OME-TIFF) example dataset and tutorial were not yet available

## Evidence
