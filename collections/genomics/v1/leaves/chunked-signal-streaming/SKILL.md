---
name: chunked-signal-streaming
description: Use when a script or GUI must access the entire raw signal of a read
  stored in a POD5 file (e.g. via the pod5 v0.3.10 API) and loading every measurement
  at once risks unresponsive behavior or excessive memory use.
when_to_use_negative:
- 'Plotting rather than inspecting/exporting: pod5Viewer subsets long signals to a
  maximum of 10000 measurements per signal (median per bin) for responsive plotting,
  which would lose full-resolution data'
- The signal is already exported (e.g. to .npy or a per-measurement text file) and
  does not need to be re-read from the POD5 file
- Only read metadata/key-value attributes are needed — plain POD5 inspection suffices
  without chunked signal streaming
edam_operation: http://edamontology.org/operation_2409
tools:
- name: pod5Viewer
  role: Reference GUI implementation whose 'View > View signal' function displays
    the complete raw signal of a read in chunks with a scrollbar, and whose 'Export
    signal' menu exports measurements to .npy or text
  repo: https://github.com/dietvin/pod5Viewer
- name: pod5
  role: Python API (v0.3.10) used to open the POD5 file, locate the read by ID, and
    access its raw signal for chunked iteration
- name: numpy
  role: v1.26.4 — concatenates buffered signal chunks into one contiguous array and
    writes it to binary .npy format
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae665
    title: 'Pod5Viewer: a GUI for inspecting raw nanopore sequencing data'
  source_content_hash: sha256:2f209a9d95b3c7f6cbe22ee7e4da6fa322c44a374b71c2a4c3bb096bf76fb3a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dietvin/pod5Viewer
  tools:
  - Python
  - numpy
  - pod5
  - pod5Viewer
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Chunked signal streaming

## Summary

Mirrors pod5Viewer's 'View signal' behavior of reading the complete raw current signal of a nanopore read from a POD5 file in bounded chunks instead of loading all measurements at once, keeping the application responsive and assembling a full contiguous signal array.

## When to use

Use when a script or GUI must access the entire raw signal of a read stored in a POD5 file (e.g. via the pod5 v0.3.10 API) and loading every measurement at once risks unresponsive behavior or excessive memory use.

## When NOT to use

- Plotting rather than inspecting/exporting: pod5Viewer subsets long signals to a maximum of 10000 measurements per signal (median per bin) for responsive plotting, which would lose full-resolution data
- The signal is already exported (e.g. to .npy or a per-measurement text file) and does not need to be re-read from the POD5 file
- Only read metadata/key-value attributes are needed — plain POD5 inspection suffices without chunked signal streaming

## Inputs

- POD5 file containing nanopore read records (raw current signal plus metadata)
- Read ID identifying the target read within the POD5 file

## Outputs

- In-memory chunks of raw current measurements for the read
- Single contiguous numpy array of the complete raw signal
- Optional read_signal.npy binary file in numpy .npy format

## How to apply

Open the POD5 file with the pod5 Python API (v0.3.10) and locate the read whose read ID matches the identifier of interest. Access the read's raw signal and iterate over it in bounded chunks rather than materializing all measurements at once, exactly as pod5Viewer's 'View > View signal' shows 'the current values in chunks' with a scrollbar between them. Append each chunk to an in-memory buffer to assemble the complete signal, then concatenate the buffered chunks into a single contiguous numeric array with numpy (v1.26.4). Optionally persist the array to a .npy file, consistent with pod5Viewer's 'Export signal' option that writes values to a Numpy file. Decide between streaming (chunk-by-chunk display/processing) and full assembly depending on whether the downstream consumer needs a contiguous array; validate that the assembled array length equals the total number of measurements stored for the read in the POD5 file.

## Related tools

- **pod5Viewer** (Reference GUI implementation whose 'View > View signal' function displays the complete raw signal of a read in chunks with a scrollbar, and whose 'Export signal' menu exports measurements to .npy or text) — https://github.com/dietvin/pod5Viewer
- **pod5** (Python API (v0.3.10) used to open the POD5 file, locate the read by ID, and access its raw signal for chunked iteration)
- **numpy** (v1.26.4 — concatenates buffered signal chunks into one contiguous array and writes it to binary .npy format)

## Examples

```
import pod5, numpy as np
with pod5.Reader('file.pod5') as r:
    read = next(r.reads(selection=[target_read_id]))
    signal = np.concatenate([np.asarray(read.signal[i:i+chunk]) for i in range(0, len(read.signal), chunk)])
np.save('read_signal.npy', signal)
```

## Evaluation signals

- The assembled array's element count equals the total number of measurements stored for the read in the POD5 file — no chunks dropped or duplicated
- Chunks are appended in read order so concatenation yields the original signal sequence
- Memory footprint stays bounded by the chunk size during iteration rather than scaling with the whole signal at load time
- np.save output is loadable via np.load and round-trips to an array with identical shape and values
- Behavior matches pod5Viewer's 'View signal' output when scrolled through chunk by chunk

## Limitations

- Chunked display is a viewing aid; for export pod5Viewer writes the isolated measurements, so any full-resolution downstream analysis must reassemble all chunks rather than rely on the on-screen view
- pod5Viewer's plot-based inspection subsets long signals to at most 10000 measurements per signal (10000 bins with per-bin medians), so plotted resolution does not equal the raw measurement count except at high zoom
- Chunking applies to raw current measurements in POD5 format; other nanopore file formats are out of scope

## Evidence

- [readme] View signal chunks in viewer: "Individual current measurement can be inspected via `View > View signal`. This shows the current values in chunks. Use the scroll bar to scroll through chunks."
- [readme] Export signal to npy: "The values can be exported to a `.npy` Numpy file for further processing in a `Numpy` environment. Alternatively, they can be written to a text file, where each line contains one measurement."
- [readme] Performance-driven subsetting rationale: "For faster performance and responsiveness of the window, long signals are subset to a maximum of 10000 measurements per signal at any given point."
- [methods] Software stack versions: "The pod5Viewer is built in Python (v3.10.0) and relies on the following packages: - numpy (v1.26.4) - pod5 (v0.3.10)"
