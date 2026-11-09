---
name: imaging-workflow-timing-analysis
description: Use when timing tables from a multi-pass microscopy acquisition exist
  and the goal is per-object imaging cost plus a speedup ratio versus a single-pass
  full-slide baseline.
when_to_use_negative:
- No timing/acquisition-duration data exist and the question concerns image-analysis
  accuracy (e.g., ROIfinder detection quality) rather than throughput cost
- The workflow is a single-pass full-slide acquisition, so there is no two-pass baseline
  to normalize against
- Sample density or slide layout differs (e.g., well-based HCS plates) so the ~300
  embryos/slide normalization is invalid
edam_operation: http://edamontology.org/operation_3438
tools:
- name: OpenHiCAMM
  role: Workflow engine whose two-pass (5× detect, 20× re-image) acquisition timing
    is normalized to derive per-embryo cost
  repo: github.com/bdgp/OpenHiCAMM
- name: μManager
  role: Hardware-control layer whose microscope/slide-loader timings are captured
    in the workflow timing tables
- name: Fiji
  role: OpenHiCAMM module host; Fiji image stitching of multiple tiled Phase 1 acquisitions
    contributes to measured per-slide time
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2018.03.017
    title: 'OpenHiCAMM: High-Content Screening Software for Complex Microscope Imaging
      Workflows'
  source_content_hash: sha256:772df0525d3b23dd4f99879ae7f03e6b6630551ae2b809735b3b11ba951ccbac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fiji
  - OpenHiCAMM
  - μManager
  edam_operation: http://edamontology.org/operation_3438
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# imaging-workflow-timing-analysis

## Summary

Derives per-embryo high-resolution imaging cost and the speedup factor of a two-pass (low-res detect, high-res re-image) microscopy workflow by normalizing two-pass acquisition timing against full-slide 20× single-pass imaging, as done for OpenHiCAMM Drosophila embryo screening (~300 embryos/slide, ~10-fold speedup).

## When to use

Use when timing tables from a multi-pass microscopy acquisition exist and the goal is per-object imaging cost plus a speedup ratio versus a single-pass full-slide baseline.

## When NOT to use

- No timing/acquisition-duration data exist and the question concerns image-analysis accuracy (e.g., ROIfinder detection quality) rather than throughput cost
- The workflow is a single-pass full-slide acquisition, so there is no two-pass baseline to normalize against
- Sample density or slide layout differs (e.g., well-based HCS plates) so the ~300 embryos/slide normalization is invalid

## Inputs

- Reproduced two-pass workflow timing table (Transparent Methods supplemental file)
- Alignment-calibration timing table (same slide loaded twice, imaged at different magnifications)
- Embryo count per slide (~300 detected objects per slide)
- Full-slide 20× single-pass baseline image counts (3,000–6,500 images/slide)

## Outputs

- derived_embryo_imaging_cost.csv with per-embryo cost rows and computed speedup value

## How to apply

Load the reproduced two-pass workflow timing table and the alignment-calibration timing table from the Transparent Methods supplemental file. Phase 1 images manually selected coverslip areas as tiled 5× images and detects embryo coordinates; Phase 2 re-images detected embryos after manual switch to a 20× objective, so normalize the total two-pass time per slide (about 20 min per 100 detected objects) by the ~300 embryos imaged per slide to get ~12 s/embryo. Compute the speedup as the ratio of single-pass baseline per-embryo time — based on 3,000–6,500 images for full-slide 20× imaging — to the derived two-pass per-embryo time. Compare the computed speedup against the paper's stated >10-fold figure and write per-embryo cost rows plus the speedup to derived_embryo_imaging_cost.csv. Decision point: agreement with the reported value validates the two-pass strategy's timing advantage; disagreement suggests accounting for alignment-calibration overhead (slide loaded twice at different magnifications).

## Related tools

- **OpenHiCAMM** (Workflow engine whose two-pass (5× detect, 20× re-image) acquisition timing is normalized to derive per-embryo cost) — github.com/bdgp/OpenHiCAMM
- **μManager** (Hardware-control layer whose microscope/slide-loader timings are captured in the workflow timing tables)
- **Fiji** (OpenHiCAMM module host; Fiji image stitching of multiple tiled Phase 1 acquisitions contributes to measured per-slide time)

## Examples

```
python -c "per_embryo_s = 20*60/100; embryos = 300; print(per_embryo_s, 's/embryo x', embryos, 'embryos/slide; speedup vs 3000-6500 full-slide 20x images >10x')"
```

## Evaluation signals

- Per-embryo cost computes to ~12 s (20 min per 100 detected objects / ~300 embryos per slide)
- Computed speedup exceeds 10-fold over full-slide 20× imaging (3,000–6,500 images baseline), matching the paper's reported value
- Timing normalization uses embryos-per-slide count consistent with detected object counts from Phase 1 analysis, not raw tile counts
- derived_embryo_imaging_cost.csv contains both per-embryo cost rows and the speedup factor with an explicit agreement/disagreement flag versus the reported ~10x figure
- Alignment-calibration overhead (slide loaded twice at different magnifications) is accounted for or explicitly excluded with justification

## Limitations

- Phase 1 to Phase 2 transition required manual microscope adjustment (moving to a 20× objective), introducing operator-dependent timing not fully automated
- Timing figures depend on the specific hardware platform (Prior PL-200 slide loader, Zeiss Axioplan 2, Nikon DSLR D5100) and do not transfer directly to other microscopes
- The article provides qualitative capability claims in the discussion with no quantitative benchmark there; timing evidence lives only in the supplemental Transparent Methods tables

## Evidence

- [full_text] Two-pass speedup vs full-slide baseline: [evidence span withheld — anchor status: not_found]
- [results] Phase 1 workflow definition: "We ran the workflow to first image manually selected areas of the slide occupied by the coverslip as tiled images, followed by an image analysis module to capture the coordinates of the detected"
- [results] Phase 2 entry point: "After completion of the first pass, we manually adjusted the microscope, moving to a 20× objective, and resumed the workflow at the second entry point (Phase 2)."
- [supplementary] Alignment calibration data: "Figure S4. Alignment calibration for the same slide, loaded twice and imaged at different magnifications, Related to Figure 1."
- [readme] Software provenance: "Microsocope automation plugin for Micromanager (http://micromanager.org) and Fiji (http://fiji.sc)."
