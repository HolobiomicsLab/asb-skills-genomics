---
name: positive-population-preservation-quantification
description: Use when benchmarking an MTI normalization method on single-cell feature
  data and you need to verify that biological signal fidelity — the fraction of marker-positive
  cells — is preserved between local and global thresholding strategies.
when_to_use_negative:
- Input is raw microscopy images (OME-TIFF) rather than feature-level single-cell
  intensity tables — use pixel-level normalization and segmentation-mask-based analysis
  instead
- Marker distributions are highly heterogeneous or irregular (e.g., CD45 in PRAD-CyCIF),
  violating two-component GaussianMixture assumptions and requiring manual fine-tuning
  of thresholds
- Data are mIHC (chromogenic) samples where non-linear signal amplification makes
  linear normalization factors unsuitable
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_2640
tools:
- name: UniFORM
  role: Normalization method whose positive-population preservation is quantified;
    applies automatic rigid-landmark functional registration to align negative peaks
  repo: https://github.com/kunlunW/UniFORM
- name: scikit-learn
  role: GaussianMixture fits two-component models to log-transformed marker intensities
    to derive local positive thresholds
- name: anndata
  role: Loads the PRAD-CyCIF feature-level .h5ad single-cell dataset
- name: pandas
  role: Tabulates and saves the positive population comparison table as CSV
- name: MxNorm
  role: Comparison normalization method (registration option) benchmarked against
    UniFORM on positive population change
  repo: https://github.com/ColemanRHarris/mxnorm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:c1e12c320674becf5ef7d0bdb2e95ac195c437f686a3e519227a2960c7c4621c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kunlunW/UniFORM
  tools:
  - MxNorm
  - Python
  - UniFORM
  - anndata
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# Positive-population preservation quantification

## Summary

Quantifies how much each normalization method (UniFORM, MxNorm, Z score, ComBat, mean division) changes marker-positive cell population proportions when moving from local per-sample GMM thresholds to dataset-wide global thresholds, on PRAD-CyCIF single-cell .h5ad data. A good method preserves positive populations (e.g., ECAD) despite batch correction.

## When to use

Use when benchmarking an MTI normalization method on single-cell feature data and you need to verify that biological signal fidelity — the fraction of marker-positive cells — is preserved between local and global thresholding strategies.

## When NOT to use

- Input is raw microscopy images (OME-TIFF) rather than feature-level single-cell intensity tables — use pixel-level normalization and segmentation-mask-based analysis instead
- Marker distributions are highly heterogeneous or irregular (e.g., CD45 in PRAD-CyCIF), violating two-component GaussianMixture assumptions and requiring manual fine-tuning of thresholds
- Data are mIHC (chromogenic) samples where non-linear signal amplification makes linear normalization factors unsuitable

## Inputs

- Feature-level single-cell MTI data in .h5ad (AnnData) format, e.g., PRAD-CyCIF from https://zenodo.org/records/16383766
- Log-transformed per-cell marker intensity distributions for each marker (e.g., ECAD)
- Batch/sample identifiers to compute per-batch positive proportions

## Outputs

- positive_population_proportions.csv containing all markers, both threshold strategies (local GMM-derived and global), and raw vs. normalized conditions
- Per-marker positive population change values (before-vs-after normalization) for each normalization method

## How to apply

Load the PRAD-CyCIF feature-level single-cell data (.h5ad) from the Zenodo deposit with anndata, log-transform per-cell marker intensities, and fit a two-component scikit-learn GaussianMixture to each marker's distribution to derive per-marker (local) positive thresholds. Compute baseline per-marker positive population proportions per batch under both GMM-derived local thresholds and dataset-wide (global) thresholds. Apply the normalization method under test (e.g., UniFORM's automatic rigid-landmark functional registration) to the raw feature data, recompute positive proportions with the same local and global thresholds, and tabulate before-vs-after changes. A minimal change for bimodal markers such as ECAD after UniFORM normalization indicates positive population preservation, consistent with UniFORM's design of aligning biologically invariant negative populations while sparing positive-population biology. Save the full comparison table as positive_population_proportions.csv.

## Related tools

- **UniFORM** (Normalization method whose positive-population preservation is quantified; applies automatic rigid-landmark functional registration to align negative peaks) — https://github.com/kunlunW/UniFORM
- **scikit-learn** (GaussianMixture fits two-component models to log-transformed marker intensities to derive local positive thresholds)
- **anndata** (Loads the PRAD-CyCIF feature-level .h5ad single-cell dataset)
- **pandas** (Tabulates and saves the positive population comparison table as CSV)
- **MxNorm** (Comparison normalization method (registration option) benchmarked against UniFORM on positive population change) — https://github.com/ColemanRHarris/mxnorm

## Examples

```
import anndata as ad; from sklearn.mixture import GaussianMixture; adata = ad.read_h5ad('prad_cycif.h5ad'); gmm = GaussianMixture(n_components=2).fit(np.log1p(adata[:, 'ECAD'].X))
```

## Evaluation signals

- Positive population change for bimodal markers such as ECAD after UniFORM normalization is minimal (positive populations preserved), consistent with the paper's reported preservation
- The output table contains all markers, both threshold strategies (local and global), and both raw and normalized conditions
- Per-batch positive proportions computed with GMM-derived local thresholds reproduce expected two-component (negative/positive mode) separations on log-transformed intensities
- Comparison across methods shows UniFORM with smaller positive population changes than MxNorm registration, Z score, ComBat, and mean division

## Limitations

- Highly heterogeneous/irregular marker distributions (e.g., CD45 in PRAD-CyFIF) violate automatic workflow assumptions; irregularities are flagged with a Kullback-Leibler (KL) divergence-based metric and require manual fine-tuning
- UniFORM's effectiveness depends on the quality of the MTI dataset and its cell/nuclei segmentation masks; compromised signals cannot be recovered
- No software version numbers are stated for the tools used, which may affect exact reproducibility

## Evidence
