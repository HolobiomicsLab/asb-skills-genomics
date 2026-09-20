---
name: multiplex-imaging-feature-histogram-estimation
description: Use when starting a UniFORM feature-level normalization run on single-cell
  MTI data (e.g., PRAD-CyCIF .pkl or .h5ad) and per-marker intensity distributions
  must be made comparable across batches prior to registration.
when_to_use_negative:
- Data are pixel-level OME-TIFF images rather than a single-cell feature table — pixel-level
  normalization requires nucleus/cell segmentation masks and a different pipeline
  branch
- Data are mIHC (chromogenic) samples — linear shift/scale normalization factors are
  unsuitable due to non-linear enzyme-catalyzed signal amplification
- The input is already a normalized feature matrix or the goal is batch correction
  by mean division/Z score/ComBat rather than distribution alignment
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: UniFORM
  role: Pipeline whose feature-level workflow begins with histogram calculation of
    log-transformed cell intensities per marker
  repo: https://github.com/kunlunW/UniFORM
- name: numpy
  role: Computes the histogram distribution of log-transformed cell mean intensities
    over the shared global range
- name: scikit-fda
  role: Converts each histogram into functional data objects for the subsequent registration
    step
- name: anndata
  role: Loads .h5ad feature-level single-cell input data
- name: pandas
  role: Loads .pkl (pickle) feature-level single-cell input data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:ce314deefd66f700061954c4543c9d83b944cb56cde781fd003b608ce20fed98
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# multiplex-imaging-feature-histogram-estimation

## Summary

First stage of UniFORM's feature-level normalization: compute per-marker histogram distributions of log-transformed single-cell mean intensities over a shared global range so that distributions from different MTI batches are directly comparable before functional data registration.

## When to use

Use when starting a UniFORM feature-level normalization run on single-cell MTI data (e.g., PRAD-CyCIF .pkl or .h5ad) and per-marker intensity distributions must be made comparable across batches prior to registration.

## When NOT to use

- Data are pixel-level OME-TIFF images rather than a single-cell feature table — pixel-level normalization requires nucleus/cell segmentation masks and a different pipeline branch
- Data are mIHC (chromogenic) samples — linear shift/scale normalization factors are unsuitable due to non-linear enzyme-catalyzed signal amplification
- The input is already a normalized feature matrix or the goal is batch correction by mean division/Z score/ComBat rather than distribution alignment

## Inputs

- Feature-level single-cell MTI dataset (*.pkl via pandas or *.h5ad via anndata), e.g. PRAD-CyCIF from Zenodo record 16383766
- Per-cell mean intensity values for each marker (log-transformed before histogramming)
- Shared global histogram range definition covering all markers/batches

## Outputs

- Per-marker, per-batch histogram distributions of log-transformed cell mean intensities on a shared global range
- Functional data curve objects (scikit-fda) ready for rigid-landmark registration

## How to apply

Load the feature-level single-cell dataset (PRAD-CyCIF *.pkl via pandas/pickle or *.h5ad via anndata, from Zenodo record 16383766) in a Python 3.10 environment with the UniFORM requirements installed. For each marker, log-transform the per-cell mean intensities and compute the histogram distribution with numpy over a shared global range — the shared binning is essential so histograms from different batches (e.g., the 7 PRAD-CyCIF batches) are comparable. These histograms then serve as the input to the next stage, where each is converted into a functional data curve (scikit-fda) and aligned to a reference distribution via FFT-based maximum cross-correlation rigid-landmark registration. Rationale: MTI intensity distributions are right-skewed and heterogeneous across batches due to staining/imaging variation, and a common-range histogram representation enables distribution-level alignment without assuming Gaussianity (unlike Z score or ComBat). Markers with highly heterogeneous or irregular distributions (e.g., CD45 in PRAD-CyCIF) will be flagged downstream by a KL divergence-based metric and may need guided fine-tuning instead of the automatic path.

## Related tools

- **UniFORM** (Pipeline whose feature-level workflow begins with histogram calculation of log-transformed cell intensities per marker) — https://github.com/kunlunW/UniFORM
- **numpy** (Computes the histogram distribution of log-transformed cell mean intensities over the shared global range)
- **scikit-fda** (Converts each histogram into functional data objects for the subsequent registration step)
- **anndata** (Loads .h5ad feature-level single-cell input data)
- **pandas** (Loads .pkl (pickle) feature-level single-cell input data)

## Examples

```
import pandas as pd, numpy as np; df = pd.read_pickle('PRAD-CyCIF_feature_data.pkl'); log_int = np.log(df[markers].values); hist, edges = np.histogram(log_int[:, i], range=GLOBAL_RANGE, bins=N_BINS)
```

## Evaluation signals

- Histograms for all markers/batches are computed on the identical global range and binning so distributions are directly overlayable
- Each histogram converts cleanly to a scikit-fda functional data object without NaNs or mismatched grid points
- Downstream registration aligns the biologically invariant negative-population peaks while preserving overall distribution shape (no warping)
- Positive-population proportions are preserved and mutually exclusive markers remain mutually exclusive after the normalization factors are applied
- Irregular/heterogeneous markers (e.g., CD45 in PRAD-CyCIF) are flagged by the KL divergence-based metric rather than silently normalized

## Limitations

- Highly heterogeneous or irregular marker distributions violate the assumptions of the automatic workflow and require manual guided fine-tuning
- Effectiveness depends on MTI dataset quality and segmentation-mask quality; compromised signals cannot be recovered
- Linear normalization factors estimated downstream are unsuitable for mIHC due to non-linear chromogenic signal amplification

## Evidence
