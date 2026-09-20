---
name: interactive-plotly-curve-annotation
description: Use when automatic rigid-landmark normalization is flagged as insufficient
  (e.g., by a KL divergence metric) for a marker with irregular or heterogeneous intensity
  distributions, and negative-peak landmarks must be selected or refined by hand on
  interactive histogram curves.
when_to_use_negative:
- The automatic rigid-landmark functional registration workflow already produces well-aligned
  negative peaks (irregularities are automatically flagged via a KL divergence-based
  metric) — manual annotation is only for flagged markers
- Input is pixel-level OME-TIFF data — the UniFORM Pixel-Level tutorial and example
  dataset are noted as 'Being assembled and will be available soon'
- The marker distributions come from mIHC, where chromogenic amplification makes linear
  (shift-based) normalization factors unsuitable
edam_operation: http://edamontology.org/operation_0337
tools:
- name: plotly
  role: Renders interactive histogram curves of log-transformed marker intensities
    so users can inspect and manually refine negative-peak landmark positions
  repo: https://github.com/plotly/plotly.py
- name: UniFORM
  role: Hosts the guided fine-tuning mode and applies the manually guided rigid landmark-based
    shift to the feature-level data
  repo: https://github.com/kunlunW/UniFORM
- name: scikit-learn
  role: Gaussian mixture model fitting to guide negative-peak landmark selection before
    manual refinement
- name: scikit-fda
  role: Functional-data registration utilities used to compute the rigid landmark-based
    shift once landmarks are fixed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:e956bf71cab1fa01a714a88a3add55b5d6c399d9780d3d848853b9fdfc2dcba4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/plotly/plotly.py
  tools:
  - Python
  - UniFORM
  - plotly
  - scikit-fda
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# interactive-plotly-curve-annotation

## Summary

An interactive plotly-based curve inspection and annotation workflow from UniFORM in which log-transformed marker intensity histograms are rendered as per-sample curves and users manually refine negative-peak landmark positions to guide the rigid landmark-based registration for hard-to-normalize markers such as CD45 in PRAD-CyCIF.

## When to use

Use when automatic rigid-landmark normalization is flagged as insufficient (e.g., by a KL divergence metric) for a marker with irregular or heterogeneous intensity distributions, and negative-peak landmarks must be selected or refined by hand on interactive histogram curves.

## When NOT to use

- The automatic rigid-landmark functional registration workflow already produces well-aligned negative peaks (irregularities are automatically flagged via a KL divergence-based metric) — manual annotation is only for flagged markers
- Input is pixel-level OME-TIFF data — the UniFORM Pixel-Level tutorial and example dataset are noted as 'Being assembled and will be available soon'
- The marker distributions come from mIHC, where chromogenic amplification makes linear (shift-based) normalization factors unsuitable

## Inputs

- PRAD-CyCIF feature-level single-cell dataset as .pkl pickle or .h5ad AnnData (https://zenodo.org/records/16383766)
- Log-transformed per-sample marker intensity values (e.g., CD45) extracted from the feature table
- GMM-fitted candidate negative-peak landmark positions per sample/batch

## Outputs

- Interactive plotly histogram curves per sample/batch for landmark inspection and manual refinement
- Manually refined negative-peak landmark positions aligned to a designated reference peak
- Per-sample rigid shift normalization factors and normalized feature table (cd45_landmark_normalized.h5ad)
- Overlaid before/after histogram curve export (cd45_landmark_alignment.png)

## How to apply

Load the PRAD-CyCIF feature-level single-cell table (.pkl pickle or .h5ad AnnData from Zenodo record 16383766) and compute the log-transformed CD45 intensity histogram per sample/batch, rendering each distribution as an interactive plotly histogram curve for visual inspection. Identify samples whose distributions are irregular (these are automatically flagged with a Kullback-Leibler divergence-based metric) and fit a Gaussian mixture model with scikit-learn to propose negative-peak landmark candidates. On the interactive plotly plot, manually drag or specify landmark positions to correct the GMM-guided negative-peak selection, then designate a reference negative peak. Compute a rigid (shift-only) normalization factor per sample that aligns each selected negative-peak landmark to the reference, using scikit-fda functional-data registration utilities, and apply this shift to the raw intensities so that the overall distribution shape and positive-population proportions are preserved. Export the normalized table (cd45_landmark_normalized.h5ad) and overlaid before/after histogram curves (cd45_landmark_alignment.png) to verify that negative peaks co-align across all 20 samples/7 batches while positive populations remain unchanged.

## Related tools

- **plotly** (Renders interactive histogram curves of log-transformed marker intensities so users can inspect and manually refine negative-peak landmark positions) — https://github.com/plotly/plotly.py
- **UniFORM** (Hosts the guided fine-tuning mode and applies the manually guided rigid landmark-based shift to the feature-level data) — https://github.com/kunlunW/UniFORM
- **scikit-learn** (Gaussian mixture model fitting to guide negative-peak landmark selection before manual refinement)
- **scikit-fda** (Functional-data registration utilities used to compute the rigid landmark-based shift once landmarks are fixed)

## Examples

```
import plotly.graph_objects as go; fig = go.Figure(); [fig.add_trace(go.Histogram(x=adata[adata.obs.batch==b, 'CD45'].X.flatten())) for b in batches]; fig.show()
```

## Evaluation signals

- Negative peaks co-align to the reference landmark across all 20 PRAD-CyCIF samples and 7 batches after applying the rigid shift
- Overall distribution shape is preserved — the normalization is a rigid shift only, so positive-population proportions should be unchanged
- Interactive plotly curves show per-sample negative peaks on the same log-intensity axis before vs. after normalization (cd45_landmark_alignment.png)
- Normalized output is a valid .h5ad AnnData file (cd45_landmark_normalized.h5ad) with the same cell x feature schema as the input table

## Limitations

- Manual fine-tuning is only needed because irregular distributions (e.g., CD45 in PRAD-CyCIF) violate the assumptions of the automatic workflow; irregularities are detected via a Kullback-Leibler (KL) divergence-based metric
- Linear normalization factors (including rigid shifts) are unsuitable for mIHC due to non-linear chromogenic signal amplification; a warping function for non-linear mIHC normalization is not implemented
- The guided workflow operates on feature-level data; the pixel-level tutorial and OME-TIFF example dataset are not yet available, and no software version numbers are stated for the tools used

## Evidence
