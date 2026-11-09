---
name: nanopore-signal-binary-npy-export
description: Use when a single read's raw nanopore current measurements stored in
  a POD5 file must be extracted into a .npy Numpy binary or plain-text file for downstream
  numpy processing without the pod5 API.
when_to_use_negative:
- Input is not a POD5 file (e.g. legacy FAST5 or FASTQ reads) — pod5Viewer is built
  for inspecting POD5 files.
- Goal is bulk scripted extraction of many reads' signals — only the all-info JSON
  export offers an 'All opened reads' option; the signal export targets a given read,
  so use the pod5 API/scripts instead for batch jobs.
- Goal is saving the plotted figure — use `Export > Export current view...` for an
  SVG vector graphic instead.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: pod5Viewer
  role: GUI used to open the POD5 file, focus the read by its read ID, and trigger
    the `Export signal` (and `File > Export all info > Current read`) export flows
  repo: https://github.com/dietvin/pod5Viewer
- name: numpy
  role: 'Target binary format of the export: the .npy file is written for further
    processing in a Numpy environment and loads with numpy.load (pod5Viewer bundles
    numpy v1.26.4)'
- name: pod5
  role: Underlying library (v0.3.10) pod5Viewer uses internally to read POD5 files;
    explicitly not required after export
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:0cfbe90d58403f0d41377365545f9966c8252d07eaf1350e03fee9d1770828ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dietvin/pod5Viewer
  tools:
  - Python
  - numpy
  - pod5
  - pod5Viewer
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# nanopore-signal-binary-npy-export

## Summary

Exports the current measurements (signal) of a focused read from a POD5 file via pod5Viewer's `Export signal` menu to a .npy Numpy binary file (or a one-measurement-per-line text file), decoupling downstream analysis from the pod5 API.

## When to use

Use when a single read's raw nanopore current measurements stored in a POD5 file must be extracted into a .npy Numpy binary or plain-text file for downstream numpy processing without the pod5 API.

## When NOT to use

- Input is not a POD5 file (e.g. legacy FAST5 or FASTQ reads) — pod5Viewer is built for inspecting POD5 files.
- Goal is bulk scripted extraction of many reads' signals — only the all-info JSON export offers an 'All opened reads' option; the signal export targets a given read, so use the pod5 API/scripts instead for batch jobs.
- Goal is saving the plotted figure — use `Export > Export current view...` for an SVG vector graphic instead.

## Inputs

- POD5 file(s) containing raw nanopore reads (opened via `File > Open file(s)...` or `File > Open directory`)
- Read ID of the focused read whose signal is to be exported

## Outputs

- .npy Numpy binary file containing the focused read's current measurements
- Plain text file with one measurement per line (alternative signal export format)
- JSON file named by the read ID containing all key-value attributes of the focused read (optional companion export via `File > Export all info > Current read`)

## How to apply

Open the POD5 file in pod5Viewer via `File > Open file(s)...` (or `File > Open directory`) and locate the target read by its read ID in the left panel; double-click to open it permanently so it becomes the focused read. Use the `Export signal` menu to write the read's measurements isolated from the remaining data of the read: choose the .npy Numpy file for further processing in a Numpy environment (pod5Viewer ships numpy v1.26.4), or a text file where each line contains one measurement for plain-text access. If the read's key-value attributes are also needed, export them separately with `File > Export all info > Current read` (Ctrl+S), which creates one JSON file per exported read named with the read ID. Rationale: the export lets downstream code load the .npy with numpy.load and parse the JSON with a standard JSON parser without importing the pod5 package (v0.3.10), which pod5Viewer only needs internally to read POD5 files.

## Related tools

- **pod5Viewer** (GUI used to open the POD5 file, focus the read by its read ID, and trigger the `Export signal` (and `File > Export all info > Current read`) export flows) — https://github.com/dietvin/pod5Viewer
- **numpy** (Target binary format of the export: the .npy file is written for further processing in a Numpy environment and loads with numpy.load (pod5Viewer bundles numpy v1.26.4))
- **pod5** (Underlying library (v0.3.10) pod5Viewer uses internally to read POD5 files; explicitly not required after export)

## Examples

```
import numpy as np; signal = np.load('read_id.npy')
```

## Evaluation signals

- numpy.load reads the produced .npy file successfully without importing the pod5 package
- Exported JSON parses with a standard JSON parser without the pod5 API installed
- Signal export is isolated from the read's metadata — measurements only, no key-value attributes mixed in
- All-info export creates one JSON file per exported read with the read ID as the file name
- Text-file variant contains exactly one measurement per line

## Limitations

- For plotting performance pod5Viewer subsets long signals to a maximum of 10000 measurements per signal (median value per 10000 bins); this subsetting pertains to the plot view and must not be confused with the exported signal data.
- When installed via pip, pod5Viewer can only be started from the command line and cannot open POD5 files directly from a file browser.
- The `Export signal` menu exports a given read's measurements; only the all-info JSON export offers an `All opened reads...` option.

## Evidence

- [readme] The measurements of a read can be exported isolated from the remaining data of a given read through the `Export signal` menu.: "The measurements of a read can be exported isolated from the remaining data of a given read through the `Export signal` menu."
- [readme] The values can be exported to a `.npy` Numpy file for further processing in a `Numpy` environment.: "The values can be exported to a `.npy` Numpy file for further processing in a `Numpy` environment."
- [readme] Either all opened reads (`All opened reads...`) or only the currently focused one (`Current read...`) can be exported to JSON format using `File > Export all info`.: "Either all opened reads (`All opened reads...`) or only the currently focused one (`Current read...`) can be exported to JSON format using `File > Export all info`."
- [methods] long signals are subset to a maximum of 10000 measurements per signal at any given point.: "long signals are subset to a maximum of 10000 measurements per signal at any given point."
