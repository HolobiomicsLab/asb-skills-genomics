---
name: nanopore-read-attribute-parsing
description: Use when a POD5 nanopore file's per-read metadata must be fully enumerated
  and flattened — e.g. to inspect nested attributes of a selected read ID or export
  them to per-read JSON files.
when_to_use_negative:
- The goal is to inspect or export the raw current measurements (signal data) rather
  than the read's metadata — use pod5Viewer's `View signal` or `Export signal` workflows
  instead
- The input is not a POD5 file but an already-extracted JSON/CSV of read attributes
  — re-parsing would duplicate existing work
edam_operation: http://edamontology.org/operation_0286
tools:
- name: pod5
  role: Python library (v0.3.10) used to open the POD5 file and access the per-read
    attribute records that are traversed and expanded
  repo: https://github.com/dietvin/pod5Viewer
- name: pod5Viewer
  role: 'GUI application providing the reference behaviour: displays read attributes
    as expandable key-value pairs and exports them to per-read JSON via `File > Export
    all info`'
  repo: https://github.com/dietvin/pod5Viewer
- name: Python 3.10.0
  role: Runtime environment in which the pod5 library is invoked to traverse read
    attributes programmatically
  repo: https://github.com/dietvin/pod5Viewer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:6f330140b127a0f51a430ca3d9d57d6d18e38f6aac3844a4a519986b3b6540ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dietvin/pod5Viewer
  tools:
  - Python
  - Python 3.10.0
  - pod5
  - pod5Viewer
  edam_operation: http://edamontology.org/operation_0286
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# nanopore-read-attribute-parsing

## Summary

Recursively extracts all attributes stored for a single read in a POD5 file as key-value pairs, expanding nested groups (e.g. context or run-info sub-structures) down to every leaf value, mirroring pod5Viewer's data view panel and `File > Export all info` behaviour.

## When to use

Use when a POD5 nanopore file's per-read metadata must be fully enumerated and flattened — e.g. to inspect nested attributes of a selected read ID or export them to per-read JSON files.

## When NOT to use

- The goal is to inspect or export the raw current measurements (signal data) rather than the read's metadata — use pod5Viewer's `View signal` or `Export signal` workflows instead
- The input is not a POD5 file but an already-extracted JSON/CSV of read attributes — re-parsing would duplicate existing work

## Inputs

- POD5 file(s) containing nanopore sequencing reads (raw signal plus associated metadata)
- A read ID identifying a specific read in the file's read collection

## Outputs

- One JSON file per read (file name = read ID) containing the flattened key-value representation of all read attributes, including expanded nested groups
- Optionally, an in-memory nested key-value dict of the read's attributes for further programmatic use

## How to apply

Load the POD5 file with the pod5 library (v0.3.10) inside a Python 3.10 environment, mirroring pod5Viewer's dependency stack. Select one read by its read ID from the file's read collection. Traverse all attributes stored for that read as key-value pairs, recursively descending into nested groups (e.g. context or run-info sub-structures) so that every leaf value is captured, exactly as the pod5Viewer data view panel displays them with expandable nested entries. Serialize the flattened key-value structure to JSON and write one file per read, using the read ID as the file name — the same behaviour as pod5Viewer's `File > Export all info`. Validate that the output JSON parses and that its keys cover the full attribute set of the record including expanded nested entries.

## Related tools

- **pod5** (Python library (v0.3.10) used to open the POD5 file and access the per-read attribute records that are traversed and expanded) — https://github.com/dietvin/pod5Viewer
- **pod5Viewer** (GUI application providing the reference behaviour: displays read attributes as expandable key-value pairs and exports them to per-read JSON via `File > Export all info`) — https://github.com/dietvin/pod5Viewer
- **Python 3.10.0** (Runtime environment in which the pod5 library is invoked to traverse read attributes programmatically) — https://github.com/dietvin/pod5Viewer

## Examples

```
pod5Viewer file1.pod5  # open POD5, double-click a read ID to view nested key-value attributes, then File > Export all info to write one JSON per read
```

## Evaluation signals

- The output JSON file exists and parses as valid JSON
- The JSON keys cover the full set of attributes stored in the POD5 record for the selected read, including expanded nested entries
- The output file is named after the read ID, matching pod5Viewer's `Export all info` convention (one JSON per exported read)
- No leaf value from any nested group (e.g. context, run-info) is missing compared to the attributes shown in the pod5Viewer data view tab
- Re-running the extraction on the same read ID yields an identical JSON structure (deterministic output)

## Limitations

- POD5 files contain nested information, so a naive flat key-value traversal will silently drop nested entries unless recursion into sub-groups is implemented
- pod5 (v0.3.10) within Python 3.10 is the version combination the article validates; other versions are untested in this context
- The GUI-based pod5Viewer path (double-click to open a read, `File > Export all info`) is interactive and does not scale to batch extraction of many reads; programmatic traversal via the pod5 library is needed for automation

## Evidence

- [other] In pod5Viewer's data view panel, the data of the focused read are displayed as key-value pairs; nested groups can be expanded by clicking the overarching entry.: [evidence span withheld — anchor status: not_found]
- [other] Workflow step: load POD5 with pod5 v0.3.10 in Python 3.10 and traverse attributes recursively.: [evidence span withheld — anchor status: not_found]
- [readme] README: read tabs show attributes as key-value pairs with expandable nested information.: "A tab shows all attributes stored in a pod5 file as key-value pairs. Nested information can be expanded."
- [readme] README: Export all info writes one JSON per read named by read ID.: "When exporting, the user selects an output file or directory in the file browser, where a JSON file is created for each exported read with the read-id as the file name."
- [methods] Methods: pod5Viewer is built in Python v3.10.0 and relies on pod5 v0.3.10.: "The pod5Viewer is built in Python (v3.10.0) and relies on the following packages:"
