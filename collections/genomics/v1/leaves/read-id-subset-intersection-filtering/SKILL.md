---
name: read-id-subset-intersection-filtering
description: Use when a task requires locating or narrowing a subset of reads enumerated
  from POD5 nanopore files by substring search, lexicographic sorting, or intersection
  with a supplied list of read IDs (mirroring pod5Viewer's navigator search and File
  > Filter reads workflow).
when_to_use_negative:
- Input data is not in POD5 format (e.g. FAST5 or FASTQ) — this skill replicates pod5Viewer's
  POD5-specific navigator and filter behavior
- Filtering requires signal-level criteria (e.g. length or signal value cutoffs) rather
  than membership in a supplied read-ID set — pod5Viewer filtering only retains reads
  found in the given ID set
- A GUI inspection session is sufficient — opening the pod5Viewer and using `File
  > Filter reads...` directly is simpler than reproducing the workflow in a script
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: pod5Viewer
  role: Reference GUI whose navigator-panel search, sort buttons, and `File > Filter
    reads...` (Load IDs from file... / Done) workflow this skill replicates programmatically
  repo: https://github.com/dietvin/pod5Viewer
- name: pod5
  role: Python library (v0.3.10) used to enumerate read IDs from the input POD5 files
- name: Python
  role: Runtime (v3.10.0) in which the enumeration, substring search, sorting, and
    intersection logic is implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:e7f0d5832cecb88cc5cfae7b13d7c9303569f3f3c5fb3a76450f2c1d45d50d66
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# read-id-subset-intersection-filtering

## Summary

Implements the pod5Viewer read-filtering behavior programmatically: enumerate read IDs from POD5 files with the pod5 reader library (v0.3.10), apply live-style substring search and ascending/descending lexicographic sorting, then intersect with a filter text file containing one read ID per line to retain only reads in the submitted set.

## When to use

Use when a task requires locating or narrowing a subset of reads enumerated from POD5 nanopore files by substring search, lexicographic sorting, or intersection with a supplied list of read IDs (mirroring pod5Viewer's navigator search and File > Filter reads workflow).

## When NOT to use

- Input data is not in POD5 format (e.g. FAST5 or FASTQ) — this skill replicates pod5Viewer's POD5-specific navigator and filter behavior
- Filtering requires signal-level criteria (e.g. length or signal value cutoffs) rather than membership in a supplied read-ID set — pod5Viewer filtering only retains reads found in the given ID set
- A GUI inspection session is sufficient — opening the pod5Viewer and using `File > Filter reads...` directly is simpler than reproducing the workflow in a script

## Inputs

- POD5 file(s) containing raw nanopore reads
- Optional filter text file containing one read ID per line
- Optional search query string for substring matching against read IDs
- Optional sort order specification (ascending or descending lexicographic)

## Outputs

- filtered_read_ids.txt with the resulting read IDs, one per line

## How to apply

First enumerate all read IDs from the input POD5 file using the pod5 reader library (v0.3.10) under Python 3.10, matching how pod5Viewer populates its file navigator panel. Apply the search query as a live substring match against the read IDs, replicating the panel search bar that updates fitting reads in real time. Sort the matched IDs alphabetically in ascending or descending lexicographic order, as the navigator's sort buttons do. If a filter text file is provided (one read ID per line, as loaded via `Load IDs from file...` in pod5Viewer), intersect it with the enumerated list, retaining only reads found in the given set; with no filter, retain the full sorted/searched list. Write the resulting IDs one per line to filtered_read_ids.txt, preserving pod5Viewer's one-ID-per-line filter-file convention.

## Related tools

- **pod5Viewer** (Reference GUI whose navigator-panel search, sort buttons, and `File > Filter reads...` (Load IDs from file... / Done) workflow this skill replicates programmatically) — https://github.com/dietvin/pod5Viewer
- **pod5** (Python library (v0.3.10) used to enumerate read IDs from the input POD5 files)
- **Python** (Runtime (v3.10.0) in which the enumeration, substring search, sorting, and intersection logic is implemented)

## Examples

```
from pod5 import Reader; ids = sorted(r.read_id for r in Reader('file1.pod5').reads()); filt = set(open('filter_ids.txt').read().split()); out = [i for i in ids if i in filt]; open('filtered_read_ids.txt','w').write('\n'.join(map(str,out)))
```

## Evaluation signals

- Every read ID in filtered_read_ids.txt must be an exact string match present in the source POD5 enumeration
- When a filter file is supplied, every output ID must be a member of the submitted filter set (intersection semantics)
- With no filter file, the output must equal the full sorted/searched read list derived from the POD5 file
- Output ordering must match the requested ascending or descending lexicographic sort
- Output file format must contain exactly one read ID per line, matching pod5Viewer's filter-file convention

## Limitations

- pod5Viewer filtering only retains reads that are found in the given ID set; it does not support signal-based or metadata-based filtering
- After filtering, the subset must be cleared (reopen the filter window, press Clear then Done) before all read IDs are shown again
- Search updates in the navigator mirror live substring matching against read IDs; no regex or fuzzy matching is described

## Evidence

- [readme] Specific read IDs can be searched using the search bar. Typing a search query updates the panel to show fitting reads in real-time.: "Specific read IDs can be searched using the search bar. Typing a search query updates the panel to show fitting reads in real-time."
- [readme] Opened files and reads can be sorted alphabetically in ascending or descending order using the respective buttons above the panel.: "Opened files and reads can be sorted alphabetically in ascending or descending order using the respective buttons above the panel."
- [readme] To enable the user to focus on a specific subset of reads in given POD5 files, reads can be filtered to a subset.: "To enable the user to focus on a specific subset of reads in given POD5 files, reads can be filtered to a subset."
- [readme] Afterwards, only reads that are found in the given set are shown.: "Afterwards, only reads that are found in the given set are shown."
- [methods] The pod5Viewer is built in Python (v3.10.0) and relies on the following packages:: "The pod5Viewer is built in Python (v3.10.0) and relies on the following packages:"
