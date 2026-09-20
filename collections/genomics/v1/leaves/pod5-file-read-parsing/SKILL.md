---
name: pod5-file-read-parsing
description: Use when raw nanopore reads are stored in POD5 format and you need to
  pull a focused read's key-value attributes or current measurements out for JSON/Numpy-based
  downstream processing without coding against the pod5 API.
when_to_use_negative:
- You only need to visualize or plot signals interactively rather than extract files
  — use pod5Viewer's View > Plot signal or Export current view (SVG) instead
- Your downstream pipeline can read POD5 directly via the pod5 Python API and does
  not need JSON/Numpy/plain-text intermediates
- Input data is not in POD5 format (e.g. already-extracted FASTQ/BAM signal data)
  — pod5Viewer's export mechanism operates on POD5 files
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: pod5Viewer
  role: GUI application that opens POD5 files, identifies reads by read ID, and exports
    read info to JSON and signals to .npy/text
  repo: https://github.com/dietvin/pod5Viewer
- name: Python
  role: Runtime environment (v3.10.0) in which pod5Viewer is built and installed via
    pip; also used to validate exports with numpy.load
- name: numpy
  role: Consumes the exported .npy signal files for further processing in a Numpy
    environment
- name: pod5
  role: Underlying POD5 reading library (v0.3.10) used internally by pod5Viewer; explicitly
    not required downstream after export
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:2ce5160a8182590bb08f3df590d8539cda0a275c5a33d14fa84a510f5e9bc1c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dietvin/pod5Viewer
  tools:
  - Python
  - numpy
  - pod5
  - pod5Viewer
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pod5-file-read-parsing

## Summary

Extracts per-read metadata and signal measurements out of Oxford Nanopore POD5 files using pod5Viewer, exporting either all opened reads or the focused read to JSON and its signal to .npy or plain text. This gives downstream scripts API-free, human-readable access to POD5 content without importing the pod5 package.

## When to use

Use when raw nanopore reads are stored in POD5 format and you need to pull a focused read's key-value attributes or current measurements out for JSON/Numpy-based downstream processing without coding against the pod5 API.

## When NOT to use

- You only need to visualize or plot signals interactively rather than extract files — use pod5Viewer's View > Plot signal or Export current view (SVG) instead
- Your downstream pipeline can read POD5 directly via the pod5 Python API and does not need JSON/Numpy/plain-text intermediates
- Input data is not in POD5 format (e.g. already-extracted FASTQ/BAM signal data) — pod5Viewer's export mechanism operates on POD5 files

## Inputs

- POD5 file(s) of raw nanopore sequencing reads (open via File > Open file(s)... or File > Open directory)
- Read ID of the focused read (or a list of read IDs for filtering via File > Filter reads...)

## Outputs

- JSON file per exported read, named with the read ID, containing all key-value attributes stored in the POD5 file
- .npy Numpy binary file with the read's isolated signal measurements
- Plain text file with one current measurement per line (alternative signal export)

## How to apply

Install pod5Viewer (Windows installer, Linux snap `sudo snap install pod5viewer`, or `pip install pod5Viewer` in a Python 3.10 environment) and open the POD5 file(s) via `File > Open file(s)...` or `File > Open directory`. Identify the target read by its read ID in the left panel (optionally narrow the set with `File > Filter reads...`). Export all key-value attributes for the focused read with `File > Export all info > Current read` (or `All opened reads...` for every open read); each read is written to a JSON file named with its read ID. To isolate signal, use the `Export signal` menu to write the current measurements to a `.npy` Numpy binary or a text file with one measurement per line. Validate by parsing the JSON with a standard parser and loading the `.npy` with `numpy.load` in an environment where the pod5 package is not installed.

## Related tools

- **pod5Viewer** (GUI application that opens POD5 files, identifies reads by read ID, and exports read info to JSON and signals to .npy/text) — https://github.com/dietvin/pod5Viewer
- **Python** (Runtime environment (v3.10.0) in which pod5Viewer is built and installed via pip; also used to validate exports with numpy.load)
- **numpy** (Consumes the exported .npy signal files for further processing in a Numpy environment)
- **pod5** (Underlying POD5 reading library (v0.3.10) used internally by pod5Viewer; explicitly not required downstream after export)

## Examples

```
conda create -n p5v python==3.10; conda activate p5v; pip install pod5Viewer; pod5Viewer file1.pod5
```

## Evaluation signals

- The exported JSON file parses with a standard JSON parser without errors
- The exported .npy file loads with numpy.load without importing the pod5 package, confirming API-free downstream usability
- Each exported JSON file is named with the corresponding read ID, one file per exported read
- The signal text export contains exactly one measurement per line, matching the number of measurements shown via View > View signal
- Key-value attributes in the JSON match those displayed in the pod5Viewer tab for the same read (including nested expanded information)

## Limitations

- Exports are driven manually through the GUI (or shortcuts like Ctrl+S/Ctrl+A); no scripted/batch export interface is described
- JSON export covers all info for selected reads but not the isolated signal, and signal export covers measurements only — the two export paths are separate menus
- When installed via pip, pod5Viewer can only be started from the command line and cannot be opened directly from a file browser

## Evidence

- [methods] Export all info to JSON: "Either all opened reads (`All opened reads...`) or only the currently focused one (`Current read...`) can be exported to JSON format using `File > Export all info`."
- [methods] Export signal: "The measurements of a read can be exported isolated from the remaining data of a given read through the `Export signal` menu."
- [readme] Exporting reads (README): "The values can be exported to a `.npy` Numpy file for further processing in a `Numpy` environment. Alternatively, they can be written to a text file, where each line contains one measurement."
- [readme] JSON naming (README): "a JSON file is created for each exported read with the read-id as the file name."
- [readme] Installation (README): "Optionally specify one or more path(s) to POD5 file(s) to open these directly:"
