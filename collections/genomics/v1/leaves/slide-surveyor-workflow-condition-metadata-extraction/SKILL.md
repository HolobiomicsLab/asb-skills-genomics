---
name: slide-surveyor-workflow-condition-metadata-extraction
description: Use when article results or Transparent Methods text reports a microscopy
  acquisition rate (e.g., minutes per slide) whose value depends on workflow phase
  (Phase 1 survey vs Phase 2 re-imaging), objective magnification (5× vs 20×), or
  imaging mode, and must be captured with that metadata.
when_to_use_negative:
- The statement describes the Phase 2 re-imaging pass (slide reloading and manual
  switch to the 20× objective) rather than the SlideSurveyor rapid survey pass — the
  conditions and rate are not interchangeable
- The text contains no numeric rate or no condition qualifiers (phase/magnification/imaging
  mode), so the extracted value would be unattributable
- The goal is to actually acquire images rather than extract reported throughput —
  use the OpenHiCAMM SlideSurveyor/SlideImager acquisition workflow instead
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3068
- http://edamontology.org/topic_3383
tools:
- name: SlideSurveyor
  role: OpenHiCAMM module whose rapid live-view imaging rate is the subject of the
    extracted record; its live camera video feed and object-detection-then-SlideImager
    re-imaging define the imaging-mode metadata
  repo: github.com/bdgp/OpenHiCAMM
- name: OpenHiCAMM
  role: Acquisition framework (Fiji/μManager module) whose Phase 1/Phase 2 workflow
    entry points supply the phase metadata recorded alongside the rate
  repo: github.com/bdgp/OpenHiCAMM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2018.03.017
    title: 'OpenHiCAMM: High-Content Screening Software for Complex Microscope Imaging
      Workflows'
  source_content_hash: sha256:23ad4b746c4a3d21ecbfe102e6c43f68744d10b3a70ec77b92a2e45023365020
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OpenHiCAMM
  - SlideSurveyor
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3068
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# slide-surveyor-workflow-condition-metadata-extraction

## Summary

Extracts numeric imaging-throughput values (e.g., minutes per slide) reported for the OpenHiCAMM SlideSurveyor rapid-imaging workflow and binds each value to its condition metadata — workflow phase, objective magnification, and imaging mode — in a validated structured record. Needed because throughput numbers in this paper are only interpretable together with their phase/magnification context.

## When to use

Use when article results or Transparent Methods text reports a microscopy acquisition rate (e.g., minutes per slide) whose value depends on workflow phase (Phase 1 survey vs Phase 2 re-imaging), objective magnification (5× vs 20×), or imaging mode, and must be captured with that metadata.

## When NOT to use

- The statement describes the Phase 2 re-imaging pass (slide reloading and manual switch to the 20× objective) rather than the SlideSurveyor rapid survey pass — the conditions and rate are not interchangeable
- The text contains no numeric rate or no condition qualifiers (phase/magnification/imaging mode), so the extracted value would be unattributable
- The goal is to actually acquire images rather than extract reported throughput — use the OpenHiCAMM SlideSurveyor/SlideImager acquisition workflow instead

## Inputs

- Article results text containing the SlideSurveyor throughput statement
- Transparent Methods supplemental file referenced in the paper
- Condition metadata qualifiers: workflow phase, objective magnification, imaging mode (live-view survey vs SlideImager re-imaging)

## Outputs

- slide_surveyor_phase1_rate structured record (numeric rate plus phase, magnification, imaging-mode fields)
- Validation report confirming file existence, numeric rate field, and populated phase/magnification metadata fields

## How to apply

Load the article's results text and the Transparent Methods supplemental file referenced in the paper, then search for the SlideSurveyor throughput statement, restricting to the rapid survey pass (the module that 'takes advantage of the camera video feed to rapidly image the slide from live view' followed by re-imaging with SlideImager). Parse the numeric rate value together with its units, and record the condition metadata fields: workflow phase, magnification (e.g., 20×), and imaging mode (live-view survey vs SlideImager re-imaging). Resolve phase labels against the paper's own definitions — Phase 1 is the initial tiled pass over manually selected slide areas, and Phase 2 is re-imaging of detected embryos after manually moving to the 20× objective — so a SlideSurveyor-based rapid pass is not conflated with the Phase 2 20× re-imaging entry point. Write the value and metadata into the slide_surveyor_phase1_rate record, then validate: verify the file exists, the rate field is present and numeric, and the phase and magnification metadata fields are populated. Confirm consistency with slide-loading condition metadata, since the paper's alignment calibration explicitly characterizes slides loaded twice and imaged at different magnifications.

## Related tools

- **SlideSurveyor** (OpenHiCAMM module whose rapid live-view imaging rate is the subject of the extracted record; its live camera video feed and object-detection-then-SlideImager re-imaging define the imaging-mode metadata) — github.com/bdgp/OpenHiCAMM
- **OpenHiCAMM** (Acquisition framework (Fiji/μManager module) whose Phase 1/Phase 2 workflow entry points supply the phase metadata recorded alongside the rate) — github.com/bdgp/OpenHiCAMM

## Evaluation signals

- Rate field is present, numeric, positive, and carries units (minutes per slide)
- Phase metadata resolves to the rapid survey pass, not the Phase 2 re-imaging pass that follows the manual switch to the 20× objective
- Magnification field is populated and matches the condition stated in the source sentence (e.g., 20×)
- Imaging-mode field distinguishes live-view SlideSurveyor surveying from SlideImager re-imaging of detected objects
- Output record passes validation: file exists and all required phase/magnification metadata fields are non-empty

## Limitations

- The 20-minutes-per-slide rate is only available from the surfaced finding, not from the article's extracted entity index — it must be verified verbatim against the full results text and Transparent Methods before recording
- Phase/magnification coupling is a failure mode: the paper's Phase 1 pass images manually selected areas as tiled images (default 5×), whereas Phase 2 requires manually moving to the 20× objective, so a SlideSurveyor rapid pass at 20× must be distinguished from both
- The paper's discussion reports no quantitative benchmarks for OpenHiCAMM workflows, so the extracted rate cannot be corroborated from that section

## Evidence

- [results] SlideSurveyor, which takes advantage of the camera video feed to rapidly image the slide from live view. We detect objects and re-image with SlideImager.: "SlideSurveyor, which takes advantage of the camera video feed to rapidly image the slide from live view. We detect objects and re-image with SlideImager."
- [results] After completion of the first pass, we manually adjusted the microscope, moving to a 20× objective, and resumed the workflow at the second entry point (Phase 2).: "After completion of the first pass, we manually adjusted the microscope, moving to a 20× objective, and resumed the workflow at the second entry point (Phase 2)."
- [supplementary] Figure S4. Alignment calibration for the same slide, loaded twice and imaged at different magnifications, Related to Figure 1.: "Figure S4. Alignment calibration for the same slide, loaded twice and imaged at different magnifications, Related to Figure 1."
