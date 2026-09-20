---
name: nanopore-read-metadata-inspection
description: Use when the task requires locating a subset of reads by read ID within
  one or more POD5 files — e.g. searching read IDs by substring, sorting them lexicographically,
  or intersecting them with a supplied list of read IDs.
when_to_use_negative:
- The input is not a POD5 file but an already-extracted list of read IDs or another
  format (e.g. FASTQ/BAM), where the pod5 reader library is unnecessary.
- The goal is to inspect signal values, plot signals, or export reads to JSON/NumPy
  rather than to select which reads are shown.
- No read selection is needed — all reads from the POD5 file should be retained, so
  no search/sort/filter step is required.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: pod5Viewer
  role: GUI application whose file navigator panel provides the search bar, ascending/descending
    sort buttons, and `File > Filter reads...` workflow being replicated programmatically
  repo: https://github.com/dietvin/pod5Viewer
- name: pod5
  role: Python library (v0.3.10) used to enumerate all read IDs from the input POD5
    file
- name: Python
  role: Runtime (v3.10.0) in which the pod5 reader library is invoked to enumerate
    and process read IDs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:aabbd06e2c64cd9e51de6d544bfb55c30b32eae3971ffec326b60d17f2122c8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dietvin/pod5Viewer
  tools:
  - Python
  - pod5
  - pod5Viewer
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# nanopore-read-metadata-inspection

## Summary

Locates specific reads among those enumerated in a POD5 file's navigator panel by applying live substring search, ascending/descending alphabetical sorting, and set-based read ID filtering, mirroring pod5Viewer's file navigation functions.

## When to use

Use when the task requires locating a subset of reads by read ID within one or more POD5 files — e.g. searching read IDs by substring, sorting them lexicographically, or intersecting them with a supplied list of read IDs.

## When NOT to use

- The input is not a POD5 file but an already-extracted list of read IDs or another format (e.g. FASTQ/BAM), where the pod5 reader library is unnecessary.
- The goal is to inspect signal values, plot signals, or export reads to JSON/NumPy rather than to select which reads are shown.
- No read selection is needed — all reads from the POD5 file should be retained, so no search/sort/filter step is required.

## Inputs

- POD5 file(s) of raw nanopore sequencing data
- Search query string for substring matching against read IDs
- Sort order: ascending or descending lexicographic
- Optional text file of read IDs to filter against (one read ID per line)

## Outputs

- filtered_read_ids.txt containing the matching/filtered read IDs, one per line

## How to apply

Enumerate all read IDs from the input POD5 file using the pod5 reader library (v0.3.10) under Python 3.10, as pod5Viewer does when expanding a file entry in its left navigator panel. Apply the search query as a live substring match against the read IDs, mirroring the panel search bar that updates fitting reads in real time; the adjacent button clears the query. Sort the matched read IDs alphabetically in the requested ascending or descending lexicographic order, replicating the panel's sort buttons. If a filter text file is supplied (one read ID per line, as loaded via `File > Filter reads...` > `Load IDs from file...`), intersect it with the read list and retain only reads found in the given set; write the resulting read IDs one per line to filtered_read_ids.txt.

## Related tools

- **pod5Viewer** (GUI application whose file navigator panel provides the search bar, ascending/descending sort buttons, and `File > Filter reads...` workflow being replicated programmatically) — https://github.com/dietvin/pod5Viewer
- **pod5** (Python library (v0.3.10) used to enumerate all read IDs from the input POD5 file)
- **Python** (Runtime (v3.10.0) in which the pod5 reader library is invoked to enumerate and process read IDs)

## Examples

```
pod5Viewer file1.pod5  # then File > Filter reads... > Load IDs from file... to submit one read ID per line for filtering
```

## Evaluation signals

- Every read ID in filtered_read_ids.txt must be an exact string match present in the full enumeration of read IDs from the source POD5 file(s).
- When a filter file is supplied, every output read ID must be a member of the submitted filter ID set.
- With no filter supplied, the output must equal the full (sorted/searched) read list.
- Output ordering must match the requested lexicographic sort direction (ascending or descending), as the navigator's sort buttons do.
- Each line of filtered_read_ids.txt contains exactly one read ID, matching the one-ID-per-line format the pod5Viewer filter text browser expects.

## Limitations

- Filtering operates purely on exact read ID membership; the text-browser filter requires one read ID per line and only shows reads found in the given set.
- Search is a live substring match on read IDs only — it does not search read attributes or metadata key-value pairs.
- To restore the full read list, the filter window must be reopened, `Clear` then `Done` pressed — filtering persists until explicitly cleared.

## Evidence

- [readme] Specific read IDs can be searched using the search bar. Typing a search query updates the panel to show fitting reads in real-time.: "Specific read IDs can be searched using the search bar. Typing a search query updates the panel to show fitting reads in real-time."
- [readme] Opened files and reads can be sorted alphabetically in ascending or descending order using the respective buttons above the panel.: "Opened files and reads can be sorted alphabetically in ascending or descending order using the respective buttons above the panel."
- [readme] To activate this feature navigate to `File > Filter reads...`. This opens a new window containing a text browser, in which read IDs can be inserted.: "To activate this feature navigate to `File > Filter reads...`. This opens a new window containing a text browser, in which read IDs can be inserted."
- [readme] Afterwards, only reads that are found in the given set are shown.: "Afterwards, only reads that are found in the given set are shown."
- [methods] To enable the user to focus on a specific subset of reads in given POD5 files, reads can be filtered to a subset.: "To enable the user to focus on a specific subset of reads in given POD5 files, reads can be filtered to a subset."
