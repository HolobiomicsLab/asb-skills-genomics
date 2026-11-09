---
name: per-read-event-alignment-to-reference
description: Use when tombo-resquiggled single fast5 files for a MODIFIED or UNMODIFIED
  direct RNA nanopore sample must be stacked into a 2D (num_reads x target_rna_length)
  signal array for per-read modification analysis.
when_to_use_negative:
- Input is still multi-read fast5 files — these must first be converted with multi_to_single_fast5
  and resquiggled with tombo before per-read arrays can be extracted
- The goal is population-level modification detection statistics rather than per-read
  single-molecule signal arrays — use 'tombo detect_modifications model_sample_compare'
  directly
- The reads have not been resquiggled against the reference FASTA, so raw signal events
  are not yet aligned to reference positions
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: tombo
  role: Resquiggles raw nanopore signal events to the FASTA reference (with --include-event-stdev)
    so per-base current/dwell/stdev values are position-anchored before array extraction
  repo: https://nanoporetech.github.io/tombo/
- name: extract_array.py
  role: Custom script that traverses single fast5 files, extracts the user-selected
    per-base feature (--current/--dwell/--stdev), optionally sorts reads, and stacks
    them into the output .npy array
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: python
  role: Runtime for extract_array.py and for downstream loading/plotting of the per-read
    numpy array
  repo: github.com/physnano/rRNA_nanoSHAPE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xgen.2022.100097
    title: Direct detection of RNA modifications and structure using single-molecule
      nanopore sequencing
  source_content_hash: sha256:3b38ce46b49cb5b3b1858c9e9f1f1ff23ed9c9e1a6a2f4c9886821c28991391f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/physnano/rRNA_nanoSHAPE
  tools:
  - extract_array.py
  - python
  - tombo
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# per-read-event-alignment-to-reference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts tombo-resquiggled single-read fast5 files into a per-read by position numpy array of raw-signal features (pore current, dwell time, or current standard deviation) aligned to a target RNA's reference positions, enabling single-molecule RNA modification visualization and plotting.

## When to use

Use when tombo-resquiggled single fast5 files for a MODIFIED or UNMODIFIED direct RNA nanopore sample must be stacked into a 2D (num_reads x target_rna_length) signal array for per-read modification analysis.

## When NOT to use

- Input is still multi-read fast5 files — these must first be converted with multi_to_single_fast5 and resquiggled with tombo before per-read arrays can be extracted
- The goal is population-level modification detection statistics rather than per-read single-molecule signal arrays — use 'tombo detect_modifications model_sample_compare' directly
- The reads have not been resquiggled against the reference FASTA, so raw signal events are not yet aligned to reference positions

## Inputs

- Directory of tombo-resquiggled single fast5 files (MODIFIED or UNMODIFIED/control), produced after multi_to_single_fast5 conversion and resquiggle
- FASTA reference file containing the target RNA sequence
- RNA name of the target transcript as it appears in the FASTA reference
- Choice of per-base signal feature: current, dwell time, or current standard deviation

## Outputs

- 2D numpy array (.npy) of shape (num_reads, target_rna_length) containing the selected per-base signal feature for each read aligned to reference positions

## How to apply

First ensure the reads have been resquiggled with 'tombo resquiggle' against the FASTA reference using --include-event-stdev so that per-base event-level signals are anchored to reference positions. Then run the physnano/rRNA_nanoSHAPE extract_array.py script: load the FASTA reference to retrieve the target RNA sequence (--rna-name), traverse the directory of single fast5 files (--f5, either MODIFIED or UNMODIFIED), and extract the per-base feature selected by the user's flag: --current (pore current), --dwell (dwell time), or --stdev (current standard deviation). Apply --sort-reads to order reads consistently before stacking them, and write the resulting 2D array to --out as a '.npy' file. The rationale is that per-read arrays preserve single-molecule modification context that tombo's population-level per-read statistics plots do not export in a machine-readable form.

## Related tools

- **tombo** (Resquiggles raw nanopore signal events to the FASTA reference (with --include-event-stdev) so per-base current/dwell/stdev values are position-anchored before array extraction) — https://nanoporetech.github.io/tombo/
- **extract_array.py** (Custom script that traverses single fast5 files, extracts the user-selected per-base feature (--current/--dwell/--stdev), optionally sorts reads, and stacks them into the output .npy array) — github.com/physnano/rRNA_nanoSHAPE
- **python** (Runtime for extract_array.py and for downstream loading/plotting of the per-read numpy array) — github.com/physnano/rRNA_nanoSHAPE

## Examples

```
python extract_array.py --fa <FASTA reference file> --f5 <path to single fast5 files, MODIFIED or UNMODIFIED> --out <array path with .npy> --rna-name <RNA name from FASTA> --sort-reads --current
```

## Evaluation signals

- Output .npy file exists and loads cleanly with numpy.load
- Array shape equals (num_reads, target_rna_length) for the specified --rna-name, with no missing positions along the target RNA
- The --f5 input directory contains tombo-resquiggled single fast5 files (not multi-read fast5s), otherwise extraction fails or yields unaligned events
- Only one feature flag (--current, --dwell, or --stdev) is selected per run, and the array values correspond to that feature
- Re-running with --sort-reads produces a consistently row-ordered array across MODIFIED and UNMODIFIED samples for comparable visualization

## Limitations

- Only one per-base feature (current, dwell, or stdev) can be extracted per invocation; multiple features require separate runs
- Reads must already be resquiggled with tombo (with --include-event-stdev) against the reference; raw unaligned fast5 files cannot be used directly
- Signal discrimination remains an outstanding challenge for direct RNA nanopore sequencing modification detection, as it depends on kmer sequence context and raw-current alignment quality
- High rates of AcIm modification lower the number of full-length reads, overall yield, and alignment rates, reducing the number of rows in the output array

## Evidence
