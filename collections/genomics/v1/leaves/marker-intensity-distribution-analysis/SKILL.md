---
name: marker-intensity-distribution-analysis
description: Use when comparing marker-positive population percentages before and
  after normalization on feature-level MTI data, especially testing whether local
  (GMM-derived) vs global thresholds give consistent percentages for markers like
  ECAD.
when_to_use_negative:
- Input is pixel-level imaging data or OME-TIFF without segmentation masks — the feature-level
  workflow requires single-cell intensity tables
- Marker distributions are highly heterogeneous/irregular (e.g., CD45 in PRAD-CyCIF),
  violating the assumptions of the automatic workflow and requiring guided fine-tuning
  instead
- Data are mIHC/chromogenic, where linear normalization factors are unsuitable due
  to non-linear enzymatic signal amplification
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_2269
tools:
- name: UniFORM
  role: Performs the automatic rigid-landmark functional registration normalization
    whose effect on positive populations is being quantified
  repo: https://github.com/kunlunW/UniFORM
- name: anndata
  role: Loads the .h5ad feature-level single-cell dataset from the Zenodo deposit
- name: scikit-learn
  role: Fits two-component GaussianMixture models to log-transformed marker intensities
    to derive local positive thresholds
- name: pandas
  role: Tabulates and saves the positive population comparison table as positive_population_proportions.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:82df2a3b447c96ee340a5550a1b0ee6c9b9d55634bf752bac4fca2c0b1c7dd28
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.47368421052631576
n_claims: 19
---

# marker-intensity-distribution-analysis

## Summary

Quantifies marker-positive cell population percentages on multiplex tissue imaging single-cell data (e.g., PRAD-CyCIF .h5ad) using two-component Gaussian mixture model thresholds, and evaluates how normalization methods such as UniFORM preserve positive populations when moving from local per-marker thresholds to dataset-wide global thresholds.

## When to use

Use when comparing marker-positive population percentages before and after normalization on feature-level MTI data, especially testing whether local (GMM-derived) vs global thresholds give consistent percentages for markers like ECAD.

## When NOT to use

- Input is pixel-level imaging data or OME-TIFF without segmentation masks — the feature-level workflow requires single-cell intensity tables
- Marker distributions are highly heterogeneous/irregular (e.g., CD45 in PRAD-CyCIF), violating the assumptions of the automatic workflow and requiring guided fine-tuning instead
- Data are mIHC/chromogenic, where linear normalization factors are unsuitable due to non-linear enzymatic signal amplification

## Inputs

- Feature-level single-cell MTI data in AnnData .h5ad format (e.g., PRAD-CyCIF from Zenodo https://zenodo.org/records/16383766)
- Per-cell marker intensity matrix with batch/sample annotations (e.g., 20 PRAD-CyCIF samples across 7 batches)
- List of markers to threshold (e.g., ECAD and other bimodal markers)

## Outputs

- positive_population_proportions.csv — table of per-marker positive population percentages for each threshold strategy (local vs global) and each condition (raw vs normalized)
- Per-marker GMM-derived local positive thresholds from two-component Gaussian mixture fits
- Before-vs-after positive population change values per normalization method

## How to apply

Load the feature-level single-cell data (.h5ad, e.g., the PRAD-CyCIF deposit on Zenodo record 16383766) with anndata, log-transform per-cell marker intensities, and fit a two-component sklearn GaussianMixture to each marker's intensity distribution to identify negative and positive modes and derive per-marker (local) positive thresholds. Compute baseline positive population proportions per batch using both the GMM-derived local thresholds and dataset-wide global thresholds. Apply UniFORM's automatic rigid-landmark functional registration normalization — which aligns biologically invariant negative population peaks to a reference distribution via maximum cross-correlation — to the raw feature data. Recompute positive population proportions on normalized data with the same local and global thresholds and tabulate before-vs-after changes per normalization condition (e.g., UniFORM, MxNorm registration, Z score, ComBat, mean division). The rationale is that aligning negative populations removes technical batch variation while preserving tissue-specific positive-population proportions.

## Related tools

- **UniFORM** (Performs the automatic rigid-landmark functional registration normalization whose effect on positive populations is being quantified) — https://github.com/kunlunW/UniFORM
- **anndata** (Loads the .h5ad feature-level single-cell dataset from the Zenodo deposit)
- **scikit-learn** (Fits two-component GaussianMixture models to log-transformed marker intensities to derive local positive thresholds)
- **pandas** (Tabulates and saves the positive population comparison table as positive_population_proportions.csv)

## Examples

```
import anndata, numpy as np; from sklearn.mixture import GaussianMixture; adata = anndata.read_h5ad('PRAD-CyCIF.h5ad'); gmm = GaussianMixture(n_components=2).fit(np.log1p(adata[:, 'ECAD'].X))
```

## Evaluation signals

- For bimodal markers such as ECAD, the positive population change after UniFORM normalization is minimal, confirming positive population preservation as reported in the paper
- The output table contains all markers, both threshold strategies (GMM local and dataset-wide global), and both raw and normalized conditions
- Log-transformed marker intensity histograms show a clear two-mode separation supporting a valid two-component GaussianMixture fit
- Negative population peaks align across batches after normalization while distribution shapes (positive modes) are preserved

## Limitations

- Highly heterogeneous/irregular marker distributions (e.g., CD45 in PRAD-CyCIF) violate the automatic workflow assumptions and are flagged via a Kullback-Leibler divergence-based metric, requiring manual guided fine-tuning
- UniFORM effectiveness depends on the quality of the MTI dataset and the cell/nuclei segmentation masks used to extract signals; compromised signals cannot be recovered
- Linear normalization factors are unsuitable for mIHC due to non-linear chromogenic signal amplification, so this positive-population analysis does not transfer to mIHC data

## Evidence
