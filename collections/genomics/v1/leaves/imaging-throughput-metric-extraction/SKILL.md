---
name: imaging-throughput-metric-extraction
description: Use when an article reports imaging throughput or stage-precision numbers
  for automated microscopy workflows (s/image tiling rates, minutes-per-slide survey
  rates, slide-reload displacement, speedup ratios) and these must be captured as
  structured data.
when_to_use_negative:
- The source section contains only qualitative capability claims (e.g. the OpenHiCAMM
  discussion section's 'robust, flexible' statements) with no numeric throughput values
  to extract
- A structured benchmark table with condition/value/units already exists — re-extraction
  adds no information
- 'Throughput figures are needed for different hardware: the reported values are tied
  to the authors'' setup (Prior PL-200 slide loader, Zeiss Axioplan 2, Nikon DSLR
  D5100 camera) and do not transfer to other platforms'
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3397
- http://edamontology.org/topic_0091
tools:
- name: OpenHiCAMM
  role: Source software whose two-phase workflow benchmarks (5× tiling s/image, 20×
    detection s/image, Phase 2 time per 100 objects, ~10× speedup) are the metrics
    being extracted
  repo: github.com/bdgp/OpenHiCAMM
- name: SlideSurveyor
  role: OpenHiCAMM module whose Phase 1 rapid-imaging rate (20 minutes per slide at
    20× magnification) is the target throughput metric
  repo: github.com/bdgp/OpenHiCAMM
- name: manual expert review
  role: Human check that each extracted value is bound to the correct workflow phase,
    magnification, and units before it is written to the output CSV
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2018.03.017
    title: 'OpenHiCAMM: High-Content Screening Software for Complex Microscope Imaging
      Workflows'
  source_content_hash: sha256:105e80e918e06aade5e8b8df7238dd90612b3122a31018f6c76ab947f93dedc4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OpenHiCAMM
  - SlideSurveyor
  - manual expert review
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3397
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# imaging-throughput-metric-extraction

## Summary

Extracts quantitative stage-calibration and acquisition-speed benchmarks from microscope-workflow papers — such as the OpenHiCAMM two-phase imaging metrics (0.059 μm slide-reload displacement, 2.3 s/image at 5×, 20 min/slide SlideSurveyor at 20×) — into structured, unit-annotated records.

## When to use

Use when an article reports imaging throughput or stage-precision numbers for automated microscopy workflows (s/image tiling rates, minutes-per-slide survey rates, slide-reload displacement, speedup ratios) and these must be captured as structured data.

## When NOT to use

- The source section contains only qualitative capability claims (e.g. the OpenHiCAMM discussion section's 'robust, flexible' statements) with no numeric throughput values to extract
- A structured benchmark table with condition/value/units already exists — re-extraction adds no information
- Throughput figures are needed for different hardware: the reported values are tied to the authors' setup (Prior PL-200 slide loader, Zeiss Axioplan 2, Nikon DSLR D5100 camera) and do not transfer to other platforms

## Inputs

- Article results text describing the two-phase (Phase 1 / Phase 2) OpenHiCAMM imaging workflow
- Transparent Methods supplemental file with stage-calibration and speed test results and alignment-calibration figures (Figure S4)
- SlideSurveyor rapid Phase 1 survey throughput statement at 20× magnification

## Outputs

- calibration_speed_measurements.csv with one row per measurement (columns: condition, quantity, value, units)
- slide_surveyor_phase1_rate record containing the numeric rate plus phase and magnification metadata

## How to apply

Locate the benchmark statements in the article's results text and the Transparent Methods supplemental file, which holds the stage-calibration and speed test results (including the alignment-calibration figures S4). Extract the reported quantities with their conditions and units: average stage displacement after slide reload (0.059 μm, ~1.5% of a 20× image), 5× tiling speed (2.3 s/image over 200–300 images per slide), 20× detection speed (12 s/image for twenty-five 20× images including a focus search), Phase 2 time per 100 objects, the ~10× speedup of the two-phase strategy over single-pass imaging, and the SlideSurveyor Phase 1 rate (20 minutes per slide at 20× magnification). Write one row per measurement into a CSV with columns condition, quantity, value, units (e.g. calibration_speed_measurements.csv), keeping phase, magnification, and imaging-mode metadata alongside each rate. Validate that each rate field is present and numeric and that phase/magnification metadata fields are populated. Rationale: these throughput figures quantify the benefit of the two-phase workflow (low-resolution 5× detection followed by 20× re-imaging of detected embryos) over single-pass imaging, so each value must stay bound to its workflow phase and magnification to be interpretable.

## Related tools

- **OpenHiCAMM** (Source software whose two-phase workflow benchmarks (5× tiling s/image, 20× detection s/image, Phase 2 time per 100 objects, ~10× speedup) are the metrics being extracted) — github.com/bdgp/OpenHiCAMM
- **SlideSurveyor** (OpenHiCAMM module whose Phase 1 rapid-imaging rate (20 minutes per slide at 20× magnification) is the target throughput metric) — github.com/bdgp/OpenHiCAMM
- **manual expert review** (Human check that each extracted value is bound to the correct workflow phase, magnification, and units before it is written to the output CSV)

## Examples

```
python -c "import csv; w=csv.writer(open('calibration_speed_measurements.csv','w',newline='')); w.writerow(['condition','quantity','value','units']); w.writerow(['Phase1 5x tiling','imaging speed','2.3','s/image'])"
```

## Evaluation signals

- Every extracted rate field is present and numeric, and phase/magnification metadata fields are non-empty in the slide_surveyor_phase1_rate record
- 5× tiling speed equals 2.3 s/image, consistent with 200–300 images acquired per slide in Phase 1
- 20× detection speed equals 12 s/image for the twenty-five 20× images including the focus-search step
- Slide-reload displacement equals 0.059 μm and is cross-checkable as ~1.5% of the image extent at 20× magnification
- SlideSurveyor Phase 1 rate equals 20 minutes per slide at 20× magnification, recorded with the condition that slide reloading and objective changes were avoided

## Limitations

- The benchmark values are reported in the Transparent Methods supplemental file and alignment-calibration figures, not the main results text, so they are easy to miss when reading only the primary narrative
- The discussion section offers no quantitative validation — capability claims there are qualitative only ('robust, flexible, easily adapted') — so throughput numbers must come from the results/supplement, not the discussion
- No version number or changelog is stated for OpenHiCAMM, so extracted benchmarks cannot be pinned to a specific software release

## Evidence

- [results] average stage displacement after slide reload: "The average displacement was 0.059 μm, about 1.5% of the image at 20× magnification"
- [results] SlideSurveyor rapid Phase 1 imaging: "SlideSurveyor, which takes advantage of the camera video feed to rapidly image the slide from live view. We detect objects and re-image with SlideImager."
- [supplementary] alignment-calibration benchmark source: "Figure S4. Alignment calibration for the same slide, loaded twice and imaged at different magnifications, Related to Figure 1."
- [readme] OpenHiCAMM software context: "Microsocope automation plugin for Micromanager (http://micromanager.org) and Fiji (http://fiji.sc)."
