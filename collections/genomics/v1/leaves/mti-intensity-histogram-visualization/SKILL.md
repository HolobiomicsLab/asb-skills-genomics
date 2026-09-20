---
name: mti-intensity-histogram-visualization
description: Use when a marker's intensity distributions across samples/batches (e.g.,
  CD45 in PRAD-CyCIF, 20 samples/7 batches) look irregular, high-noise, or heterogeneous
  and must be visually inspected or manually landmarked before normalization.
when_to_use_negative:
- Input is pixel-level OME-TIFF data — the UniFORM pixel-level tutorial and example
  dataset are noted as 'being assembled and will be available soon', so this feature-level
  histogram workflow does not yet have a documented pixel counterpart.
- The marker's distributions pass the automatic workflow (no KL-divergence flag, well-behaved
  negative populations) — the automatic rigid-landmark registration handles alignment
  without manual histogram-guided landmark refinement.
- Data are mIHC chromogenic signals — UniFORM's linear normalization factors are unsuitable
  for mIHC because enzyme-catalyzed colorimetric amplification produces non-linear
  signal intensities.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: UniFORM
  role: Pipeline whose histogram-calculation and guided fine-tuning mode this visualization
    step supports; irregular distributions are flagged via a KL divergence-based metric.
  repo: https://github.com/kunlunW/UniFORM
- name: plotly
  role: Interactive plotting library used to render histogram curves so users can
    inspect distributions and manually refine landmark positions.
  repo: https://github.com/plotly/plotly.py
- name: scikit-learn
  role: Gaussian mixture model fitting to guide negative-peak landmark selection on
    markers with irregular distributions.
- name: scikit-fda
  role: Functional-data registration utilities used downstream to compute the rigid
    landmark-based shift once landmarks are chosen.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:f3dd5b404bba9a3768648f19ceec669122db848f1d9d3dd1248b701a7dac0ec4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kunlunW/UniFORM
  tools:
  - Python
  - UniFORM
  - plotly
  - scikit-fda
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mti-intensity-histogram-visualization

## Summary

Compute and render interactive histogram curves of log-transformed marker intensities per sample/batch from multiplex tissue imaging (MTI) feature tables, as the first inspection and landmark-selection step of the UniFORM normalization workflow (e.g., irregular CD45 distributions in the PRAD-CyCIF dataset).

## When to use

Use when a marker's intensity distributions across samples/batches (e.g., CD45 in PRAD-CyCIF, 20 samples/7 batches) look irregular, high-noise, or heterogeneous and must be visually inspected or manually landmarked before normalization.

## When NOT to use

- Input is pixel-level OME-TIFF data — the UniFORM pixel-level tutorial and example dataset are noted as 'being assembled and will be available soon', so this feature-level histogram workflow does not yet have a documented pixel counterpart.
- The marker's distributions pass the automatic workflow (no KL-divergence flag, well-behaved negative populations) — the automatic rigid-landmark registration handles alignment without manual histogram-guided landmark refinement.
- Data are mIHC chromogenic signals — UniFORM's linear normalization factors are unsuitable for mIHC because enzyme-catalyzed colorimetric amplification produces non-linear signal intensities.

## Inputs

- Feature-level single-cell MTI feature table (.pkl pickle or .h5ad AnnData), e.g., PRAD-CyCIF from https://zenodo.org/records/16383766
- Per-sample/batch log-transformed marker intensity values (e.g., log CD45 intensities)
- KL-divergence flags identifying markers whose distributions violate the automatic workflow assumptions

## Outputs

- Per-sample log-transformed intensity histogram distributions
- Interactive plotly histogram curves overlaying samples/batches for landmark inspection
- Manually refined negative-peak landmark positions (e.g., reference negative peak designation) feeding the rigid shift computation

## How to apply

Load the feature-level single-cell dataset (PRAD-CyCIF .pkl pickle or .h5ad AnnData from Zenodo record 16383766) with pandas/AnnData, then compute the histogram distribution of log-transformed cell intensities for each marker per sample/batch. Render the distributions as interactive plotly histogram curves so per-sample negative peaks can be visually compared and landmark positions manually refined. For markers flagged by the Kullback-Leibler (KL) divergence-based metric — i.e., whose irregularities violate the automatic rigid-landmark registration assumptions — fit a Gaussian mixture model (scikit-learn) to guide negative-peak landmark selection on the interactive plot. This visualization step precedes the rigid landmark-based shift computed via scikit-fda functional-data registration and is what makes the guided fine-tuning mode possible; the downstream normalization must preserve distribution shape and positive-population proportions.

## Related tools

- **UniFORM** (Pipeline whose histogram-calculation and guided fine-tuning mode this visualization step supports; irregular distributions are flagged via a KL divergence-based metric.) — https://github.com/kunlunW/UniFORM
- **plotly** (Interactive plotting library used to render histogram curves so users can inspect distributions and manually refine landmark positions.) — https://github.com/plotly/plotly.py
- **scikit-learn** (Gaussian mixture model fitting to guide negative-peak landmark selection on markers with irregular distributions.)
- **scikit-fda** (Functional-data registration utilities used downstream to compute the rigid landmark-based shift once landmarks are chosen.)

## Examples

```
import pandas as pd, plotly.express as px; df = pd.read_pickle('prad_cycif.pkl'); px.histogram(df, x='CD45_log', color='sample', barmode='overlay').show()
```

## Evaluation signals

- Histograms are computed on log-transformed cell intensities, not raw intensities, for each marker and each sample/batch.
- Negative peaks of the plotted curves are co-aligned across all 20 samples / 7 batches of PRAD-CyCIF after the downstream landmark-based shift is applied.
- Positive populations and overall distribution shapes are preserved after normalization (rigid shift only, no warping).
- Irregular markers targeted by this step correspond to those flagged by the KL divergence-based metric rather than arbitrary choices.
- Output overlays export as interactive plotly plots (and before/after alignment images such as cd45_landmark_alignment.png) showing per-sample curves.

## Limitations

- The guided fine-tuning is needed only because highly heterogeneous/irregular marker distributions (e.g., CD45 in PRAD-CyCIF) violate the assumptions of the automatic normalization workflow.
- Effectiveness depends on the quality of the MTI dataset and its cell/nuclei segmentation masks used to extract signals; compromised signals cannot be recovered.
- The TMA-Lunaphore dataset used in the paper is only available from the authors upon request, limiting reproduction on that platform.

## Evidence
