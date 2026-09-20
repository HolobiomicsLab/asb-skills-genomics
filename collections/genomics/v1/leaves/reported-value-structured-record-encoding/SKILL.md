---
name: reported-value-structured-record-encoding
description: Use when a paper states a numeric performance or throughput figure tied
  to a specific instrument configuration (e.g., minutes per slide, magnification,
  workflow phase) that must be captured as a machine-readable record rather than free
  text.
when_to_use_negative:
- The reported quantity is qualitative or not tied to explicit condition metadata
  (phase, magnification, mode) — there is nothing to structure beyond a bare claim
- The goal is to actually acquire images (e.g., run the SlideSurveyor/SlideImager
  workflow on a Prior PL-200 slide loader platform) rather than encode a reported
  number from the literature
- The value is already available as a structured, machine-readable table or figure
  source data — extraction and encoding would be redundant
edam_operation: http://edamontology.org/operation_2409
tools:
- name: OpenHiCAMM
  role: The software system whose reported SlideSurveyor Phase 1 throughput (20 min/slide
    at 20×) is the value being encoded; its results text and supplementary methods
    are the extraction source
  repo: github.com/bdgp/OpenHiCAMM
- name: manual expert review
  role: Human confirmation that the parsed rate (20 minutes per slide) and its conditions
    (Phase 1, 20×) match the sentence in the paper before writing the record
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2018.03.017
    title: 'OpenHiCAMM: High-Content Screening Software for Complex Microscope Imaging
      Workflows'
  source_content_hash: sha256:248925973ae42b0198d0bd03caa941e40feff02886cf777f5835e4bc2726d115
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OpenHiCAMM
  - manual expert review
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# reported-value-structured-record-encoding

## Summary

Encodes a specific quantitative value reported in a paper's text (here, the SlideSurveyor rapid Phase 1 imaging rate of 20 minutes per slide at 20×) into a structured, validated record with its condition metadata (workflow phase, magnification, imaging mode), so downstream agents can consume the number without re-reading the prose.

## When to use

Use when a paper states a numeric performance or throughput figure tied to a specific instrument configuration (e.g., minutes per slide, magnification, workflow phase) that must be captured as a machine-readable record rather than free text.

## When NOT to use

- The reported quantity is qualitative or not tied to explicit condition metadata (phase, magnification, mode) — there is nothing to structure beyond a bare claim
- The goal is to actually acquire images (e.g., run the SlideSurveyor/SlideImager workflow on a Prior PL-200 slide loader platform) rather than encode a reported number from the literature
- The value is already available as a structured, machine-readable table or figure source data — extraction and encoding would be redundant

## Inputs

- Article results text or Transparent Methods supplemental file describing the throughput experiment
- Target record path (e.g., slide_surveyor_phase1_rate) with a schema containing rate and condition metadata fields

## Outputs

- Structured record (e.g., slide_surveyor_phase1_rate) containing: rate = 20 (minutes per slide), phase = Phase 1 (rapid survey pass), magnification = 20x, imaging mode = SlideSurveyor live-feed survey with SlideImager re-imaging
- Validation confirmation that the file exists, rate is numeric, and phase/magnification fields are non-empty

## How to apply

Load the article's results text and any supplemental methods (e.g., the Transparent Methods file referenced by the paper), then locate the sentence reporting the metric of interest — here, the SlideSurveyor rapid Phase 1 throughput at 20×, reported as 20 minutes per slide and explicitly noted to avoid slide reloading and objective changes. Parse the numeric value together with its condition metadata: which workflow phase (Phase 1 rapid survey pass vs. Phase 2 high-resolution re-imaging), which magnification (20×), and the imaging mode (live camera video feed-based SlideSurveyor followed by SlideImager re-imaging). Write the value and metadata into a named output record (slide_surveyor_phase1_rate), keeping the number separate from its qualifiers so both are individually queryable. Finally, validate the record: confirm the file exists, the rate field is present and numeric, and the phase and magnification metadata fields are populated.

## Related tools

- **OpenHiCAMM** (The software system whose reported SlideSurveyor Phase 1 throughput (20 min/slide at 20×) is the value being encoded; its results text and supplementary methods are the extraction source) — github.com/bdgp/OpenHiCAMM
- **manual expert review** (Human confirmation that the parsed rate (20 minutes per slide) and its conditions (Phase 1, 20×) match the sentence in the paper before writing the record)

## Examples

```
python -c "import json; r=json.load(open('slide_surveyor_phase1_rate.json')); assert isinstance(r['rate'],(int,float)) and r['phase'].startswith('Phase 1') and r['magnification']=='20x'; print(r)"
```

## Evaluation signals

- Output record file exists at the expected path (slide_surveyor_phase1_rate)
- Rate field is present and numeric and equals the reported 20 minutes per slide
- Phase metadata is present and reads as the rapid Phase 1 survey pass (not Phase 2 re-imaging)
- Magnification metadata is present and equals 20× (20x)
- Imaging-mode metadata distinguishes SlideSurveyor live-feed survey from SlideImager re-imaging and does not conflate the two passes

## Limitations

- The rate is tied to this specific hardware platform (Prior PL-200 slide loader, Zeiss Axioplan 2, Nikon DSLR D5100) and workflow configuration; it should not be generalized to other setups
- The article's discussion section contains no quantitative benchmark or validation results (capability claims are qualitative only), so the results text is the sole source for this numeric value
- Phase 2 requires manual microscope adjustment to a 20× objective and slide reloading; only the Phase 1 rapid pass avoids these steps — mixing phase metadata will misattribute the rate

## Evidence

- [other] Phase 1 rate and conditions: [evidence span withheld — anchor status: not_found]
- [results] SlideSurveyor imaging mode: "SlideSurveyor, which takes advantage of the camera video feed to rapidly image the slide from live view. We detect objects and re-image with SlideImager."
- [results] Phase 2 contrast condition: "After completion of the first pass, we manually adjusted the microscope, moving to a 20× objective, and resumed the workflow at the second entry point (Phase 2)."
