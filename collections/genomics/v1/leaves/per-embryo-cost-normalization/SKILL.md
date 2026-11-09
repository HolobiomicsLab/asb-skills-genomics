---
name: per-embryo-cost-normalization
description: Use when two-pass imaging timing data (per-slide acquisition time, detected-object
  counts, full-slide image counts) must be converted into per-sample cost and a speedup
  factor to benchmark against a reported figure.
when_to_use_negative:
- The imaging dataset lacks per-slide timing or detected-object counts, so per-embryo
  normalization cannot be computed
- The screen uses single-pass acquisition or well-based (e.g., cell culture plate)
  samples rather than slide-based two-pass imaging, so the full-slide 20x baseline
  comparison is invalid
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: OpenHiCAMM
  role: OpenHiCAMM executed the two-pass workflow (Phase 1 low-resolution detection
    at 5x, Phase 2 20x re-imaging with SlideImager) whose timing data this skill normalizes
  repo: github.com/bdgp/OpenHiCAMM
- name: Fiji
  role: Fiji image stitching post-processes tiled Phase 1 images used in the two-pass
    workflow being costed
- name: µManager
  role: Microscope hardware control underlying OpenHiCAMM acquisition timing measured
    in this cost analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2018.03.017
    title: 'OpenHiCAMM: High-Content Screening Software for Complex Microscope Imaging
      Workflows'
  source_content_hash: sha256:0651e55a1a32f11f3738db51e7719a59773fb4048bd5f6b4952f3767c276245d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fiji
  - OpenHiCAMM
  - µManager
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# per-embryo-cost-normalization

## Summary

Derives the per-embryo high-resolution imaging cost from the OpenHiCAMM two-pass (Phase 1/Phase 2) timing data by normalizing per-slide acquisition time by the ~300 embryos imaged per slide, then validates the computed >10x speedup against full-slide 20x imaging baselines.

## When to use

Use when two-pass imaging timing data (per-slide acquisition time, detected-object counts, full-slide image counts) must be converted into per-sample cost and a speedup factor to benchmark against a reported figure.

## When NOT to use

- The imaging dataset lacks per-slide timing or detected-object counts, so per-embryo normalization cannot be computed
- The screen uses single-pass acquisition or well-based (e.g., cell culture plate) samples rather than slide-based two-pass imaging, so the full-slide 20x baseline comparison is invalid

## Inputs

- Two-pass workflow timing table (Phase 1 5x tiled acquisition and Phase 2 20x re-imaging) from the Transparent Methods supplemental file
- Alignment-calibration timing table (same slide loaded twice and imaged at different magnifications)
- Detected-object counts per slide (~300 Drosophila embryos per slide)
- Full-slide 20x imaging baseline image counts (3,000-6,500 images per slide)

## Outputs

- derived_embryo_imaging_cost.csv containing per-embryo imaging cost rows and the computed speedup value
- Computed speedup factor versus the paper's reported ~10x figure with agreement status

## How to apply

Load the reproduced two-pass workflow timing table and the alignment-calibration timing table from the Transparent Methods supplemental file. Normalize the total Phase 1 + Phase 2 acquisition time per slide by the ~300 Drosophila embryos detected per slide to yield a per-embryo cost; the article's data give roughly 20 min per 100 detected objects (~12 s/embryo at ~300 embryos/slide). Compute the speedup as the ratio of full-slide single-pass 20x imaging time per embryo (derived from 3,000-6,500 images per slide) to the two-pass per-embryo time. Compare the computed speedup against the paper's stated >10x figure and record agreement. Write the derived per-embryo cost rows and the computed speedup to derived_embryo_imaging_cost.csv.

## Related tools

- **OpenHiCAMM** (OpenHiCAMM executed the two-pass workflow (Phase 1 low-resolution detection at 5x, Phase 2 20x re-imaging with SlideImager) whose timing data this skill normalizes) — github.com/bdgp/OpenHiCAMM
- **Fiji** (Fiji image stitching post-processes tiled Phase 1 images used in the two-pass workflow being costed)
- **µManager** (Microscope hardware control underlying OpenHiCAMM acquisition timing measured in this cost analysis)

## Examples

```
python normalize_cost.py --timing phase1_phase2_timings.csv --objects-per-slide 300 --baseline-images 3000:6500 --output derived_embryo_imaging_cost.csv
```

## Evaluation signals

- Per-embryo cost should fall in the ~12 s/embryo range given ~20 min per 100 detected objects and ~300 embryos/slide
- Computed speedup factor should exceed 10x, matching the paper's reported value
- derived_embryo_imaging_cost.csv contains rows for both the per-embryo cost and the speedup comparison
- Normalization denominator matches the detected-object count from Phase 1 (~300 embryos per slide), not the raw slide count
- Baseline comparison uses the reported 3,000-6,500 full-slide 20x images per slide rather than an assumed default

## Limitations

- Timing is derived from a single instrument platform (Prior PL-200 slide loader, Zeiss Axioplan 2, Nikon DSLR D5100) and may not transfer to other hardware configurations
- Per-embryo cost depends on embryo density per slide (~300/slide); sparser slides change the normalization
- No version number or changelog is available for OpenHiCAMM, so the exact software build underlying the timing table is unrecorded

## Evidence

- [results] two-pass speedup: [evidence span withheld — anchor status: not_found]
- [results] workflow phases: "We ran the workflow to first image manually selected areas of the slide occupied by the coverslip as tiled images, followed by an image analysis module to capture the coordinates of the detected"
- [results] phase 2: "After completion of the first pass, we manually adjusted the microscope, moving to a 20× objective, and resumed the workflow at the second entry point (Phase 2)."
- [abstract] tooling: "we imaged Drosophila embryos, detecting the embryos at low resolution, followed by re-imaging the detected embryos at high resolution, suitable for computational analysis and screening"
- [readme] install: "Build and install fiji and micromanager as outlined in the script file for your OS:"
