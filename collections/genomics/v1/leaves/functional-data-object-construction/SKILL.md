---
name: functional-data-object-construction
description: 'Use when batch-effect-corrected feature-level MTI data requires aligning
  per-marker intensity distributions: histograms have been computed over a shared
  global range and must be registered as functional curves before extracting normalization
  factors.'
when_to_use_negative:
- The data are pixel-level (OME-TIFF) rather than a single-cell feature table — pixel-level
  normalization additionally requires nucleus/cell segmentation masks to exclude non-cell
  regions
- 'The data are mIHC samples: chromogenic detection produces non-linear signal amplification,
  so UniFORM''s linear normalization factors are unsuitable and a warping function
  is not implemented'
- A marker has a highly heterogeneous/irregular distribution (e.g., CD45 in PRAD-CyCIF)
  flagged by the KL divergence-based metric — the automatic workflow's assumptions
  are violated and guided manual fine-tuning is required
edam_topics:
- http://edamontology.org/topic_3070
- http://edamontology.org/topic_2269
tools:
- name: UniFORM
  role: Pipeline that implements histogram calculation, rigid-landmark functional
    registration, and normalization of MTI data
  repo: https://github.com/kunlunW/UniFORM
- name: scikit-fda
  role: Converts histograms into functional data objects and performs the rigid-landmark
    registration
- name: scipy
  role: Provides the FFT-based maximum cross-correlation algorithm used to align each
    marker's distribution to the reference
- name: numpy
  role: Computes histogram distributions of log-transformed cell mean intensities
    over a shared global range
- name: anndata
  role: Loads the .h5ad feature-level single-cell input
- name: pandas
  role: Loads the .pkl (pickle) feature-level single-cell input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:ef6aca087f3aaca3369baa3a382857a3088ad3258222bdaf5f4c25a1b5eca6cf
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3070
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# functional-data-object-construction

## Summary

Converts per-marker histograms of log-transformed cell mean intensities from MTI datasets (e.g., PRAD-CyCIF .pkl/.h5ad feature tables) into functional data curves with scikit-fda, then aligns them to a reference distribution via rigid-landmark registration using a maximum cross-correlation algorithm. This is the middle stage of UniFORM's histogram -> registration -> normalization pipeline and yields the shift/scale factors applied downstream.

## When to use

Use when batch-effect-corrected feature-level MTI data requires aligning per-marker intensity distributions: histograms have been computed over a shared global range and must be registered as functional curves before extracting normalization factors.

## When NOT to use

- The data are pixel-level (OME-TIFF) rather than a single-cell feature table — pixel-level normalization additionally requires nucleus/cell segmentation masks to exclude non-cell regions
- The data are mIHC samples: chromogenic detection produces non-linear signal amplification, so UniFORM's linear normalization factors are unsuitable and a warping function is not implemented
- A marker has a highly heterogeneous/irregular distribution (e.g., CD45 in PRAD-CyCIF) flagged by the KL divergence-based metric — the automatic workflow's assumptions are violated and guided manual fine-tuning is required

## Inputs

- Feature-level single-cell dataset (*.pkl or *.h5ad) from Zenodo record 16383766 (PRAD-CyCIF)
- Histogram distributions of log-transformed cell mean intensities per marker, computed over a shared global range with numpy

## Outputs

- scikit-fda functional data curve objects per marker/batch
- Rigid-landmark aligned distributions matched to a reference distribution
- Per-marker, per-batch shift/scale normalization factors
- normalized_feature_data.pkl (normalized feature matrix)

## How to apply

Load the single-cell feature table (PRAD-CyCIF from Zenodo record 16383766, as *.pkl via pandas or *.h5ad via anndata) in a Python 3.10 environment with UniFORM requirements installed. For each marker, compute with numpy the histogram of log-transformed cell mean intensities over a shared global range so distributions from different batches are comparable. Convert each histogram into a scikit-fda functional data object, then apply automatic rigid-landmark registration, using scipy's FFT-based maximum cross-correlation algorithm to align each marker's distribution to an appropriate reference distribution. The rationale for rigid (shift/scale only, no warping) alignment is to preserve distribution shape and positive-population proportions while matching the biologically invariant negative-population peaks. Finally, extract the per-marker, per-batch shift/scale normalization factors estimated from the alignment and apply them to the raw intensities to produce normalized_feature_data.pkl.

## Related tools

- **UniFORM** (Pipeline that implements histogram calculation, rigid-landmark functional registration, and normalization of MTI data) — https://github.com/kunlunW/UniFORM
- **scikit-fda** (Converts histograms into functional data objects and performs the rigid-landmark registration)
- **scipy** (Provides the FFT-based maximum cross-correlation algorithm used to align each marker's distribution to the reference)
- **numpy** (Computes histogram distributions of log-transformed cell mean intensities over a shared global range)
- **anndata** (Loads the .h5ad feature-level single-cell input)
- **pandas** (Loads the .pkl (pickle) feature-level single-cell input)

## Examples

```
git clone https://github.com/kunlunW/UniFORM.git
```

## Evaluation signals

- Aligned distributions match the reference at their biologically invariant negative-population peaks
- Overall distribution shape is preserved with no warping of the curve (rigid shift/scale only)
- Positive-population proportions are preserved after registration and normalization
- Markers that are mutually exclusive remain mutually exclusive after normalization
- Shift/scale normalization factors are extracted for every marker and batch and, when applied, produce normalized_feature_data.pkl

## Limitations

- Highly heterogeneous or irregular marker distributions (e.g., CD45 in PRAD-CyCIF) violate the automatic workflow's assumptions and are flagged via a Kullback-Leibler divergence-based metric, requiring guided manual fine-tuning
- Effectiveness depends on the quality of the MTI dataset and its cell/nuclei segmentation masks; compromised signals cannot be recovered
- Linear normalization factors are unsuitable for mIHC due to non-linear chromogenic signal amplification; a non-linear warping function is future work

## Evidence
