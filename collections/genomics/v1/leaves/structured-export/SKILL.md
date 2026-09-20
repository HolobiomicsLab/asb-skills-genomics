---
name: structured-export
description: Use when read-level metadata and nested key-value structures from a POD5
  (nanopore) file must be flattened and persisted as JSON per read ID, e.g. for programmatic
  downstream use.
when_to_use_negative:
- You only need the raw current measurements of a read — use pod5Viewer's `Export
  signal` menu to write a .npy Numpy file or a plain text file (one measurement per
  line) instead.
- You want to visualize or plot signal data rather than capture metadata — use `View
  > Plot signal...` / `View > Plot pA signal...`.
- The metadata is already available as flat, machine-readable key-value JSON for the
  target reads.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: pod5Viewer
  role: GUI application used to inspect a POD5 read as key-value pairs with expandable
    nested information and to export all info per read to JSON (`File > Export all
    info`); installable via pip, snap, or Windows installer.
  repo: https://github.com/dietvin/pod5Viewer
- name: pod5
  role: Python library (v0.3.10) used to load the POD5 file and access the read collection
    and per-read attributes programmatically when replicating the export outside the
    GUI.
  repo: https://github.com/dietvin/pod5Viewer
- name: Python
  role: Runtime environment (v3.10.0) in which the pod5-based loading and JSON serialization
    workflow is executed.
  repo: https://github.com/dietvin/pod5Viewer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:f9580b6e16d1b285c87c9506fc921b5ecf128b4a1d8ea67a35b13b9e0e6b5132
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dietvin/pod5Viewer
  tools:
  - Python
  - pod5
  - pod5Viewer
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# -structured-export

## Summary

Exports all attributes stored for a POD5 read — including nested groups such as context and run-info structures — into one JSON file per read, mirroring pod5Viewer's `File > Export all info` behaviour.

## When to use

Use when read-level metadata and nested key-value structures from a POD5 (nanopore) file must be flattened and persisted as JSON per read ID, e.g. for programmatic downstream use.

## When NOT to use

- You only need the raw current measurements of a read — use pod5Viewer's `Export signal` menu to write a .npy Numpy file or a plain text file (one measurement per line) instead.
- You want to visualize or plot signal data rather than capture metadata — use `View > Plot signal...` / `View > Plot pA signal...`.
- The metadata is already available as flat, machine-readable key-value JSON for the target reads.

## Inputs

- POD5 file(s) containing nanopore reads with nested metadata (context, run-info sub-structures)
- Read ID(s) selecting which reads to export (single focused read or all opened reads)

## Outputs

- One JSON file per exported read, named by the read ID, containing the flattened key-value structure of all read attributes including expanded nested entries

## How to apply

Load the POD5 file with the pod5 Python library (v0.3.10) in a Python 3.10 environment, or open it in the pod5Viewer GUI (Python application built on PySide6 6.5.2). Select a read by its read ID from the file's read collection and traverse all attributes stored for that read as key-value pairs, recursively descending into nested groups so every leaf value is captured — in the GUI this is done by expanding the nested entries shown as key-value pairs in a data-viewer tab. Serialize the flattened structure to JSON, creating one JSON file per read with the read ID as the file name, exactly as pod5Viewer's `File > Export all info` does for `Current read...` or `All opened reads...`. Validate that each output file parses as valid JSON and that its keys cover the full set of attributes on the POD5 record, including the expanded nested entries. Rationale: POD5 records carry nested metadata that is not easily consumable in its native grouping, so flattening to JSON makes it inspectable and machine-readable.

## Related tools

- **pod5Viewer** (GUI application used to inspect a POD5 read as key-value pairs with expandable nested information and to export all info per read to JSON (`File > Export all info`); installable via pip, snap, or Windows installer.) — https://github.com/dietvin/pod5Viewer
- **pod5** (Python library (v0.3.10) used to load the POD5 file and access the read collection and per-read attributes programmatically when replicating the export outside the GUI.) — https://github.com/dietvin/pod5Viewer
- **Python** (Runtime environment (v3.10.0) in which the pod5-based loading and JSON serialization workflow is executed.) — https://github.com/dietvin/pod5Viewer

## Examples

```
pip install pod5Viewer; pod5Viewer file1.pod5  # then use File > Export all info / Ctrl+S to write one JSON per read ID
```

## Evaluation signals

- Each output file exists and parses as valid JSON.
- Each file is named by the read ID and one file is produced per exported read.
- The set of keys in the JSON covers all attributes stored in the POD5 record for that read, including entries from nested groups (context, run-info).
- No leaf values are lost after flattening: recursively re-nesting the JSON reproduces the original key-value hierarchy shown in pod5Viewer's data view panel.

## Limitations

- When installing pod5Viewer via pip it can only be started from the command line; POD5 files cannot be opened directly from a file browser in that mode.
- The GUI export path is interactive (file browser selection of output file or directory), so bulk programmatic export requires reimplementing the flattening via the pod5 library.
- In the data view, values are only accessible via selection/copy unless the full export is used; the data-viewer preview of a single click is overwritten when another entry is selected (use double-click for a permanent tab).

## Evidence

- [readme] Either all opened reads (`All opened reads...`) or only the currently focused one (`Current read...`) can be exported to JSON format using `File > Export all info`.: "Either all opened reads (`All opened reads...`) or only the currently focused one (`Current read...`) can be exported to JSON format using `File > Export all info`."
- [readme] One JSON file per read named by read ID.: "When exporting, the user selects an output file or directory in the file browser, where a JSON file is created for each exported read with the read-id as the file name."
- [readme] Nested information shown as key-value pairs in the data tab.: "A tab shows all attributes stored in a pod5 file as key-value pairs. Nested information can be expanded."
- [methods] Python 3.10 and pod5 0.3.10 dependencies for the programmatic workflow.: "The pod5Viewer is built in Python (v3.10.0) and relies on the following packages:"
