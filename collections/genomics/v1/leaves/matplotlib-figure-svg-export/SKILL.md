---
name: matplotlib-figure-svg-export
description: Use when raw current measurements from POD5 nanopore reads must be visualized
  as a zoomable line plot and exported as an SVG vector graphic, with optional picoampere
  or normalized scaling.
when_to_use_negative:
- You need to inspect individual current values numerically rather than visually —
  use pod5Viewer's 'View > View signal' chunk view or 'Export signal' to .npy/text
  instead
- The data are not raw current measurements from POD5 files (e.g. already a feature
  table or summary statistics) — a line plot of currents is not the appropriate rendering
- A raster/bitmap figure or lossy image is required rather than a vector graphic export
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: pod5Viewer
  role: Reference GUI whose 'View > Plot signal...'/'Plot pA signal...' and 'Export
    > Export current view...' functions this skill replicates programmatically
  repo: https://github.com/dietvin/pod5Viewer
- name: pod5
  role: Python library (v0.3.10) used to open the POD5 file and extract raw current
    measurements for a read by read ID
  repo: https://github.com/dietvin/pod5Viewer
- name: numpy
  role: Holds the extracted signal array (v1.26.4) and supports picoampere conversion
    and median-per-bin subsetting
- name: matplotlib
  role: Renders the line plot in an interactive figure window (v3.9.2) and writes
    the current view to SVG via its SVG backend
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:2d8ced12ccabe4cbc72ecdb1e1482ffa36fbf7f130cf45c601ed0d9fe6346d9d
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# matplotlib-figure-svg-export

## Summary

Renders nanopore raw current measurements from POD5 files as interactive matplotlib line plots (raw or pA-calibrated, optionally normalized) and exports the current view to a vector SVG file, replicating pod5Viewer's 'Plot signal...' and 'Export current view...' functions.

## When to use

Use when raw current measurements from POD5 nanopore reads must be visualized as a zoomable line plot and exported as an SVG vector graphic, with optional picoampere or normalized scaling.

## When NOT to use

- You need to inspect individual current values numerically rather than visually — use pod5Viewer's 'View > View signal' chunk view or 'Export signal' to .npy/text instead
- The data are not raw current measurements from POD5 files (e.g. already a feature table or summary statistics) — a line plot of currents is not the appropriate rendering
- A raster/bitmap figure or lossy image is required rather than a vector graphic export

## Inputs

- POD5 file(s) containing raw nanopore current measurements and read metadata
- Read ID(s) selecting which read's signal to plot (focussed read or all open reads)
- Calibration option: raw current values vs pA-converted signal
- Zoom interval (start and end coordinates) for the signal of interest

## Outputs

- Interactive matplotlib figure window showing the current measurement(s) as a line plot with navigation map, zoom fields, and legend
- SVG vector graphic file of the current view (e.g. plot_signal.svg)
- Optionally normalized signal plot when 'Show normalized data' is toggled

## How to apply

Open the POD5 file with the pod5 library (v0.3.10) and extract the raw current measurements for a read by its read ID into a numpy (v1.26.4) array; if pA calibration is required, convert raw values to picoamperes, mirroring pod5Viewer's 'View > Plot pA signal...' vs 'View > Plot signal...'. Render the signal as a line plot in a matplotlib (v3.9.2) figure window with a navigation map so the user can drag-select a start/end interval or type exact start/end values and press zoom, with a reset option restoring the initial view. For long signals, subset to a maximum of 10000 measurements per signal by splitting the data into 10000 bins and taking the median per bin, so resolution increases dynamically at higher zoom levels; indicate whether subsetting is active in the current section. Optionally switch to normalized signals (pod5Viewer's 'Data > Show normalized data') before export. Finally, write the current view to an SVG vector graphic via matplotlib's SVG backend (pod5Viewer's 'Export > Export current view...'), opening a file browser to select the output path.

## Related tools

- **pod5Viewer** (Reference GUI whose 'View > Plot signal...'/'Plot pA signal...' and 'Export > Export current view...' functions this skill replicates programmatically) — https://github.com/dietvin/pod5Viewer
- **pod5** (Python library (v0.3.10) used to open the POD5 file and extract raw current measurements for a read by read ID) — https://github.com/dietvin/pod5Viewer
- **numpy** (Holds the extracted signal array (v1.26.4) and supports picoampere conversion and median-per-bin subsetting)
- **matplotlib** (Renders the line plot in an interactive figure window (v3.9.2) and writes the current view to SVG via its SVG backend)

## Examples

```
python -c "import pod5, numpy, matplotlib.pyplot as plt; 
with pod5.Reader('file1.pod5') as r: sig = next(r.reads()).signal[:]
sig_pa = sig * 5; plt.plot(sig_pa); plt.savefig('plot_signal.svg')"
```

## Evaluation signals

- The output file plot_signal.svg exists and parses as a valid SVG vector graphic
- The embedded line path in the SVG corresponds to the plotted current measurements of the selected read
- Zooming to a start/end coordinate interval updates the plot to show exactly that interval, and 'Reset zoom' restores the initial full view
- With pA calibration selected, y-axis values reflect the raw-to-pA converted signal as in 'Plot pA signal...'
- For long signals, no more than 10000 measurements per signal are drawn at once, with subsetting via 10000 bins of median values and the label indicating bin size

## Limitations

- Long signals are subset to a maximum of 10000 measurements per signal at any given point (median per bin of 10000 bins), so the displayed curve is a downsampled approximation until zoomed in
- The SVG export captures only the current view; changing zoom/normalization before export changes the output
- Plotting is limited to signals of focussed or opened reads within pod5Viewer's session, not batch rendering of entire files

## Evidence

- [methods] The current view can be exported to a vector graphic (SVG) by pressing `Export > Export current view...`.: "The current view can be exported to a vector graphic (SVG) by pressing `Export > Export current view...`."
- [readme] long signals are subset to a maximum of 10000 measurements per signal at any given point. When subsetting, the data is split into 10000 bins, for each of which the median value is calculated.: "long signals are subset to a maximum of 10000 measurements per signal at any given point. When subsetting, the data is split into 10000 bins, for each of which the median value is calculated."
- [methods] To switch to plotting the normalized signals select `Data > Show normalized data` in the menu.: "To switch to plotting the normalized signals select `Data > Show normalized data` in the menu."
- [abstract] It includes functionalities for viewing, plotting, and exporting individual reads.: "It includes functionalities for viewing, plotting, and exporting individual reads."
