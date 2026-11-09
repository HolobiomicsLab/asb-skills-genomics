---
name: read-id-substring-search-filtering
description: Use when a user needs to find a subset of reads within one or more POD5
  files by read-ID substring, alphabetical ordering, or an external list of read IDs,
  as in the pod5Viewer navigator panel.
when_to_use_negative:
- The goal is to inspect or plot the signal data of a read rather than select read
  IDs — use pod5Viewer's View/Plot signal functions instead
- The input is not POD5 (e.g. already a FASTQ/BAM read list) — navigator enumeration
  from POD5 does not apply
- Selection must be based on read signal properties or metadata values rather than
  read-ID identity
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: pod5Viewer
  role: GUI reference implementation whose file navigator provides the live substring
    search, ascending/descending sort buttons, and read-ID filter window (`File >
    Filter reads...`) that this skill reproduces programmatically
  repo: https://github.com/dietvin/pod5Viewer
- name: pod5
  role: reader library (v0.3.10) used to enumerate read IDs from the input POD5 files
    before searching, sorting, and filtering
- name: Python
  role: runtime (v3.10) in which the read-ID enumeration, substring matching, sorting,
    and set intersection are implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:bb764c55856f0f88058955b31877d78544d40177f48d556612939548eeb4a80d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dietvin/pod5Viewer
  tools:
  - Python
  - pod5
  - pod5Viewer
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# read-id-substring-search-filtering

## Summary

Locates specific nanopore reads among those enumerated from POD5 files by combining live substring search over read IDs, lexicographic sorting, and set-based filtering against a submitted read-ID list, mirroring the pod5Viewer file navigator panel's search, sort, and filter functions.

## When to use

Use when a user needs to find a subset of reads within one or more POD5 files by read-ID substring, alphabetical ordering, or an external list of read IDs, as in the pod5Viewer navigator panel.

## When NOT to use

- The goal is to inspect or plot the signal data of a read rather than select read IDs — use pod5Viewer's View/Plot signal functions instead
- The input is not POD5 (e.g. already a FASTQ/BAM read list) — navigator enumeration from POD5 does not apply
- Selection must be based on read signal properties or metadata values rather than read-ID identity

## Inputs

- One or more POD5 files containing nanopore reads with read IDs
- Optional search query string (substring match against read IDs)
- Optional sort direction (ascending or descending lexicographic)
- Optional text file of read IDs, one per line, used as a filter set

## Outputs

- filtered_read_ids.txt containing the matching read IDs, one per line
- Optionally, the ranked/searched full read-ID list when no filter file is supplied

## How to apply

First enumerate all read IDs from the input POD5 file(s) using the pod5 reader library (v0.3.10) under Python 3.10, replicating the entries the pod5Viewer file navigator shows when a file entry is expanded. Apply the search query as a live substring match against the read IDs, exactly as the panel search bar updates fitting reads in real time while typing. Sort matched read IDs alphabetically in the requested ascending or descending lexicographic order, mirroring the sort buttons above the navigator panel. If a filter file is supplied (one read ID per line, as accepted by pod5Viewer's `File > Filter reads...` text browser), intersect it with the read list so only reads found in the given set are retained. Write the resulting read IDs one per line to filtered_read_ids.txt and validate that each output ID is an exact string match present in the source POD5 enumeration and, when applicable, a member of the submitted ID set.

## Related tools

- **pod5Viewer** (GUI reference implementation whose file navigator provides the live substring search, ascending/descending sort buttons, and read-ID filter window (`File > Filter reads...`) that this skill reproduces programmatically) — https://github.com/dietvin/pod5Viewer
- **pod5** (reader library (v0.3.10) used to enumerate read IDs from the input POD5 files before searching, sorting, and filtering)
- **Python** (runtime (v3.10) in which the read-ID enumeration, substring matching, sorting, and set intersection are implemented)

## Examples

```
from pod5 import Reader; ids = sorted(str(r.read_id) for r in Reader('file.pod5').reads()); sel = [i for i in ids if 'query' in i and i in set(open('ids.txt').read().split())]
```

## Evaluation signals

- Every read ID in filtered_read_ids.txt is an exact string match present in the full enumeration of the source POD5 file(s)
- When a filter file is supplied, every output read ID is a member of the submitted ID set
- With no filter file, the output equals the full (sorted/searched) read list
- Output ordering matches the requested ascending or descending lexicographic sort
- Output format is one read ID per line, mirroring the one-ID-per-line format pod5Viewer requires for filter input

## Limitations

- Search is a plain substring match on read IDs; no fuzzy or regex matching is described
- Filtering retains only reads found in the given set; clearing the subset requires reopening the filter window, pressing Clear and Done
- The pod5Viewer GUI ties this behavior to opened files in the navigator; pip-installed pod5Viewer can only be started from the command line, not by opening POD5 files from a file browser

## Evidence

- [readme] Specific read IDs can be searched using the search bar. Typing a search query updates the panel to show fitting reads in real-time.: "Specific read IDs can be searched using the search bar. Typing a search query updates the panel to show fitting reads in real-time."
- [readme] Opened files and reads can be sorted alphabetically in ascending or descending order using the respective buttons above the panel.: "Opened files and reads can be sorted alphabetically in ascending or descending order using the respective buttons above the panel."
- [readme] Here one read ID must be given per line. Alternatively, read IDs can be loaded from a text file, again containing one read ID per line.: "Here one read ID must be given per line. Alternatively, read IDs can be loaded from a text file, again containing one read ID per line."
- [methods] To enable the user to focus on a specific subset of reads in given POD5 files, reads can be filtered to a subset.: "To enable the user to focus on a specific subset of reads in given POD5 files, reads can be filtered to a subset."
- [readme] Afterwards, only reads that are found in the given set are shown.: "Afterwards, only reads that are found in the given set are shown."
