---
name: nanopore-current-signal-assembly
description: Use when a task requires extracting the entire raw current measurement
  trace of a specific read ID from a POD5 file (e.g. for downstream numeric analysis
  or figure export) while keeping the reading process responsive, mirroring pod5Viewer's
  'View signal' and 'Plot signal' behavior.
when_to_use_negative:
- The goal is interactive visual exploration only and pod5Viewer itself is available
  — the GUI already handles chunked viewing and median-based subsetting to 10000 measurements
  per signal
- Only summary metadata or attributes of the read are needed rather than the raw signal
  — use pod5Viewer's read tabs or 'Export all info' JSON export instead
- The POD5 file does not contain the requested read ID — chunked assembly will silently
  produce an empty/partial buffer rather than an error
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: pod5
  role: Opens the POD5 file and provides access to a read's raw signal measurements
    by read ID (v0.3.10)
  repo: https://github.com/dietvin/pod5Viewer
- name: numpy
  role: Buffers and concatenates chunked signal measurements into a contiguous array
    and persists it in .npy binary format (v1.26.4)
  repo: https://github.com/dietvin/pod5Viewer
- name: matplotlib
  role: Renders the calibrated (pA) signal as a zoomable line plot and exports the
    current view as an SVG vector graphic (v3.9.2)
  repo: https://github.com/dietvin/pod5Viewer
- name: pod5Viewer
  role: Reference GUI whose 'View signal', 'Plot signal'/'Plot pA signal' and 'Export
    signal'/'Export current view' menu actions this skill reproduces programmatically
  repo: https://github.com/dietvin/pod5Viewer
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:aad43c3db5f4cad06cab31fdbf9164221bf49cdc0882e822a7e0d899f9323d13
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# nanopore-current-signal-assembly

## Summary

Assemble and inspect the complete raw current signal of a nanopore read from a POD5 file by reading measurements in bounded chunks (as pod5Viewer's View signal does), optionally calibrating to picoamperes, and exporting the full signal to a .npy Numpy file or the plotted view to an SVG vector graphic without freezing the application.

## When to use

Use when a task requires extracting the entire raw current measurement trace of a specific read ID from a POD5 file (e.g. for downstream numeric analysis or figure export) while keeping the reading process responsive, mirroring pod5Viewer's 'View signal' and 'Plot signal' behavior.

## When NOT to use

- The goal is interactive visual exploration only and pod5Viewer itself is available — the GUI already handles chunked viewing and median-based subsetting to 10000 measurements per signal
- Only summary metadata or attributes of the read are needed rather than the raw signal — use pod5Viewer's read tabs or 'Export all info' JSON export instead
- The POD5 file does not contain the requested read ID — chunked assembly will silently produce an empty/partial buffer rather than an error

## Inputs

- POD5 file containing nanopore reads with raw current measurements
- Read ID of the target read whose signal should be assembled
- Flag indicating whether picoampere (pA) calibration should be applied

## Outputs

- Contiguous numpy array of the read's full raw current measurements (e.g. saved as read_signal.npy)
- SVG vector graphic of the plotted signal (e.g. plot_signal.svg)
- Optionally a per-line text file with one measurement per line (pod5Viewer 'Export signal' alternative)

## How to apply

Open the POD5 file with the pod5 library (v0.3.10), locate the read whose read ID matches the target identifier, and access its raw signal. Iterate over the measurements in bounded chunks instead of loading everything at once — pod5Viewer's 'View signal' deliberately 'shows the current values in chunks' so the GUI stays responsive. Append each chunk to an in-memory buffer and concatenate the chunks into a single contiguous numpy (v1.26.4) array; validate that the array's element count equals the total number of measurements stored for the read. If picoampere calibration is required (pod5Viewer's 'Plot pA signal...'), convert raw values to pA before rendering a matplotlib (v3.9.2) line plot with zoom/interval selection (start/end coordinate fields and reset), and export the figure via matplotlib's SVG backend, replicating 'Export > Export current view...'. Persist the assembled array with numpy's binary .npy format, consistent with pod5Viewer's 'Export signal' option.

## Related tools

- **pod5** (Opens the POD5 file and provides access to a read's raw signal measurements by read ID (v0.3.10)) — https://github.com/dietvin/pod5Viewer
- **numpy** (Buffers and concatenates chunked signal measurements into a contiguous array and persists it in .npy binary format (v1.26.4)) — https://github.com/dietvin/pod5Viewer
- **matplotlib** (Renders the calibrated (pA) signal as a zoomable line plot and exports the current view as an SVG vector graphic (v3.9.2)) — https://github.com/dietvin/pod5Viewer
- **pod5Viewer** (Reference GUI whose 'View signal', 'Plot signal'/'Plot pA signal' and 'Export signal'/'Export current view' menu actions this skill reproduces programmatically) — https://github.com/dietvin/pod5Viewer

## Examples

```
import pod5, numpy as np; with pod5.Reader('file.pod5') as r: chunks=[np.asarray(read.signal) for read in r if str(read.read_id)==target_id]; sig=np.concatenate(chunks); np.save('read_signal.npy', sig)
```

## Evaluation signals

- The assembled array's element count equals the total number of measurements stored for the read in the POD5 file (no chunks dropped or duplicated)
- read_signal.npy loads via numpy.load and is a single contiguous 1-D numeric array of current values
- plot_signal.svg parses as a valid SVG vector graphic whose embedded line path corresponds to the plotted current measurements of the selected read
- If pA calibration was requested, converted values differ from raw values by the calibration scale/offset and the plot is labeled as pA signal (pod5Viewer 'Plot pA signal...')
- Chunked reading completes without loading all measurements at once (memory-bounded iteration), mirroring pod5Viewer's chunked View signal behavior

## Limitations

- pod5Viewer's plotting window subsets long signals to a maximum of 10000 measurements per signal, splitting data into 10000 bins and taking the median per bin — so GUI views show median-binned data at low zoom, not the full-resolution trace
- When installed via pip, pod5Viewer can only be started from the command line and cannot open POD5 files directly from a file browser
- Chunked full-signal assembly is intended for extraction/export; it does not provide the GUI's dynamic-resolution zooming where 'the resolution of the shown data increases dynamically with higher zoom levels'

## Evidence

- [methods] View signal shows full raw signal in chunks: "Individual current measurement can be inspected via `View > View signal`. This shows the current values in chunks."
- [methods] Export signal writes .npy Numpy file: "The values can be exported to a `.npy` Numpy file for further processing in a `Numpy` environment."
- [readme] Plot signal and pA calibration options: "To plot the current measurements select `View > Plot signal...`, `View > Plot pA signal...`. Signals can be plotted for a single read (`Focussed read...`)"
- [readme] Subsetting threshold of 10000 measurements with median binning: "long signals are subset to a maximum of 10000 measurements per signal at any given point. When subsetting, the data is split into 10000 bins, for each of which the median value is calculated."
- [readme] SVG export of current view: "The current view can be exported to a vector graphic (SVG) by pressing `Export > Export current view...`."
