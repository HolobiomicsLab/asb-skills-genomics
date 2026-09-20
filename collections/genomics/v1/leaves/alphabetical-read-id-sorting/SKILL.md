---
name: alphabetical-read-id-sorting
description: Use when navigating or outputting read IDs from POD5 files and a deterministic
  lexicographic order (ascending or descending) is needed to locate or compare specific
  nanopore reads.
when_to_use_negative:
- Input is not POD5 raw nanopore data or the IDs to sort are not read IDs from a POD5
  enumeration
- A non-lexicographic ordering is required (e.g. by read length, signal quality, or
  file order)
- Interactive signal inspection or plotting is the goal rather than ordered read ID
  listing
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3168
tools:
- name: pod5Viewer
  role: GUI whose file navigator panel exposes sort buttons for alphabetical ascending/descending
    ordering of opened files and their reads
  repo: https://github.com/dietvin/pod5Viewer
- name: pod5
  role: Python library (v0.3.10) used to enumerate read IDs from POD5 files prior
    to sorting
  repo: https://github.com/dietvin/pod5Viewer
- name: Python
  role: Runtime (v3.10.0) in which pod5Viewer is built and in which read enumeration
    and lexicographic sorting are implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:f3ef96a00505f4b92157c141e0ca7ed9742ef17b1a6d4507c1cf176c60aae646
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dietvin/pod5Viewer
  tools:
  - Python
  - pod5
  - pod5Viewer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Alphabetical read ID sorting

## Summary

Sort read IDs enumerated from POD5 nanopore files alphabetically in ascending or descending lexicographic order, mirroring the pod5Viewer file navigator's sort buttons so users can locate specific reads predictably.

## When to use

Use when navigating or outputting read IDs from POD5 files and a deterministic lexicographic order (ascending or descending) is needed to locate or compare specific nanopore reads.

## When NOT to use

- Input is not POD5 raw nanopore data or the IDs to sort are not read IDs from a POD5 enumeration
- A non-lexicographic ordering is required (e.g. by read length, signal quality, or file order)
- Interactive signal inspection or plotting is the goal rather than ordered read ID listing

## Inputs

- POD5 file(s) containing raw nanopore reads with read IDs
- Optional search query string (substring match against read IDs)
- Optional sort direction: ascending or descending
- Optional text file of read IDs for filtering, one read ID per line

## Outputs

- Read ID list sorted alphabetically (ascending or descending), e.g. written one per line to filtered_read_ids.txt

## How to apply

First enumerate all read IDs from the input POD5 file(s) using the pod5 reader library (v0.3.10) under Python 3.10, as the pod5Viewer does when expanding a file entry in its left navigator panel. Apply any live substring search query against the read IDs before sorting, since the panel updates fitting reads in real time. Then sort the matched IDs alphabetically in the requested direction (ascending or descending lexicographic order), replicating the behavior of the sort buttons above the navigator panel. If a read filter (File > Filter reads..., one read ID per line) is active, intersect the sorted list with the submitted ID set so only reads found in the given set are retained. Write the resulting IDs one per line to the output file and validate that each is an exact string match against the source POD5 enumeration.

## Related tools

- **pod5Viewer** (GUI whose file navigator panel exposes sort buttons for alphabetical ascending/descending ordering of opened files and their reads) — https://github.com/dietvin/pod5Viewer
- **pod5** (Python library (v0.3.10) used to enumerate read IDs from POD5 files prior to sorting) — https://github.com/dietvin/pod5Viewer
- **Python** (Runtime (v3.10.0) in which pod5Viewer is built and in which read enumeration and lexicographic sorting are implemented)

## Examples

```
conda create -n p5v python==3.10; conda activate p5v; pip install pod5Viewer; pod5Viewer file1.pod5
```

## Evaluation signals

- Every output read ID is an exact string match present in the source POD5 enumeration
- When a filter file is supplied, every output ID is a member of the submitted filter ID set; with no filter, the output equals the full sorted/searched read list
- Output order matches the requested ascending or descending lexicographic sort (verified by comparing against sorted()/reversed(sorted()) in Python)
- Output format is one read ID per line in the result file (e.g. filtered_read_ids.txt)
- Reads not matching an active substring search query are absent from the output, mirroring the panel's real-time search behavior

## Limitations

- Sorting is purely alphabetical/lexicographic; pod5Viewer offers no numeric or metadata-based read ordering
- In the GUI, the sort applies to opened files and their reads in the navigator panel; it must be replicated programmatically (pod5 v0.3.10, Python 3.10) for scripted use
- When installing via pip, pod5Viewer can only be started from the command line, not by double-clicking POD5 files

## Evidence

- [readme] README sort buttons: "Opened files and reads can be sorted alphabetically in ascending or descending order using the respective buttons above the panel."
- [readme] README search bar: "Specific read IDs can be searched using the search bar. Typing a search query updates the panel to show fitting reads in real-time."
- [readme] README filtering: "After loading, the contents of the file are shown in the text browser. Press `Done` to submit given read IDs for filtering. Afterwards, only reads that are found in the given set are shown."
- [methods] Article navigator description: "Expanding an entry shows all reads in a given file by their read ID."
- [methods] Article dependencies: "The pod5Viewer is built in Python (v3.10.0) and relies on the following packages: - pod5 (v0.3.10)"
