---
name: gmm-based-positive-cell-thresholding
description: Use when per-cell feature-level MTI data (e.g., PRAD-CyCIF .h5ad) must
  be binarized into marker-negative/positive populations using data-driven thresholds,
  or when comparing positive population percentages across local vs global threshold
  strategies and normalization conditions.
when_to_use_negative:
- Marker intensity distributions are unimodal or highly irregular (e.g., CD45 in PRAD-CyCIF),
  violating the two-Gaussian assumption of the GMM thresholding
- Data are pixel-level (OME-TIFF) rather than feature-level single-cell measurements
- Signals come from mIHC chromogenic staining, where enzyme-catalyzed amplification
  produces non-linear intensities unsuited to this linear-threshold framework
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: scikit-learn
  role: Provides GaussianMixture used to fit a two-component model to each marker's
    log-transformed intensity distribution and derive positive thresholds
- name: anndata
  role: Loads the PRAD-CyCIF feature-level single-cell data (.h5ad) from the Zenodo
    deposit
- name: UniFORM
  role: Normalizes the raw feature data (automatic rigid-landmark functional registration)
    so positive population proportions can be compared before vs after normalization
  repo: https://github.com/kunlunW/UniFORM
- name: pandas
  role: Tabulates the before-vs-after positive population comparison and saves positive_population_proportions.csv
- name: numpy
  role: Log-transforms per-cell marker intensities prior to GMM fitting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:8bbd3f9a2723af44cdb47a8857c5fa910e11da86f1d3f728d15ee60c2cb40ea7
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
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# gmm-based-positive-cell-thresholding

## Summary

Defines marker-positive cell populations in multiplex tissue imaging (MTI) single-cell data by fitting a two-component Gaussian mixture model to log-transformed per-cell marker intensities, deriving per-marker (local) thresholds and dataset-wide (global) thresholds to quantify positive population proportions before and after normalization.

## When to use

Use when per-cell feature-level MTI data (e.g., PRAD-CyCIF .h5ad) must be binarized into marker-negative/positive populations using data-driven thresholds, or when comparing positive population percentages across local vs global threshold strategies and normalization conditions.

## When NOT to use

- Marker intensity distributions are unimodal or highly irregular (e.g., CD45 in PRAD-CyCIF), violating the two-Gaussian assumption of the GMM thresholding
- Data are pixel-level (OME-TIFF) rather than feature-level single-cell measurements
- Signals come from mIHC chromogenic staining, where enzyme-catalyzed amplification produces non-linear intensities unsuited to this linear-threshold framework

## Inputs

- Feature-level single-cell MTI data in .h5ad format (e.g., PRAD-CyCIF prostate cancer whole-slide data, 20 samples across 7 batches)
- List of marker channels (e.g., ECAD and other bimodal CyCIF markers)
- Batch/sample labels for per-batch proportion tabulation

## Outputs

- Per-marker (local) positive thresholds derived from the two-component GMM fit on log-transformed intensities
- Positive population proportions per batch under local and global threshold strategies, for raw and normalized conditions
- Comparison table saved as positive_population_proportions.csv

## How to apply

Load feature-level single-cell data (.h5ad, e.g., the PRAD-CyCIF deposit on Zenodo) with anndata, then log-transform per-cell marker intensities and fit a two-component scikit-learn GaussianMixture to each marker's distribution to identify the negative and positive modes and derive per-marker (local) positive thresholds. Compute baseline per-marker positive population proportions per batch under both the GMM-derived local thresholds and dataset-wide (global) thresholds. After applying normalization (e.g., UniFORM's automatic rigid-landmark functional registration), recompute the proportions with the same thresholds and tabulate before-vs-after changes. For bimodal markers such as ECAD, a correctly applied thresholding should show minimal positive population change after UniFORM normalization, since UniFORM preserves positive-population proportions while aligning negative populations.

## Related tools

- **scikit-learn** (Provides GaussianMixture used to fit a two-component model to each marker's log-transformed intensity distribution and derive positive thresholds)
- **anndata** (Loads the PRAD-CyCIF feature-level single-cell data (.h5ad) from the Zenodo deposit)
- **UniFORM** (Normalizes the raw feature data (automatic rigid-landmark functional registration) so positive population proportions can be compared before vs after normalization) — https://github.com/kunlunW/UniFORM
- **pandas** (Tabulates the before-vs-after positive population comparison and saves positive_population_proportions.csv)
- **numpy** (Log-transforms per-cell marker intensities prior to GMM fitting)

## Examples

```
from sklearn.mixture import GaussianMixture; import numpy as np; gmm = GaussianMixture(n_components=2).fit(np.log10(marker_intensities).reshape(-1, 1))
```

## Evaluation signals

- Each marker's two-component GaussianMixture converges and identifies distinct negative and positive modes in the log-transformed intensity histogram
- For bimodal markers such as ECAD, the positive population change between local-threshold (pre-normalization) and global-threshold (post-normalization) proportions after UniFORM normalization is minimal, matching the paper's reported positive population preservation
- The output table positive_population_proportions.csv contains all markers, both threshold strategies (local and global), and both raw and normalized conditions
- Dataset-wide (global) thresholds computed per marker fall between the negative and positive GMM mode positions in log-intensity space

## Limitations

- Highly heterogeneous/irregular marker distributions (e.g., CD45 in PRAD-CyCIF) violate the assumptions of the automatic workflow and require manual fine-tuning
- Effectiveness depends on the quality of the MTI dataset and its cell/nuclei segmentation masks used to extract signals; compromised signals cannot be recovered
- Bimodality-based thresholding assumes a separable negative-positive mode structure, so markers with broad right-skewed or mixed expression may yield unreliable thresholds

## Evidence

- [other] GMM two-component fit on log intensities: [evidence span withheld — anchor status: not_found]
