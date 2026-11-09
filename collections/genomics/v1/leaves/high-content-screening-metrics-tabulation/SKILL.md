---
name: high-content-screening-metrics-tabulation
description: Use when a source text (e.g., a Transparent Methods supplement) reports
  quantitative run metrics for an automated HCS imaging experiment — pass durations,
  per-slide rates, object-detection counts — that must be extracted, verified, and
  tabulated.
when_to_use_negative:
- The input is raw microscopy image data that still needs embryo detection or stitching
  — use the OpenHiCAMM workflow (5x detection pass plus 20x re-imaging) instead of
  metric tabulation.
- The source section contains only qualitative capability claims (e.g., the OpenHiCAMM
  discussion section) with no quantitative benchmark results to extract.
- The experiment used a single-pass commercial microscope acquisition; two-pass duration/object-count
  metrics do not apply.
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: OpenHiCAMM
  role: The software whose two-pass autonomous 95-slide HCS run produced the metrics
    being tabulated; its Fiji module coordinates detection and re-imaging.
  repo: github.com/bdgp/OpenHiCAMM
- name: pandas
  role: Used to write the assembled metric table (workflow phase, slide count, metric
    description, value) to a CSV file.
- name: Fiji
  role: Bioimage analysis package in which OpenHiCAMM runs the 5x detection and stitching
    steps whose throughput metrics are tabulated.
  repo: http://fiji.sc
- name: μManager
  role: Hardware-control layer for the automated slide-loader platform (Prior PL-200
    loader, Zeiss Axioplan 2) whose imaging rates appear in the metrics table.
  repo: http://micromanager.org
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2018.03.017
    title: 'OpenHiCAMM: High-Content Screening Software for Complex Microscope Imaging
      Workflows'
  source_content_hash: sha256:b4702b348663aea74986745201eb0a5b9865dde17ef245408d195f44bbf9fd43
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fiji
  - OpenHiCAMM
  - pandas
  - μManager
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# high-content-screening-metrics-tabulation

## Summary

Extracts and tabulates the quantitative performance metrics of an autonomous two-pass high-content screening run — Phase 1 low-resolution detection pass and Phase 2 high-resolution re-imaging pass — as reported in the OpenHiCAMM study (iScience, 2018), and writes them to a validated CSV table for reproduction and comparison.

## When to use

Use when a source text (e.g., a Transparent Methods supplement) reports quantitative run metrics for an automated HCS imaging experiment — pass durations, per-slide rates, object-detection counts — that must be extracted, verified, and tabulated.

## When NOT to use

- The input is raw microscopy image data that still needs embryo detection or stitching — use the OpenHiCAMM workflow (5x detection pass plus 20x re-imaging) instead of metric tabulation.
- The source section contains only qualitative capability claims (e.g., the OpenHiCAMM discussion section) with no quantitative benchmark results to extract.
- The experiment used a single-pass commercial microscope acquisition; two-pass duration/object-count metrics do not apply.

## Inputs

- Transparent Methods supplemental file (or equivalent source text) describing the two-pass HCS workflow and its quantitative run metrics
- Source card findings reporting Phase 1/Phase 2 numbers (95 slides, ~12 hr, 8 min/slide, 26–751 objects/slide, 39 min/61 objects/119 images)

## Outputs

- CSV file of tabulated HCS run metrics with columns: workflow_phase, slide_count, metric_description, reported_value
- Validated set of per-phase metrics matching the verbatim source values

## How to apply

Read the Transparent Methods supplemental file accompanying the article, where all methodological and run details are documented. Locate the Phase 1 (5x low-resolution detection) metrics: the pass covered 95 slides in approximately 12 hr at a rate of 8 min/slide and detected 26–751 objects per slide, where an object is a continuous area containing one or multiple Drosophila embryos. Locate the Phase 2 metrics: after manual adjustment to the 20x objective and slide reload, re-imaging a slide with 61 detected objects took 39 minutes and produced 119 high-resolution images, plus the reported maximum per-slide imaging time. Assemble the extracted values into a tabular structure with columns for workflow phase, slide count, metric description, and reported value, and write it to a CSV file using pandas. Validate each extracted value by direct comparison against the source passage (95 slides, ~12 hr Phase 1, 26–751 objects/slide, 39 min for 61 objects/119 images).

## Related tools

- **OpenHiCAMM** (The software whose two-pass autonomous 95-slide HCS run produced the metrics being tabulated; its Fiji module coordinates detection and re-imaging.) — github.com/bdgp/OpenHiCAMM
- **pandas** (Used to write the assembled metric table (workflow phase, slide count, metric description, value) to a CSV file.)
- **Fiji** (Bioimage analysis package in which OpenHiCAMM runs the 5x detection and stitching steps whose throughput metrics are tabulated.) — http://fiji.sc
- **μManager** (Hardware-control layer for the automated slide-loader platform (Prior PL-200 loader, Zeiss Axioplan 2) whose imaging rates appear in the metrics table.) — http://micromanager.org

## Examples

```
import pandas as pd; pd.DataFrame(rows, columns=['workflow_phase','slide_count','metric_description','reported_value']).to_csv('openhicamm_95slide_metrics.csv', index=False)
```

## Evaluation signals

- Slide-count invariant: Phase 1 row must report exactly 95 slides imaged in the autonomous run.
- Phase 1 duration check: total pass time ≈ 12 hr at 8 min/slide (95 × 8 min ≈ 763 min ≈ 12.7 hr, consistent with the reported value).
- Per-slide object counts must fall within the reported range 26–751 objects (continuous areas containing one or multiple embryos).
- Phase 2 spot check: a slide with 61 detected objects must correspond to 39 minutes of re-imaging yielding 119 high-resolution images.
- CSV schema check: table has the four required columns (workflow_phase, slide_count, metric_description, reported_value) and every value matches the source passage by direct string/number comparison.

## Limitations

- The two-pass workflow required a manual microscope adjustment (switching to the 20x objective) between Phase 1 and Phase 2 and slide reloading, so metrics reflect a supervised two-entry-point workflow, not a fully hands-off run.
- The OpenHiCAMM discussion section reports no quantitative benchmarks — capability claims there are qualitative only, so this skill cannot be applied to that section.
- No software version number or changelog is available, and no DOI/archival data repository is cited for the software, limiting reproducibility of the exact run configuration.

## Evidence

- [supplementary] two-pass autonomous run metrics: [evidence span withheld — anchor status: not_found]
- [supplementary] Phase 2 re-imaging throughput: [evidence span withheld — anchor status: not_found]
- [abstract] two-pass workflow definition: "we imaged Drosophila embryos, detecting the embryos at low resolution, followed by re-imaging the detected embryos at high resolution, suitable for computational analysis and screening"
- [results] phase transition procedure: "After completion of the first pass, we manually adjusted the microscope, moving to a 20× objective, and resumed the workflow at the second entry point (Phase 2)."
- [results] toolchain for the measured platform: "We assembled an HCS microscope platform with a Prior PL-200 slide loader, a Zeiss Axioplan 2 motorized microscope, a Nikon DSLR D5100 camera, and a Prior motorized stage"
- [readme] software dependency context: "Microsocope automation plugin for Micromanager (http://micromanager.org) and Fiji (http://fiji.sc)."
