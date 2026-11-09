---
name: landmark-based-distribution-registration
description: Use when normalizing feature-level MTI single-cell intensity data (e.g.,
  .h5ad/.pkl) across batches, and irregular or heterogeneous marker distributions
  (flagged by KL divergence) require landmark alignment or manual fine-tuning.
when_to_use_negative:
- 'Input is mIHC (multiplex immunohistochemistry) data: linear normalization factors
  are unsuitable because chromogenic detection produces non-linear signal intensities
  from enzyme-catalyzed colorimetric amplification'
- 'Input is pixel-level data without nucleus/cell segmentation masks, or signals from
  compromised segmentation: UniFORM normalizes pixels exclusively within masks and
  cannot recover compromised signals'
- The input is already a normalized/aligned feature table — normalization factors
  estimated from landmark registration would distort valid distributions
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_0091
tools:
- name: UniFORM
  role: Non-parametric Python pipeline implementing automatic rigid-landmark registration
    and the guided fine-tuning mode for feature- and pixel-level MTI normalization
  repo: https://github.com/kunlunW/UniFORM
- name: scikit-fda
  role: Converts histograms to functional data curves and performs rigid-landmark
    functional registration for signal alignment
- name: scikit-learn
  role: Gaussian mixture model fit to guide negative-peak landmark selection during
    guided fine-tuning (e.g., for irregular CD45 distributions)
- name: scipy
  role: FFT-based maximum cross-correlation algorithm used to align each marker's
    distribution to the reference
- name: plotly
  role: Interactive histogram curves on which users inspect irregular distributions
    and manually refine landmark positions
  repo: https://github.com/plotly/plotly.py
- name: anndata
  role: Reading/writing the .h5ad feature-level single-cell input and normalized output
    tables
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:69796045af3a6d689e9b61ed5c4935c592ca261afc24d266812c63c62a675169
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
  - plotly
  - scikit-fda
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# landmark-based-distribution-registration

## Summary

UniFORM's rigid landmark registration normalizes multiplex tissue imaging (MTI) marker intensity distributions by aligning biologically invariant negative-population peaks across samples/batches, with a GMM-assisted guided fine-tuning mode (e.g., CD45 in PRAD-CyCIF) for markers that violate the automatic workflow's assumptions.

## When to use

Use when normalizing feature-level MTI single-cell intensity data (e.g., .h5ad/.pkl) across batches, and irregular or heterogeneous marker distributions (flagged by KL divergence) require landmark alignment or manual fine-tuning.

## When NOT to use

- Input is mIHC (multiplex immunohistochemistry) data: linear normalization factors are unsuitable because chromogenic detection produces non-linear signal intensities from enzyme-catalyzed colorimetric amplification
- Input is pixel-level data without nucleus/cell segmentation masks, or signals from compromised segmentation: UniFORM normalizes pixels exclusively within masks and cannot recover compromised signals
- The input is already a normalized/aligned feature table — normalization factors estimated from landmark registration would distort valid distributions

## Inputs

- Feature-level single-cell MTI dataset in .pkl (pickle/pandas) or .h5ad (AnnData) format, e.g., PRAD-CyCIF from Zenodo record 16383766
- Log-transformed per-cell mean intensity values per marker and per sample/batch
- Segmentation-derived cell/nuclei signal extractions (prerequisite for MTI data quality)

## Outputs

- Normalized feature matrix with per-sample/per-marker rigid shift factors applied (e.g., normalized_feature_data.pkl or cd45_landmark_normalized.h5ad)
- Overlaid before/after histogram curves showing co-aligned negative peaks (e.g., cd45_landmark_alignment.png or interactive plotly curves)
- Per-marker, per-sample normalization factors estimated from rigid-landmark alignment

## How to apply

Load the feature-level single-cell table (PRAD-CyCIF .pkl or .h5ad from Zenodo record 16383766) and compute log-transformed per-marker intensity histograms over a shared global range with numpy so batch distributions are comparable. Convert histograms into scikit-fda functional data objects and perform automatic rigid-landmark registration, using the maximum cross-correlation algorithm (scipy FFT-based) to align each marker's distribution to a reference, then apply the extracted shift/scale normalization factors to the raw per-cell mean intensities. For irregular markers (e.g., CD45, automatically flagged by a Kullback-Leibler divergence-based metric), use the guided fine-tuning mode: render interactive plotly histogram curves, fit a Gaussian mixture model (scikit-learn) to guide negative-peak landmark selection, manually refine landmark positions, designate a reference negative peak, and compute the per-sample rigid shift. Apply the shift so distribution shape and positive-population proportions are preserved, and export the normalized table (e.g., cd45_landmark_normalized.h5ad) with overlaid before/after histograms.

## Related tools

- **UniFORM** (Non-parametric Python pipeline implementing automatic rigid-landmark registration and the guided fine-tuning mode for feature- and pixel-level MTI normalization) — https://github.com/kunlunW/UniFORM
- **scikit-fda** (Converts histograms to functional data curves and performs rigid-landmark functional registration for signal alignment)
- **scikit-learn** (Gaussian mixture model fit to guide negative-peak landmark selection during guided fine-tuning (e.g., for irregular CD45 distributions))
- **scipy** (FFT-based maximum cross-correlation algorithm used to align each marker's distribution to the reference)
- **plotly** (Interactive histogram curves on which users inspect irregular distributions and manually refine landmark positions) — https://github.com/plotly/plotly.py
- **anndata** (Reading/writing the .h5ad feature-level single-cell input and normalized output tables)

## Examples

```
git clone https://github.com/kunlunW/UniFORM.git
```

## Evaluation signals

- Negative-population peaks align to the reference distribution across all samples/batches (e.g., 20 PRAD-CyCIF samples across 7 batches) after the rigid shift
- Overall distribution shape is preserved with no warping of curve shape; positive-population proportions are unchanged after normalization
- Mutually exclusive markers remain mutually exclusive after normalization
- kBET batch mixing and silhouette scores improve relative to raw and competitor methods (MxNorm registration, Z score, ComBat, mean division)
- For guided fine-tuning, CD45 negative peaks co-align across all 20 samples/7 batches while positive populations remain preserved in the exported before/after histogram overlay

## Limitations

- Linear normalization factors are unsuitable for mIHC due to non-linear chromogenic signal amplification; a warping function for non-linear mIHC normalization is not implemented (future work)
- Irregular, highly heterogeneous marker distributions (e.g., CD45 in PRAD-CyCIF) violate the assumptions of the automatic workflow and require the guided fine-tuning option
- Effectiveness depends on the quality of the MTI dataset and its cell/nuclei segmentation masks; compromised signals cannot be recovered
- UniFORM is currently optimized for within-cell normalization; extracellular regions are not supported without modification

## Evidence
