---
name: read-id-extraction
description: Use when the input is one or more Oxford Nanopore POD5 files and the
  goal is a complete, ordered list of read identifiers (read IDs) contained in them,
  e.g. to build a filter ID list or verify file contents.
when_to_use_negative:
- The input is not a POD5 file (e.g. FAST5, FASTQ, or BAM) — pod5 Reader only opens
  POD5
- The goal is signal inspection, plotting, or JSON export of individual reads — use
  pod5Viewer's View/Plot signal or 'File > Export all info' features instead of raw
  ID enumeration
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: pod5
  role: Python package (v0.3.10) providing the Reader used to open the POD5 file and
    iterate over its reads to extract read_id values
- name: pod5Viewer
  role: GUI whose file navigator shows all reads in an opened POD5 file by their read
    ID; the programmatic enumeration mirrors this displayed listing
  repo: https://github.com/dietvin/pod5Viewer
- name: Python
  role: Runtime for the enumeration script (pod5Viewer is built in Python v3.10.0)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:bc88117eb397e17073cae15a0f60d67fee63e3c3ef892f0c3d7c495f15b0f9ea
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
yes_rate: 0.6666666666666666
n_claims: 6
---

# read-id-extraction

## Summary

Enumerates every read stored inside one or more POD5 files by reading with the pod5 Python package (v0.3.10) and extracting each read's read_id, mirroring how pod5Viewer lists reads under each file entry in its navigator panel.

## When to use

Use when the input is one or more Oxford Nanopore POD5 files and the goal is a complete, ordered list of read identifiers (read IDs) contained in them, e.g. to build a filter ID list or verify file contents.

## When NOT to use

- The input is not a POD5 file (e.g. FAST5, FASTQ, or BAM) — pod5 Reader only opens POD5
- The goal is signal inspection, plotting, or JSON export of individual reads — use pod5Viewer's View/Plot signal or 'File > Export all info' features instead of raw ID enumeration

## Inputs

- One or more POD5 files containing nanopore raw signal data and associated read metadata
- POD5 files loadable via the pod5 Python package (v0.3.10) Reader

## Outputs

- read_ids.txt: plain-text file with one read ID per line, in the order returned by the pod5 Reader
- Optional: count of enumerated reads for validation against the Reader's reported read count

## How to apply

Open the POD5 file with the pod5 Python package (v0.3.10) Reader, mirroring how pod5Viewer loads files via 'File > Open file(s)...'. Iterate over all reads using the Reader's read-iteration API so every stored read is enumerated. Extract the read_id attribute from each read — this is the exact identifier the pod5Viewer file navigator shows when a file entry is expanded. Write the IDs to read_ids.txt one per line, preserving the order returned by the reader. Validate that the line count matches the number of reads the Reader reports and that each line is a well-formed read ID string.

## Related tools

- **pod5** (Python package (v0.3.10) providing the Reader used to open the POD5 file and iterate over its reads to extract read_id values)
- **pod5Viewer** (GUI whose file navigator shows all reads in an opened POD5 file by their read ID; the programmatic enumeration mirrors this displayed listing) — https://github.com/dietvin/pod5Viewer
- **Python** (Runtime for the enumeration script (pod5Viewer is built in Python v3.10.0))

## Examples

```
from pod5 import Reader; ids=[str(r.read_id) for r in Reader('sample.pod5').reads()]; open('read_ids.txt','w').write('\n'.join(ids)+'\n')
```

## Evaluation signals

- Number of lines in read_ids.txt equals the number of reads the pod5 Reader reports for the file
- Every line in read_ids.txt is a valid read ID string
- The enumerated read IDs match exactly the entries shown when expanding the file in pod5Viewer's file navigator panel
- Each read ID appears exactly once (no duplicates introduced by re-iteration)
- Ordering of IDs matches the order returned by the Reader's read iteration

## Limitations

- pod5Viewer sorts reads alphabetically in ascending or descending order in the panel, so the GUI display order may differ from the Reader's iteration order
- Enumeration relies on the pod5 v0.3.10 Reader API; behavior could differ with other pod5 versions

## Evidence

- [readme] read IDs shown when expanding a file entry in the navigator: "All opened files are shown in the left panel. Expanding an entry shows all reads in a given file by their read ID."
- [intro] purpose of the pod5Viewer regarding POD5 contents and read IDs: "It allows users to open multiple POD5 files, explore their contents, and display detailed data for selected read IDs."
- [methods] pod5 dependency version: "The pod5Viewer is built in Python (v3.10.0) and relies on the following packages: - pod5 (v0.3.10)"
