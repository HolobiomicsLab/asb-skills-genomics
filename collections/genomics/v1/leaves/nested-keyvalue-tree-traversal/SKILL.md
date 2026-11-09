---
name: nested-keyvalue-tree-traversal
description: Use when inspecting or exporting the metadata of a single nanopore read
  from a POD5 file, and the attributes contain nested groups that must be expanded
  to flat key-value pairs before serialization.
when_to_use_negative:
- The goal is to export only the signal measurements of a read — use pod5Viewer's
  `Export signal` menu (.npy or text) instead
- The input is not a POD5 file (e.g. already-flattened JSON, FAST5, or a feature table),
  so no nested key-value traversal is needed
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: pod5Viewer
  role: Reference GUI whose data view panel and `File > Export all info` behaviour
    define the expected nested key-value expansion and JSON-per-read export format
  repo: https://github.com/dietvin/pod5Viewer
- name: pod5
  role: Python library (v0.3.10) used to load the POD5 file and access the read collection
    and per-read attributes programmatically
- name: Python
  role: Execution environment (v3.10.0) in which pod5 is loaded and the traversal/serialization
    script is run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:d6ebd3b8c3904fa3c0340207af16b283fd012ebfeb97a6a26687ba0b33e425d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dietvin/pod5Viewer
  tools:
  - Python
  - pod5
  - pod5Viewer
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# nested-keyvalue-tree-traversal

## Summary

Recursively traverses all attributes of a selected POD5 read as key-value pairs, expanding nested groups (e.g. context or run-info sub-structures) so every leaf value is captured, mirroring pod5Viewer's data view and 'Export all info' behaviour.

## When to use

Use when inspecting or exporting the metadata of a single nanopore read from a POD5 file, and the attributes contain nested groups that must be expanded to flat key-value pairs before serialization.

## When NOT to use

- The goal is to export only the signal measurements of a read — use pod5Viewer's `Export signal` menu (.npy or text) instead
- The input is not a POD5 file (e.g. already-flattened JSON, FAST5, or a feature table), so no nested key-value traversal is needed

## Inputs

- POD5 file containing nanopore reads with nested metadata attributes
- Read ID of the selected read within the file's read collection

## Outputs

- Flattened key-value representation of all read attributes, including expanded nested entries
- One valid JSON file per read, named by the read ID (mirroring pod5Viewer `File > Export all info`)

## How to apply

Load the POD5 file with the pod5 Python library (v0.3.10) in a Python 3.10 environment, as used by pod5Viewer. Select one read by its read ID from the file's read collection, then traverse every stored attribute as key-value pairs, recursively descending into nested groups so each leaf value is captured rather than stopping at the top-level tree. This mirrors the pod5Viewer GUI behaviour where a tab shows all attributes as key-value pairs and nested information can be expanded. Serialize the flattened structure to JSON, writing one file per read with the read ID as the file name, matching pod5Viewer's `File > Export all info` (All opened reads or Current read) export. Decide per subtree whether to keep nesting or flatten based on whether the target consumer expects leaf-level entries only.

## Related tools

- **pod5Viewer** (Reference GUI whose data view panel and `File > Export all info` behaviour define the expected nested key-value expansion and JSON-per-read export format) — https://github.com/dietvin/pod5Viewer
- **pod5** (Python library (v0.3.10) used to load the POD5 file and access the read collection and per-read attributes programmatically)
- **Python** (Execution environment (v3.10.0) in which pod5 is loaded and the traversal/serialization script is run)

## Examples

```
pod5Viewer file1.pod5
```

## Evaluation signals

- The output file exists and parses as valid JSON
- The JSON keys cover the full set of attributes stored in the POD5 record for that read, including entries from expanded nested groups
- The file name matches the read ID, as pod5Viewer creates 'a JSON file ... for each exported read with the read-id as the file name'
- The flattened output matches the key-value entries visible in the pod5Viewer data view tab for the same read
- Exporting the same read via pod5Viewer's `File > Export all info` yields a comparable attribute set (diff/comparison expectation)

## Limitations

- When installing pod5Viewer via pip it can only be started from the command line; POD5 files cannot be opened directly from a file browser
- The skill inspects metadata only; signal measurements are handled separately (pod5Viewer exports them via `Export signal` to .npy or text)
- Export output is JSON per read; users needing other structured formats must convert afterwards

## Evidence

- [readme] A tab shows all attributes stored in a pod5 file as key-value pairs. Nested information can be expanded.: "A tab shows all attributes stored in a pod5 file as key-value pairs. Nested information can be expanded."
- [methods] Export all info to JSON: "Either all opened reads (`All opened reads...`) or only the currently focused one (`Current read...`) can be exported to JSON format using `File > Export all info`."
- [readme] JSON per read with read-id as filename: "a JSON file is created for each exported read with the read-id as the file name."
- [methods] pod5 version dependency: "The pod5Viewer is built in Python (v3.10.0) and relies on the following packages:"
