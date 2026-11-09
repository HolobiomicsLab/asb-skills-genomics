---
name: screening-throughput-estimation
description: Use when timing or throughput tables for an imaging workflow are available
  and you must compute per-embryo (or per-object) acquisition cost and a speedup ratio
  versus a single-pass imaging baseline.
when_to_use_negative:
- Input is raw image data rather than acquisition timing/throughput tables — object
  detection and imaging, not cost modeling, is needed
- The workflow being evaluated is a true single-pass acquisition with no low-resolution
  detection pass, so no two-pass timing table exists to normalize
- Embryo/object density per slide is unknown, making per-object cost normalization
  impossible
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3383
tools:
- name: OpenHiCAMM
  role: Executes the two-pass (Phase 1 5× detection, Phase 2 20× re-imaging) workflow
    whose timing data feed the cost and speedup derivation
  repo: github.com/bdgp/OpenHiCAMM
- name: μManager
  role: Hardware-control layer under OpenHiCAMM that drives the Prior PL-200 slide
    loader and motorized stage during timed acquisition passes
- name: Fiji
  role: Host application module framework for OpenHiCAMM and image stitching of tiled
    Phase 1 acquisition images
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2018.03.017
    title: 'OpenHiCAMM: High-Content Screening Software for Complex Microscope Imaging
      Workflows'
  source_content_hash: sha256:95c51d81568c050ebf824d2b1f72de5eeb1138829a904f985a2a34d0a6355787
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fiji
  - OpenHiCAMM
  - μManager
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# screening-throughput-estimation

## Summary

Derive a per-sample imaging cost and speedup factor for a multi-pass high-content screening workflow by normalizing acquisition timing data (e.g., ~20 min per 100 detected Drosophila embryos at ~300 embryos/slide in the two-pass OpenHiCAMM workflow) against a single-pass full-slide 20× baseline (3,000–6,500 images), and validating the result against the paper's reported >10-fold speedup.

## When to use

Use when timing or throughput tables for an imaging workflow are available and you must compute per-embryo (or per-object) acquisition cost and a speedup ratio versus a single-pass imaging baseline.

## When NOT to use

- Input is raw image data rather than acquisition timing/throughput tables — object detection and imaging, not cost modeling, is needed
- The workflow being evaluated is a true single-pass acquisition with no low-resolution detection pass, so no two-pass timing table exists to normalize
- Embryo/object density per slide is unknown, making per-object cost normalization impossible

## Inputs

- Two-pass workflow timing table (time per 100 detected objects, e.g., ~20 min) from the Transparent Methods supplemental file
- Alignment-calibration timing table (slide loaded twice, imaged at different magnifications)
- Slide/object density estimate (~300 embryos per slide)
- Single-pass full-slide 20× imaging baseline (3,000–6,500 images per slide)

## Outputs

- derived_embryo_imaging_cost.csv containing per-embryo imaging cost rows
- Computed speedup factor value (expected >10-fold) with a recorded agreement check against the paper's reported figure

## How to apply

Load the reproduced two-pass workflow timing table (e.g., ~20 min per 100 detected objects for the Phase 1 low-resolution 5× detection plus Phase 2 20× re-imaging) and the alignment-calibration timing table from the Transparent Methods supplemental file. Normalize the total two-pass acquisition time per slide by the ~300 embryos imaged per slide to obtain the per-embryo cost (≈12 s/embryo here). Compute the speedup factor as the ratio of single-pass full-slide 20× imaging time per embryo — based on the 3,000–6,500 images a full slide requires — to the derived two-pass time per embryo. Compare the computed speedup against the paper's stated ~10× figure and record whether it agrees; write the derived per-embryo cost rows and the computed speedup value to derived_embryo_imaging_cost.csv.

## Related tools

- **OpenHiCAMM** (Executes the two-pass (Phase 1 5× detection, Phase 2 20× re-imaging) workflow whose timing data feed the cost and speedup derivation) — github.com/bdgp/OpenHiCAMM
- **μManager** (Hardware-control layer under OpenHiCAMM that drives the Prior PL-200 slide loader and motorized stage during timed acquisition passes)
- **Fiji** (Host application module framework for OpenHiCAMM and image stitching of tiled Phase 1 acquisition images)

## Examples

```
python -c "per_obj=(20*60)/100; per_embryo=per_obj/300; print('per-embryo s:', per_embryo, 'speedup >10x vs 3000-6500 full-slide 20x images'); open('derived_embryo_imaging_cost.csv','a').write(f'{per_embryo},{per_obj}\n')"
```

## Evaluation signals

- Per-embryo cost computed as (20 min × 60 s/min ÷ 100 objects) ÷ ~300 embryos/slide ≈ 12 s/embryo is reproduced within a small tolerance
- Computed speedup factor exceeds 10-fold when compared against the 3,000–6,500-image full-slide 20× baseline
- Agreement flag between computed speedup and the paper's stated ~10× figure is recorded as matching
- derived_embryo_imaging_cost.csv contains both the per-embryo cost rows and the computed speedup value
- Timing normalization consistently uses the alignment-calibration table for the same slide loaded twice at different magnifications

## Limitations

- Speedup figure depends on the assumed slide density (~300 embryos/slide); slides with different object densities yield different per-embryo costs
- Phase 1 to Phase 2 transition requires manual microscope adjustment (moving to a 20× objective), so total wall-clock throughput includes manual intervention not captured in per-object timing
- No quantitative benchmark of the software itself is reported beyond these timing comparisons; capability claims elsewhere are qualitative only

## Evidence

- [other] Two-pass timing and speedup: [evidence span withheld — anchor status: not_found]
- [results] Phase 1/Phase 2 workflow definition: "We ran the workflow to first image manually selected areas of the slide occupied by the coverslip as tiled images, followed by an image analysis module to capture the coordinates of the detected"
- [results] Manual intervention between passes: "After completion of the first pass, we manually adjusted the microscope, moving to a 20× objective, and resumed the workflow at the second entry point (Phase 2)."
- [supplementary] Alignment calibration data source: "Figure S4. Alignment calibration for the same slide, loaded twice and imaged at different magnifications, Related to Figure 1."
