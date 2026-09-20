---
name: picoampere-signal-calibration
description: Use when raw nanopore signal values stored in a POD5 file need to be
  visualized as a line plot, optionally converted to picoamperes, zoomed to a coordinate
  interval, or exported as an SVG vector graphic.
when_to_use_negative:
- The goal is to inspect individual numeric current values in chunks rather than a
  plotted signal — pod5Viewer's 'View > View signal' view is the appropriate mode
- The task is exporting the raw measurements themselves (.npy or text) rather than
  a plot — use the 'Export signal' menu functionality instead
- Input data is not raw nanopore signal in POD5 format (e.g. already basecalled sequences
  or feature tables)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: pod5Viewer
  role: GUI application providing the 'View > Plot signal...'/'Plot pA signal...'
    views, zoom navigation, normalized-data toggle, and 'Export current view...' to
    SVG
  repo: https://github.com/dietvin/pod5Viewer
- name: pod5
  role: Python library (v0.3.10) used to open POD5 files and extract raw current measurements
    by read ID
- name: numpy
  role: Array library (v1.26.4) holding the extracted signal for pA conversion and
    per-bin median subsetting
- name: matplotlib
  role: Plotting library (v3.9.2) rendering the line plot in an interactive window
    and exporting it to SVG via its SVG backend
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:abf8c3ddd6b24b7b4a6b90e8c278a8ff9ebed1eb22201bbfd25cacba4388c63e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dietvin/pod5Viewer
  tools:
  - Python
  - matplotlib
  - numpy
  - pod5
  - pod5Viewer
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# picoampere-signal-calibration

## Summary

Plots raw nanopore current measurements from POD5 reads as calibrated picoampere (pA) signal in an interactive, zoomable line plot (as implemented in pod5Viewer's 'View > Plot signal...'/'Plot pA signal...' menus) and exports the figure to SVG. It makes raw charge measurements interpretable in physical units and visually inspectable.

## When to use

Use when raw nanopore signal values stored in a POD5 file need to be visualized as a line plot, optionally converted to picoamperes, zoomed to a coordinate interval, or exported as an SVG vector graphic.

## When NOT to use

- The goal is to inspect individual numeric current values in chunks rather than a plotted signal — pod5Viewer's 'View > View signal' view is the appropriate mode
- The task is exporting the raw measurements themselves (.npy or text) rather than a plot — use the 'Export signal' menu functionality instead
- Input data is not raw nanopore signal in POD5 format (e.g. already basecalled sequences or feature tables)

## Inputs

- POD5 file containing raw nanopore current measurements for one or more reads
- Read ID of the read whose signal should be plotted
- Optional start/end coordinate interval for zooming

## Outputs

- Interactive matplotlib line plot of the read's current signal (raw or pA-calibrated)
- SVG vector graphic file of the current plot view (e.g. plot_signal.svg)

## How to apply

Open the POD5 file with the pod5 library (v0.3.10) and extract the raw current measurements for one read by its read ID, loading them into a numpy (v1.26.4) array. If physical units are required, convert raw values to picoamperes, mirroring pod5Viewer's distinction between 'View > Plot signal...' and 'View > Plot pA signal...'; normalization is toggled separately via 'Data > Show normalized data'. Render the array as a matplotlib (v3.9.2) line plot in an interactive figure window supporting start/end coordinate zooming with a reset option. To keep long signals responsive, pod5Viewer subsets to a maximum of 10000 measurements per signal, splitting into 10000 bins and taking the per-bin median, with resolution increasing dynamically at higher zoom levels. Export the rendered figure to a single SVG vector graphic via matplotlib's SVG backend, replicating 'Export > Export current view...'.

## Related tools

- **pod5Viewer** (GUI application providing the 'View > Plot signal...'/'Plot pA signal...' views, zoom navigation, normalized-data toggle, and 'Export current view...' to SVG) — https://github.com/dietvin/pod5Viewer
- **pod5** (Python library (v0.3.10) used to open POD5 files and extract raw current measurements by read ID)
- **numpy** (Array library (v1.26.4) holding the extracted signal for pA conversion and per-bin median subsetting)
- **matplotlib** (Plotting library (v3.9.2) rendering the line plot in an interactive window and exporting it to SVG via its SVG backend)

## Examples

```
pod5Viewer file1.pod5
```

## Evaluation signals

- The output file plot_signal.svg exists and parses as a valid SVG vector graphic
- The SVG's embedded line path corresponds to the plotted current measurements of the selected read
- Zooming to a start/end interval updates the plot and 'Reset zoom' restores the initial full-range view
- After 'Plot pA signal...', plotted y-values are in picoamperes rather than raw digitization units
- Signals longer than 10000 measurements are binned (10000 bins, per-bin median) and the subsetting label reports the bin size

## Limitations

- Long signals are subset to a maximum of 10000 measurements per signal at any given point (per-bin median), so low zoom levels show a reduced-resolution approximation
- When installing via pip, pod5Viewer can only be started from the command line; POD5 files cannot be opened directly from a file browser
- Normalized plotting must be toggled manually via 'Data > Show normalized data' and reverted with 'Data > Show data'

## Evidence

- [methods] To plot the current measurements select `View > Plot signal...`, `View > Plot pA signal...`.: "To plot the current measurements select `View > Plot signal...`, `View > Plot pA signal...`."
- [readme] The current view can be exported to a vector graphic (SVG) by pressing `Export > Export current view...`.: "The current view can be exported to a vector graphic (SVG) by pressing `Export > Export current view...`."
- [methods] long signals are subset to a maximum of 10000 measurements per signal at any given point. When subsetting, the data is split into 10000 bins, for each of which the median value is calculated.: "long signals are subset to a maximum of 10000 measurements per signal at any given point. When subsetting, the data is split into 10000 bins, for each of which the median value is calculated."
- [methods] The pod5Viewer is built in Python (v3.10.0) and relies on the following packages:: "The pod5Viewer is built in Python (v3.10.0) and relies on the following packages:"
- [readme] To switch to plotting the normalized signals select `Data > Show normalized data` in the menu.: "To switch to plotting the normalized signals select `Data > Show normalized data` in the menu."
