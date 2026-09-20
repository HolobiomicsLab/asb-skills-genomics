---
name: nanopore-raw-data-file-parsing
description: Use when you need to enumerate every read stored in a POD5 file and extract
  each read's read_id, e.g. to build a read inventory, feed a filter list, or verify
  the number of reads a viewer reports.
when_to_use_negative:
- The input is already a read-ID text list rather than a POD5 file — in that case
  pod5Viewer's `File > Filter reads...` window can load IDs directly.
- The goal is to view, plot, or export read signal data rather than enumerate read
  IDs — pod5Viewer's View/Plot/Export menus are the appropriate route.
- The input is not POD5 format (e.g. FAST5 or basecalled FASTQ), which this pod5-based
  parsing path does not address.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: pod5
  role: Python package (v0.3.10) whose Reader opens POD5 files and iterates over reads
    to extract read_id attributes
- name: pod5Viewer
  role: GUI reference implementation that opens POD5 files via `File > Open file(s)...`
    and lists each read by its read ID when a file entry is expanded
  repo: github.com/dietvin/pod5Viewer
- name: Python
  role: Runtime (v3.10.0) used to run the pod5 Reader-based enumeration script
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:2a85514c8b25418d8ce2385138598ec4dda6dd159efa27c3d14fb926b815f6bc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - pod5
  - pod5Viewer
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nanopore-raw-data-file-parsing

## Summary

Parses nanopore POD5 raw signal files to enumerate all contained reads and expose their read IDs, mirroring how pod5Viewer lists files and reads in its navigator panel. It provides programmatic access to the same read-level index the GUI presents.

## When to use

Use when you need to enumerate every read stored in a POD5 file and extract each read's read_id, e.g. to build a read inventory, feed a filter list, or verify the number of reads a viewer reports.

## When NOT to use

- The input is already a read-ID text list rather than a POD5 file — in that case pod5Viewer's `File > Filter reads...` window can load IDs directly.
- The goal is to view, plot, or export read signal data rather than enumerate read IDs — pod5Viewer's View/Plot/Export menus are the appropriate route.
- The input is not POD5 format (e.g. FAST5 or basecalled FASTQ), which this pod5-based parsing path does not address.

## Inputs

- POD5 file(s) containing raw nanopore signal reads

## Outputs

- read_ids.txt: one read ID per line, in reader-iteration order

## How to apply

Open the input POD5 file with the pod5 Python package (v0.3.10) Reader, which is the same library pod5Viewer relies on when loading files via `File > Open file(s)...`. Iterate over all reads returned by the Reader to enumerate every read stored in the file. Extract the read_id attribute from each read, which corresponds to the identifier shown when a file's entry is expanded in the pod5Viewer navigator panel. Write the read IDs one per line to read_ids.txt, preserving the order returned by the reader. Validate by checking that the line count equals the number of reads the pod5 Reader reports and that each line is a valid read ID string.

## Related tools

- **pod5** (Python package (v0.3.10) whose Reader opens POD5 files and iterates over reads to extract read_id attributes)
- **pod5Viewer** (GUI reference implementation that opens POD5 files via `File > Open file(s)...` and lists each read by its read ID when a file entry is expanded) — github.com/dietvin/pod5Viewer
- **Python** (Runtime (v3.10.0) used to run the pod5 Reader-based enumeration script)

## Examples

```
from pod5 import Reader; reads = [r.read_id for r in Reader('file1.pod5').reads()]; open('read_ids.txt','w').write('\n'.join(str(x) for x in reads))
```

## Evaluation signals

- The number of lines in read_ids.txt equals the number of reads the pod5 Reader reports for the file.
- Every line in read_ids.txt is a valid read ID string.
- The set of extracted read IDs matches the IDs shown when expanding the file's entry in the pod5Viewer navigator panel.
- Read IDs appear in read_ids.txt in the same order the pod5 Reader returned them.

## Limitations

- Signals longer than 10000 measurements are subset by pod5Viewer when displayed (binned medians), so full-resolution inspection of very long signals requires the signal export path rather than the viewer.
- When pod5Viewer is installed via pip it can only be started from the command line and cannot be set as the default POD5 opener from a file browser.
- This skill enumerates read IDs only; it does not parse signal values, run info, or nested attributes unless the read objects are further inspected.

## Evidence

- [readme] File navigator read listing: "All opened files are shown in the left panel. Expanding an entry shows all reads in a given file by their read ID."
- [methods] Dependency on pod5 v0.3.10: "The pod5Viewer is built in Python (v3.10.0) and relies on the following packages: - numpy (v1.26.4) - pod5 (v0.3.10)"
- [methods] File open entry point: "In the pod5Viewer open files from the menu bar in the top via `File > Open file(s)...` (individual file(s)) or `File > Open directory`"
- [readme] Read filter accepts ID lists: "This opens a new window containing a text browser, in which read IDs can be inserted. Here one read ID must be given per line."
- [abstract] Tool purpose: "The pod5Viewer offers straightforward access to raw sequencing data and associated metadata in POD5 files."
