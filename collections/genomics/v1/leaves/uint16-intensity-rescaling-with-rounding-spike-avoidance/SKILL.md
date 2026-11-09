---
name: uint16-intensity-rescaling-with-rounding-spike-avoidance
description: Use when writing pixel-level-normalized multiplex tissue imaging data
  back to a 16-bit OME-TIFF stack and a normalization factor <1 would otherwise compress
  intensities into a narrow uint16 band, creating rounding spikes at low intensities.
when_to_use_negative:
- Input is already a single-cell feature table (e.g., .h5ad per-cell intensity data)
  — use UniFORM's feature-level workflow instead of the pixel-level stretching mechanism
- Data is multiplexed immunohistochemistry (mIHC) with chromogenic detection, where
  enzyme-catalyzed colorimetric amplification produces non-linear signal intensities
  incompatible with linear normalization factors
- Images contain significant extracellular signal or compromised segmentation masks,
  since UniFORM performs pixel-level normalization exclusively within nuclear or cellular
  segmentation masks
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: UniFORM
  role: Non-parametric Python pipeline implementing the pixel-level normalization
    and uint16 stretching mechanism
  repo: https://github.com/kunlunW/UniFORM
- name: scikit-fda
  role: Performs automatic rigid-landmark functional registration used to estimate
    the normalization factors that precede rescaling
- name: Python
  role: Environment for the UniFORM pipeline, Dask-backed OME-TIFF reading, and OME-TIFF
    output writing
  repo: https://github.com/kunlunW/UniFORM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2025.101172
    title: Toward universal immunofluorescence normalization for multiplex tissue
      imaging with UniFORM
  source_content_hash: sha256:16332f8a8cfa7e593adce0de02668e1518a21889cdd7154ac096a2626af3acc6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kunlunW/UniFORM
  tools:
  - Python
  - UniFORM
  - scikit-fda
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# uint16-intensity-rescaling-with-rounding-spike-avoidance

## Summary

A post-normalization stretching mechanism in UniFORM's pixel-level pipeline that applies a stretching factor—derived from the most-compressed sample—to restore intensity ranges to pre-normalization levels before rescaling to uint16, preventing rounding-spike compression artifacts in the OME-TIFF output.

## When to use

Use when writing pixel-level-normalized multiplex tissue imaging data back to a 16-bit OME-TIFF stack and a normalization factor <1 would otherwise compress intensities into a narrow uint16 band, creating rounding spikes at low intensities.

## When NOT to use

- Input is already a single-cell feature table (e.g., .h5ad per-cell intensity data) — use UniFORM's feature-level workflow instead of the pixel-level stretching mechanism
- Data is multiplexed immunohistochemistry (mIHC) with chromogenic detection, where enzyme-catalyzed colorimetric amplification produces non-linear signal intensities incompatible with linear normalization factors
- Images contain significant extracellular signal or compromised segmentation masks, since UniFORM performs pixel-level normalization exclusively within nuclear or cellular segmentation masks

## Inputs

- OME-TIFF whole-slide image stack (uint16 pixel intensities per marker)
- Nucleus/cell segmentation masks (to restrict histogram calculation to intracellular pixels)
- Per-marker normalization factors estimated by functional data registration of log-transformed intensity histograms

## Outputs

- normalized_pixel_stack.ome.tiff — normalized OME-TIFF image stack rescaled to uint16
- Validation check: pixel values within uint16 dynamic range (0–65535) with no rounding spikes at low intensities

## How to apply

After estimating per-marker normalization factors via log-transformed histogram calculation on segmentation-masked pixels and automatic rigid-landmark functional registration (scikit-fda), identify the most-compressed sample (the sample whose normalization factor is smallest). Apply a linear stretching factor to each sample so that, once normalization factors are applied, the intensity range is restored to its pre-normalization extent and the full uint16 dynamic range is exploited. This restores range while preserving the original distribution shape and positive-population proportions, because the stretch is linear and applied uniformly per sample. Then write the normalized stack as an OME-TIFF file (normalized_pixel_stack.ome.tiff) and verify values remain within the uint16 dynamic range without rounding-spike artifacts at low intensities.

## Related tools

- **UniFORM** (Non-parametric Python pipeline implementing the pixel-level normalization and uint16 stretching mechanism) — https://github.com/kunlunW/UniFORM
- **scikit-fda** (Performs automatic rigid-landmark functional registration used to estimate the normalization factors that precede rescaling)
- **Python** (Environment for the UniFORM pipeline, Dask-backed OME-TIFF reading, and OME-TIFF output writing) — https://github.com/kunlunW/UniFORM

## Examples

```
git clone https://github.com/kunlunW/UniFORM.git
```

## Evaluation signals

- normalized_pixel_stack.ome.tiff exists and is readable after the pipeline run
- All pixel values in the normalized stack fall within the uint16 dynamic range (0–65535)
- No rounding-spike artifacts appear at low intensities in the histogram of the normalized stack
- Original intensity distribution shape is preserved (stretch is linear), with positive-population proportions unchanged relative to pre-normalization data
- Normalization factors align negative-population peaks of the most-compressed sample to the reference distribution

## Limitations

- The stretching factor is linear, so it cannot correct non-linear chromogenic amplification in mIHC samples
- Effectiveness depends on the quality of the MTI dataset and the cell/nuclei segmentation masks used to extract signals; compromised signals cannot be recovered
- UniFORM's pixel-level workflow is optimized for within-cell normalization; extracellular regions are not supported

## Evidence

- [other] pixel-level stretching mechanism: [evidence span withheld — anchor status: not_found]
