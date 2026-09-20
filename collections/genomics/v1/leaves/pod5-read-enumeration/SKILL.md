---
name: pod5-read-enumeration
description: Use when the input is one or more POD5 files and the goal is to enumerate,
  search, sort, or filter the contained reads by their read_id — e.g. to produce a
  text list of read IDs or validate a read subset against a POD5 file.
when_to_use_negative:
- The input is not a POD5 file (e.g. FAST5 or FASTQ) — this skill relies on the pod5
  package's Reader and POD5 read_id attributes
- The goal is to inspect or plot the signal data of individual reads rather than enumerate
  them — pod5Viewer's viewing/plotting workflow (`View > Plot signal...`) is the appropriate
  path
- The reads of interest are already listed in an existing ID file and no validation
  against the POD5 file is needed
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: pod5Viewer
  role: Reference GUI whose file navigator enumerates reads by read ID when a loaded
    POD5 file entry is expanded, with search, sorting, and read-ID-set filtering;
    the enumeration logic mirrors its behavior
  repo: https://github.com/dietvin/pod5Viewer
- name: pod5
  role: Python package (v0.3.10) providing the Reader used to open the POD5 file and
    iterate over its reads to extract read_id values
  repo: https://github.com/dietvin/pod5Viewer
- name: Python 3.10
  role: Runtime (v3.10.0) in which pod5Viewer is built and in which the enumeration
    script executes
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:ec8ba9eb2b3179c132e00ef6a2fa966d3f9816be8a91ee79f6627834e2f5b472
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dietvin/pod5Viewer
  tools:
  - Python
  - Python 3.10
  - pod5
  - pod5Viewer
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# pod5-read-enumeration

## Summary

Enumerates all reads stored in one or more nanopore POD5 files by extracting each read's read_id, replicating what pod5Viewer's file navigator shows when a file entry is expanded. Supports subsequent substring search, lexicographic sorting, and set-based filtering to a submitted read-ID subset.

## When to use

Use when the input is one or more POD5 files and the goal is to enumerate, search, sort, or filter the contained reads by their read_id — e.g. to produce a text list of read IDs or validate a read subset against a POD5 file.

## When NOT to use

- The input is not a POD5 file (e.g. FAST5 or FASTQ) — this skill relies on the pod5 package's Reader and POD5 read_id attributes
- The goal is to inspect or plot the signal data of individual reads rather than enumerate them — pod5Viewer's viewing/plotting workflow (`View > Plot signal...`) is the appropriate path
- The reads of interest are already listed in an existing ID file and no validation against the POD5 file is needed

## Inputs

- One or more nanopore POD5 files (readable with the pod5 Python package v0.3.10)
- Optional plain-text filter file containing one read ID per line (as accepted by pod5Viewer's `File > Filter reads...` / `Load IDs from file...`)
- Optional search query (substring of a read ID)
- Optional sort order (ascending or descending lexicographic)

## Outputs

- read_ids.txt: all read IDs from the input POD5 file(s), one per line, in reader order
- filtered_read_ids.txt: read IDs remaining after search/sort/filter, one per line

## How to apply

Open each POD5 file with the pod5 Python package (v0.3.10) Reader, mirroring pod5Viewer's `File > Open file(s)...` behavior, and iterate over all reads to enumerate every stored read. Extract the read_id attribute from each read, since expanding a file's entry in the pod5Viewer navigator 'shows all reads in a given file by their read ID'. Optionally apply a live substring match against the read IDs (as the navigator's search bar does), sort the matches alphabetically in ascending or descending order (as the navigator's sort buttons do), and, if a filter file with one read ID per line is supplied (as in `File > Filter reads...`), intersect the enumeration with that ID set. Write the resulting read IDs one per line to the output file, preserving the reader's order unless sorting is requested.

## Related tools

- **pod5Viewer** (Reference GUI whose file navigator enumerates reads by read ID when a loaded POD5 file entry is expanded, with search, sorting, and read-ID-set filtering; the enumeration logic mirrors its behavior) — https://github.com/dietvin/pod5Viewer
- **pod5** (Python package (v0.3.10) providing the Reader used to open the POD5 file and iterate over its reads to extract read_id values) — https://github.com/dietvin/pod5Viewer
- **Python 3.10** (Runtime (v3.10.0) in which pod5Viewer is built and in which the enumeration script executes)

## Examples

```
python -c "import pod5; r = pod5.Reader('file1.pod5'); ids = [read.read_id for read in r.reads()]; open('read_ids.txt','w').write('\n'.join(map(str, sorted(ids))))"
```

## Evaluation signals

- The number of lines in read_ids.txt equals the number of reads the pod5 Reader reports for the file
- Every output line is an exact string match to a read_id present in the source POD5 enumeration
- When a filter file is supplied, every output read ID is a member of the submitted ID set; with no filter, output equals the full (sorted/searched) read list
- Each line is a non-empty, valid read ID string with exactly one ID per line
- Sorting, when requested, yields ascending or descending lexicographic order of the matched IDs

## Limitations

- Only reads present in the supplied POD5 file(s) can be enumerated; IDs in a filter file that do not exist in the file are silently dropped from the output subset
- Sorted/searched output ordering diverges from the reader's native iteration order unless explicitly requested
- pod5 version pinned at v0.3.10 in pod5Viewer (v1.1.2) and Python at v3.10.0; other versions may behave differently

## Evidence

- [readme] All opened files are shown in the left panel. Expanding an entry shows all reads in a given file by their read ID.: "All opened files are shown in the left panel. Expanding an entry shows all reads in a given file by their read ID."
- [readme] Typing a search query updates the panel to show fitting reads in real-time.: "Specific read IDs can be searched using the search bar. Typing a search query updates the panel to show fitting reads in real-time."
- [readme] This opens a new window containing a text browser, in which read IDs can be inserted. Here one read ID must be given per line.: "This opens a new window containing a text browser, in which read IDs can be inserted. Here one read ID must be given per line."
- [methods] The pod5Viewer is built in Python (v3.10.0) and relies on the following packages:: "The pod5Viewer is built in Python (v3.10.0) and relies on the following packages: - numpy (v1.26.4) - pod5 (v0.3.10)"
- [intro] It allows users to open multiple POD5 files, explore their contents, and display detailed data for selected read IDs.: "It allows users to open multiple POD5 files, explore their contents, and display detailed data for selected read IDs."
