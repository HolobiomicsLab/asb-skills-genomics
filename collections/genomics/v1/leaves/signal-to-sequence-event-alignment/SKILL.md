---
name: signal-to-sequence-event-alignment
description: Use when raw nanopore multi-read FAST5 signal data must be converted
  to single-read FAST5 and resquiggled against a reference FASTA (e.g., rRNA transcripts)
  before per-position current/dwell/stdev analysis.
when_to_use_negative:
- Input data are already basecalled FASTQ or Nanopolish eventalign output is being
  used directly for per-position KS computation — this skill applies to raw-signal
  resquiggling, not sequence-space alignment.
- The reference sequence is not the same transcript molecule as the sequenced reads
  (resquiggle requires an accurate reference FASTA of the sequenced RNA).
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: multi_to_single_fast5
  role: Converts multi-read FAST5 files to single-read FAST5 files prior to Tombo
    processing
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: tombo
  role: Resquiggles raw nanopore signal onto the reference FASTA, writing aligned
    event tables into single FAST5 files
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: Nanopolish
  role: Alternative raw-signal-to-sequence aligner used for cross-tool concordance
    validation of the resquiggle step
provenance:
  source_task_ids:
  - task_007
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.xgen.2022.100097
    title: Direct detection of RNA modifications and structure using single-molecule
      nanopore sequencing
  source_content_hash: sha256:7a9f52b1b0f568a0fde8f25cce234aa3c01ac14cd2d491fd380f05142bee952f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/physnano/rRNA_nanoSHAPE
  tools:
  - Nanopolish
  - multi_to_single_fast5
  - python
  - tombo
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# signal-to-sequence-event-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns raw nanopore ionic-current events from single-read FAST5 files onto a transcript reference sequence using Tombo resquiggle, producing per-event signal-to-base assignments needed for position-by-position modification comparison.

## When to use

Use when raw nanopore multi-read FAST5 signal data must be converted to single-read FAST5 and resquiggled against a reference FASTA (e.g., rRNA transcripts) before per-position current/dwell/stdev analysis.

## When NOT to use

- Input data are already basecalled FASTQ or Nanopolish eventalign output is being used directly for per-position KS computation — this skill applies to raw-signal resquiggling, not sequence-space alignment.
- The reference sequence is not the same transcript molecule as the sequenced reads (resquiggle requires an accurate reference FASTA of the sequenced RNA).

## Inputs

- Directory of multi-read FAST5 nanopore files (native/modified and IVT control)
- Prepared transcript reference FASTA (e.g., E. coli 16S/23S, S. cerevisiae 18S/25S rRNA sequences)
- flowcell and kit specifications for multi_to_single_fast5

## Outputs

- Directory of single-read FAST5 files with resquiggled, signal-to-sequence aligned event tables embedded
- Event mean and event standard deviation values stored per read in the FAST5 files (via --include-event-stdev)
- Aligned signal data ready for tombo detect_modifications model_sample_compare and extract_array.py .npy extraction

## How to apply

First convert multi-read FAST5 files with multi_to_single_fast5 (--recursive, --save_path, --fast5_out) so Tombo can process individual reads. Run 'tombo resquiggle <single-fast5 dir> <reference FASTA> --include-event-stdev --num-most-common-errors 5 --ignore-read-locks --overwrite --processes <N>' so that resquiggled event tables aligned to the reference sequence are written back into the FAST5 files. The --include-event-stdev flag is essential because downstream feature extraction (extract_array.py with --current/--dwell/--stdev) requires event standard deviations stored alongside event means. Use --ignore-read-locks and --overwrite so stale locks do not skip reads. This resquiggling underlies both tombo detect_modifications model_sample_compare KS statistics and custom per-read array extraction, and was validated against Nanopolish eventalign to within ±2 nt of known modified sites.

## Related tools

- **multi_to_single_fast5** (Converts multi-read FAST5 files to single-read FAST5 files prior to Tombo processing) — github.com/physnano/rRNA_nanoSHAPE
- **tombo** (Resquiggles raw nanopore signal onto the reference FASTA, writing aligned event tables into single FAST5 files) — github.com/physnano/rRNA_nanoSHAPE
- **Nanopolish** (Alternative raw-signal-to-sequence aligner used for cross-tool concordance validation of the resquiggle step)

## Examples

```
tombo resquiggle <single-fast5-dir> <reference.fasta> --include-event-stdev --num-most-common-errors 5 --ignore-read-locks --overwrite --processes 8
```

## Evaluation signals

- Resquiggled event tables are written into every single FAST5 file (no reads skipped due to locks, since --ignore-read-locks and --overwrite were set).
- Event standard deviations are present alongside event means in the FAST5 files, confirming --include-event-stdev took effect.
- Tombo and Nanopolish raw-signal-to-sequence alignments agree: downstream KS profiles correlate at Pearson r = 0.75–0.79 for current and r = 0.45–0.55 for dwell time, with modification positions identified to within ±2 nt of known sites.
- Downstream extract_array.py successfully produces per-read .npy arrays (current/dwell/stdev) from the resquiggled FAST5 directory.

## Limitations

- Signal discrimination is a function of kmer sequence context and the ability to align raw current, so alignment quality limits modification detection.
- No training sets of known modifications in all possible kmer sequence contexts exist, so modification identification requires comparison with an IVT control.
- Without --overwrite and --ignore-read-locks, reads can be skipped due to stale read locks.

## Evidence
