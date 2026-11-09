---
name: derived-metric-validation-against-reported-value
description: Use when a paper reports a headline performance metric (e.g. a speedup
  factor) and you have raw timing tables from the reproduced workflow that can be
  normalized to the same per-unit cost and compared against that reported value.
when_to_use_negative:
- The input is raw image data or ROI detection outputs (e.g. GHT/FFT-bandpass ROI
  masks) rather than timing metadata — use imaging-workflow skills instead.
- There is no reported or stated metric in the source paper to validate against, so
  the comparison step has no reference value.
- The workflow being timed is single-pass with no Phase 1/Phase 2 split, so the two-pass
  per-slide normalization is not meaningful.
tools:
- name: OpenHiCAMM
  role: Executes the two-pass acquisition workflow whose Phase 1/Phase 2 timings are
    the raw data this skill normalizes into a per-embryo cost and speedup factor
  repo: github.com/bdgp/OpenHiCAMM
- name: Fiji
  role: Host module framework and image-stitching post-processor for the tiled low-resolution
    Phase 1 images whose acquisition time enters the timing table
- name: μManager
  role: Microscope hardware control layer (slide loader, motorized stage, camera)
    whose operation times are captured in the alignment-calibration timing table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2018.03.017
    title: 'OpenHiCAMM: High-Content Screening Software for Complex Microscope Imaging
      Workflows'
  source_content_hash: sha256:83ac73d69cc5a239fcfc80579dd047ea0629d909361cfb367db7dad99983984d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fiji
  - OpenHiCAMM
  - μManager
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# derived-metric-validation-against-reported-value

## Summary

A skill for normalizing measured workflow timing data to a per-unit cost (here, seconds per embryo at ~300 embryos/slide) and checking the derived speedup factor against a paper's stated headline value (the >10-fold gain of the OpenHiCAMM two-pass imaging workflow).

## When to use

Use when a paper reports a headline performance metric (e.g. a speedup factor) and you have raw timing tables from the reproduced workflow that can be normalized to the same per-unit cost and compared against that reported value.

## When NOT to use

- The input is raw image data or ROI detection outputs (e.g. GHT/FFT-bandpass ROI masks) rather than timing metadata — use imaging-workflow skills instead.
- There is no reported or stated metric in the source paper to validate against, so the comparison step has no reference value.
- The workflow being timed is single-pass with no Phase 1/Phase 2 split, so the two-pass per-slide normalization is not meaningful.

## Inputs

- Reproduced two-pass workflow timing table (Phase 1 and Phase 2 durations per slide) from the Transparent Methods supplemental file
- Alignment-calibration timing table for the same slide loaded twice at different magnifications
- Per-slide object count (~300 detected Drosophila embryos per slide)
- Single-pass baseline: full-slide 20x image count (3,000–6,500 images per slide)

## Outputs

- Derived per-embryo high-resolution imaging cost rows (~12 s/embryo)
- Computed speedup factor versus the paper's stated ~10x value
- derived_embryo_imaging_cost.csv

## How to apply

Load the reproduced two-pass (Phase 1 low-resolution detection at 5x, Phase 2 re-imaging of detected embryos at 20x) workflow timing table and the alignment-calibration timing table from the Transparent Methods supplemental file. Normalize the total two-pass acquisition time per slide by the ~300 embryos imaged per slide to derive the per-embryo cost (the paper's timing basis is ~20 min per 100 detected objects, i.e. ~12 s/embryo). Compute the speedup as the ratio of single-pass baseline time per embryo — full-slide 20x imaging requiring 3,000–6,500 images — to the derived two-pass time per embryo. Compare the computed speedup against the paper's stated ~10x figure, record agreement or discrepancy explicitly, and write the derived cost rows and speedup value to derived_embryo_imaging_cost.csv. Rationale: single-pass imaging forces a resolution/field-depth compromise, so the derived ratio must use a full-slide 20x single-pass baseline, not a mixed-magnification baseline, to be comparable to the reported value.

## Related tools

- **OpenHiCAMM** (Executes the two-pass acquisition workflow whose Phase 1/Phase 2 timings are the raw data this skill normalizes into a per-embryo cost and speedup factor) — github.com/bdgp/OpenHiCAMM
- **Fiji** (Host module framework and image-stitching post-processor for the tiled low-resolution Phase 1 images whose acquisition time enters the timing table)
- **μManager** (Microscope hardware control layer (slide loader, motorized stage, camera) whose operation times are captured in the alignment-calibration timing table)

## Examples

```
cost_per_embryo_s = (20*60)/100  # ~20 min per 100 detected objects -> ~12 s/embryo at ~300 embryos/slide
```

## Evaluation signals

- Derived per-embryo cost should be ~12 s/embryo, consistent with ~20 min per 100 detected objects at ~300 embryos/slide
- Computed speedup must exceed 10-fold relative to the 3,000–6,500-image full-slide 20x single-pass baseline to match the paper's reported value
- Agreement between the computed speedup and the paper's stated ~10x figure must be explicitly recorded in the output
- Phase 1 tiled acquisitions exceeding one camera snapshot must be accounted for, since multiple tiles are stitched via Fiji image stitching and each tile adds acquisition time
- Alignment-calibration timing should reflect the same slide loaded twice at different magnifications; re-imaged object displacement (~0.059 μm, ~1.5% of the 20x image) confirms objects were re-imaged within tolerance and timings are valid

## Limitations

- The derivation assumes ~300 embryos per slide; slides with different detected-object densities change the per-embryo cost and the speedup factor.
- The two-pass workflow requires a manual microscope adjustment (moving to the 20x objective) between Phase 1 and Phase 2, which introduces operator-dependent time not captured by automated timing alone.
- Timing validity depends on alignment calibration for slides loaded twice at different magnifications; without it, Phase 2 re-imaging coordinates may drift and timing-to-cost normalization becomes unreliable.
- The paper's discussion offers only qualitative capability claims, so the headline speedup is the main quantitative reference available for validation.

## Evidence

- [intro] Single-pass imaging forces a resolution/field-depth compromise used as the baseline: "Imaging is done in a single pass, requiring a compromise between resolution and field depth."
- [results] Two-pass workflow phases whose timings feed the derivation: "We ran the workflow to first image manually selected areas of the slide occupied by the coverslip as tiled images, followed by an image analysis module to capture the coordinates of the detected"
- [results] Manual 20x objective switch between phases affects timing: "After completion of the first pass, we manually adjusted the microscope, moving to a 20× objective, and resumed the workflow at the second entry point (Phase 2)."
- [results] Re-imaged object displacement tolerance validating timing-to-ROI alignment: "The average displacement was 0.059 μm, about 1.5% of the image at 20× magnification"
- [supplementary] Alignment-calibration timing table used as input: "Figure S4. Alignment calibration for the same slide, loaded twice and imaged at different magnifications, Related to Figure 1."
- [results] OpenHiCAMM Fiji module context for the workflow under measurement: "We developed OpenHiCAMM as a module for the popular open source bioimage analysis package Fiji (Schindelin et al., 2012)"
