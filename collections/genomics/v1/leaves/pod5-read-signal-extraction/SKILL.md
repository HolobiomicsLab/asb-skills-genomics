---
name: pod5-read-signal-extraction
description: Use when raw current measurements for one or more reads inside a POD5
  file must be extracted and persisted (e.g. to .npy, text, or JSON) for downstream
  analysis or API-free inspection.
when_to_use_negative:
- Input is not a POD5 file (e.g. FAST5 or FASTQ) — pod5Viewer and the pod5 v0.3.10
  API target POD5 specifically
- Only summary/metadata attributes are needed and the raw signal array itself is not
  required
- Data must remain inside the pod5Viewer GUI without exporting — this skill produces
  standalone .npy/text/JSON artifacts instead
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: pod5Viewer
  role: Reference GUI whose View signal and Export signal behaviors this skill replicates;
    provides open, inspect, plot, and export workflows for POD5 files
  repo: https://github.com/dietvin/pod5Viewer
- name: pod5
  role: Python library (v0.3.10) used to open the POD5 file and access the raw signal
    and attributes of the read by read ID
  repo: https://github.com/dietvin/pod5Viewer
- name: numpy
  role: v1.26.4; used to assemble chunked signal values into one contiguous array
    and persist it in .npy binary format
  repo: https://github.com/dietvin/pod5Viewer
- name: matplotlib
  role: v3.9.2; optional rendering of the extracted signal as a line plot exportable
    to SVG via Export current view
  repo: https://github.com/dietvin/pod5Viewer
- name: Python
  role: Runtime (v3.10.0) in which pod5, numpy, and matplotlib are orchestrated for
    extraction and export
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_003
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:bb475519796420bd3fab7ed2db5edaa19c145408128c877b376291207ca803f1
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# pod5-read-signal-extraction

## Summary

Extract the complete raw current signal of a selected read from a POD5 nanopore file and serialize it without the pod5 API, mirroring pod5Viewer's View signal and Export signal functionality.

## When to use

Use when raw current measurements for one or more reads inside a POD5 file must be extracted and persisted (e.g. to .npy, text, or JSON) for downstream analysis or API-free inspection.

## When NOT to use

- Input is not a POD5 file (e.g. FAST5 or FASTQ) — pod5Viewer and the pod5 v0.3.10 API target POD5 specifically
- Only summary/metadata attributes are needed and the raw signal array itself is not required
- Data must remain inside the pod5Viewer GUI without exporting — this skill produces standalone .npy/text/JSON artifacts instead

## Inputs

- POD5 file containing nanopore reads (raw current measurements and per-read metadata)
- Read ID identifying the target read within the POD5 file

## Outputs

- read_signal.npy — contiguous numpy array of the read's full raw signal in .npy binary format
- Plain-text file with one current measurement per line (alternative export)
- JSON file named by the read ID containing all key-value attributes, including expanded nested entries

## How to apply

Open the POD5 file with the pod5 Python library (v0.3.10) in a Python 3.10 environment and locate the target read by its read ID. Read the read's raw signal and iterate over it in bounded chunks rather than loading all measurements at once, replicating pod5Viewer's `View > View signal` chunked display that keeps the application responsive. Append each chunk to an in-memory buffer and concatenate into a single contiguous numpy (v1.26.4) array. Persist the result to read_signal.npy via numpy's binary .npy format (or a plain text file with one measurement per line), matching pod5Viewer's `Export signal` menu; optionally export all key-value metadata of the read to a JSON file named by the read ID, as in `File > Export all info > Current read`. If picoampere calibration is needed, convert raw values to pA before plotting (e.g. with matplotlib v3.9.2, as in `View > Plot pA signal...`).

## Related tools

- **pod5Viewer** (Reference GUI whose View signal and Export signal behaviors this skill replicates; provides open, inspect, plot, and export workflows for POD5 files) — https://github.com/dietvin/pod5Viewer
- **pod5** (Python library (v0.3.10) used to open the POD5 file and access the raw signal and attributes of the read by read ID) — https://github.com/dietvin/pod5Viewer
- **numpy** (v1.26.4; used to assemble chunked signal values into one contiguous array and persist it in .npy binary format) — https://github.com/dietvin/pod5Viewer
- **matplotlib** (v3.9.2; optional rendering of the extracted signal as a line plot exportable to SVG via Export current view) — https://github.com/dietvin/pod5Viewer
- **Python** (Runtime (v3.10.0) in which pod5, numpy, and matplotlib are orchestrated for extraction and export)

## Examples

```
import pod5, numpy as np; chunks=[np.asarray(list(r.signal)) for r in pod5.Reader('file.pod5').reads() if r.read_id=='<read_id>']; np.save('read_signal.npy', np.concatenate(chunks))
```

## Evaluation signals

- The assembled array's element count equals the total number of measurements stored for the read in the POD5 file — no chunks dropped or duplicated
- read_signal.npy loads successfully with numpy.load without importing the pod5 package, confirming API-free downstream usability
- Exported JSON parses with a standard JSON parser and its keys cover the full attribute set of the POD5 record, including expanded nested entries
- If calibrated, the pA-converted values map 1:1 in length to the raw signal array and plot as a continuous line in the exported SVG

## Limitations

- Chunked viewing exists because loading the full signal at once can cause unresponsive application behavior; large signals must be handled in bounded chunks
- When plotting long signals, pod5Viewer subsets to a maximum of 10000 measurements per signal by binning into 10000 bins and taking per-bin medians, so full resolution is only recovered at deeper zoom or via direct extraction
- When installed via pip, pod5Viewer can only be started from the command line and cannot open POD5 files directly from a file browser

## Evidence

- [methods] Individual current measurement can be inspected via `View > View signal`. This shows the current values in chunks.: "Individual current measurement can be inspected via `View > View signal`. This shows the current values in chunks."
- [readme] The measurements of a read can be exported isolated from the remaining data of a given read through the `Export signal` menu.: "The measurements of a read can be exported isolated from the remaining data of a given read through the `Export signal` menu. The values can be exported to a `.npy` Numpy file"
- [readme] long signals are subset to a maximum of 10000 measurements per signal at any given point.: "long signals are subset to a maximum of 10000 measurements per signal at any given point. When subsetting, the data is split into 10000 bins, for each of which the median value is calculated."
- [methods] numpy (v1.26.4): "- numpy (v1.26.4)
- pod5 (v0.3.10)"
