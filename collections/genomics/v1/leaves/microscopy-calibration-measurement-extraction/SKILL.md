---
name: microscopy-calibration-measurement-extraction
description: Use when reproducing or benchmarking slide-loader microscope workflows
  and you need the published calibration precision (slide-reload displacement) and
  imaging/detection throughput figures from this paper.
when_to_use_negative:
- You need calibration values for a different microscope platform or slide loader
  (e.g., a non-Prior PL-200 system) — these figures are instrument-specific to the
  Prior PL-200 slide loader, Zeiss Axioplan 2, Nikon DSLR D5100 setup
- You want software changelog or versioning metadata — the paper explicitly reports
  no changelog and no stated version number for OpenHiCAMM
tools:
- name: OpenHiCAMM
  role: The two-phase acquisition software whose stage-calibration and speed benchmarks
    are being extracted
  repo: github.com/bdgp/OpenHiCAMM
- name: μManager
  role: Microscope hardware-control layer through which the benchmarked acquisitions
    ran
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2018.03.017
    title: 'OpenHiCAMM: High-Content Screening Software for Complex Microscope Imaging
      Workflows'
  source_content_hash: sha256:dd762a765ed063d727e4d36f95bf235acb35ea85c56be8fc41e760b9361fc3a6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OpenHiCAMM
  - μManager
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# microscopy-calibration-measurement-extraction

## Summary

Extracts stage-calibration and acquisition-speed benchmark values reported for the OpenHiCAMM two-phase imaging workflow and structures them into a per-measurement CSV for reuse in method benchmarking.

## When to use

Use when reproducing or benchmarking slide-loader microscope workflows and you need the published calibration precision (slide-reload displacement) and imaging/detection throughput figures from this paper.

## When NOT to use

- You need calibration values for a different microscope platform or slide loader (e.g., a non-Prior PL-200 system) — these figures are instrument-specific to the Prior PL-200 slide loader, Zeiss Axioplan 2, Nikon DSLR D5100 setup
- You want software changelog or versioning metadata — the paper explicitly reports no changelog and no stated version number for OpenHiCAMM

## Inputs

- Transparent Methods supplemental file of the OpenHiCAMM iScience paper (DOI 10.1016/j.isci.2018.03.017)
- Figure S4 alignment-calibration data (same slide, loaded twice, imaged at different magnifications)
- Reported Phase 1 tiling and Phase 2 detection timing results for Drosophila embryo slides

## Outputs

- calibration_speed_measurements.csv with one row per measurement and columns: condition, quantity, value, units

## How to apply

Open the Transparent Methods supplemental file referenced in the methods section and locate the stage calibration and speed test results, including the alignment-calibration figure (Figure S4, same slide loaded twice and imaged at different magnifications). Extract six reported quantities: average stage displacement after slide reload (0.059 um, about 1.5% of a 20x image), 5x tiled-acquisition speed (2.3 s/image, 200-300 images per slide), 20x detection speed for twenty-five 20x images including a focus search (12 s/image), Phase 2 time per 100 objects, and the roughly 10x speedup of the two-phase (5x detect then 20x re-image) strategy over single-pass imaging. Record each value in a structured CSV with columns condition, quantity, value, and units, one row per measurement, saved as calibration_speed_measurements.csv. Validate by cross-checking each row against the verbatim sentence or figure caption it came from.

## Related tools

- **OpenHiCAMM** (The two-phase acquisition software whose stage-calibration and speed benchmarks are being extracted) — github.com/bdgp/OpenHiCAMM
- **μManager** (Microscope hardware-control layer through which the benchmarked acquisitions ran)

## Evaluation signals

- CSV contains exactly six measurement rows covering slide-reload displacement, 5x tiling speed, 20x detection speed, Phase 2 time per 100 objects, and the two-phase speedup
- Displacement row equals 0.059 um and records the relative scale (~1.5% of a 20x image)
- Speed rows carry correct units: 2.3 s/image for 5x tiling and 12 s/image for twenty-five 20x images including a focus step
- Every row traces back verbatim to a sentence or figure caption in the Transparent Methods supplement or Figure S4 caption

## Limitations

- Measurements apply to the specific HCS platform described (Prior PL-200 slide loader, Zeiss Axioplan 2 motorized microscope, Nikon DSLR D5100 camera, Prior motorized stage); they do not transfer to other hardware
- The paper's discussion section offers only qualitative capability claims with no additional quantitative benchmarks
- Timing depends on a manual microscope adjustment between Phase 1 (5x) and Phase 2 (20x objective), which is not captured in the per-image speed figures

## Evidence

- [results] average displacement after slide reload: "The average displacement was 0.059 μm, about 1.5% of the image at 20× magnification"
- [results] two-phase imaging benchmark: [evidence span withheld — anchor status: not_found]
- [supplementary] alignment calibration source: "Figure S4. Alignment calibration for the same slide, loaded twice and imaged at different magnifications, Related to Figure 1."
- [other] workflow entry point for extraction: [evidence span withheld — anchor status: not_found]
