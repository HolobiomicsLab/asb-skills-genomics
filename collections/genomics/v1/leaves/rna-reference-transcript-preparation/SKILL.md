---
name: rna-reference-transcript-preparation
description: Use when single-read FAST5 direct RNA reads must be raw-signal-aligned
  against a reference transcript FASTA (rRNA sequences) so per-read event tables exist
  for Tombo detect_modifications and extract_array.py.
when_to_use_negative:
- Input is still multi-read FAST5 — the workflow requires converting to single-read
  FAST5 via multi_to_single_fast5 before resquiggling
- The target RNA has no matching reference sequence in the FASTA, since Tombo resquiggle
  aligns raw signal to a supplied reference FASTA
- You only need basecalled-read alignment rather than raw-signal-to-sequence alignment;
  resquiggle is specifically for raw nanopore signal mapping
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: tombo
  role: Performs the resquiggle raw-signal-to-reference alignment that writes resquiggled
    event tables back into the single FAST5 files
  repo: https://nanoporetech.github.io/tombo/
- name: multi_to_single_fast5
  role: Converts multi-read FAST5 files into the single-read FAST5 files that tombo
    resquiggle requires as input
  repo: github.com/physnano/rRNA_nanoSHAPE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2022.100097
    title: Direct detection of RNA modifications and structure using single-molecule
      nanopore sequencing
  source_content_hash: sha256:e76a1a41c48897875c31da407ea074790ad4334be290fdd8b5af40c56aae9313
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/physnano/rRNA_nanoSHAPE
  tools:
  - multi_to_single_fast5
  - tombo
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# rna-reference-transcript-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare a reference transcript FASTA (e.g., E. coli 16S/23S or S. cerevisiae 18S/25S rRNA sequences) and resquiggle raw nanopore signal onto it with Tombo, so native-versus-IVT current/dwell/stdev distributions can be compared position-by-position downstream.

## When to use

Use when single-read FAST5 direct RNA reads must be raw-signal-aligned against a reference transcript FASTA (rRNA sequences) so per-read event tables exist for Tombo detect_modifications and extract_array.py.

## When NOT to use

- Input is still multi-read FAST5 — the workflow requires converting to single-read FAST5 via multi_to_single_fast5 before resquiggling
- The target RNA has no matching reference sequence in the FASTA, since Tombo resquiggle aligns raw signal to a supplied reference FASTA
- You only need basecalled-read alignment rather than raw-signal-to-sequence alignment; resquiggle is specifically for raw nanopore signal mapping

## Inputs

- Single-read FAST5 directory produced by multi_to_single_fast5 (for both MODIFIED and CONTROL/UNMODIFIED samples)
- Prepared transcript reference FASTA (e.g., E. coli 16S/23S rRNA, S. cerevisiae 18S/25S rRNA sequences)

## Outputs

- Resquiggled single FAST5 files containing signal-to-sequence aligned event tables with event means and event standard deviations
- Resquiggle-ready FAST5 directories that feed tombo detect_modifications model_sample_compare and extract_array.py

## How to apply

Start from the single-read FAST5 directory produced by multi_to_single_fast5 and the prepared transcript reference FASTA (E. coli 16S/23S, S. cerevisiae 18S/25S). Run 'tombo resquiggle <single-fast5 directory> <reference FASTA> --include-event-stdev --num-most-common-errors 5 --ignore-read-locks --overwrite --processes <num processes>' to align raw ionic-current signal to the reference sequence. Pass --include-event-stdev so event standard deviations are stored alongside event means in each FAST5 file, as required for downstream current/dwell/stdev feature extraction; use --ignore-read-locks and --overwrite so all reads are re-processed rather than skipped due to stale locks. Both unmodified/IVT (control) and native/modified reads must be resquiggled against the same reference, since Tombo detect_modifications model_sample_compare compares the two samples. Confirm that resquiggled event tables (signal-to-sequence aligned) have been written into the single FAST5 files before running detect_modifications or extract_array.py.

## Related tools

- **tombo** (Performs the resquiggle raw-signal-to-reference alignment that writes resquiggled event tables back into the single FAST5 files) — https://nanoporetech.github.io/tombo/
- **multi_to_single_fast5** (Converts multi-read FAST5 files into the single-read FAST5 files that tombo resquiggle requires as input) — github.com/physnano/rRNA_nanoSHAPE

## Examples

```
tombo resquiggle <path to the directory containing single fast5 files> <path to fasta reference> --include-event-stdev --num-most-common-errors 5 --ignore-read-locks --overwrite --processes <num processes>
```

## Evaluation signals

- Resquiggled event tables (signal-to-sequence aligned) are present in the single FAST5 files after the run
- Event standard deviations are stored alongside event means in each FAST5 file (--include-event-stdev was applied), as required for current/dwell/stdev feature extraction
- All reads were re-processed rather than skipped due to stale locks (--ignore-read-locks and --overwrite were in effect)
- Downstream tombo detect_modifications model_sample_compare and extract_array.py run without missing-input errors on the resquiggled FAST5 directories
- Raw-signal-to-sequence alignment with Tombo (or Nanopolish) reproduces known rRNA modification positions to within ±2 nt

## Limitations

- Signal discrimination depends on the kmer sequence context and the ability to align raw current, which remains an outstanding challenge for direct RNA nanopore sequencing
- High rates of AcIm modification lower the number of full-length reads, overall yield, and alignment rates
- nanoSHAPE is limited by poor resolution of reactivity profiles at the 5' ends of longer RNA molecules

## Evidence
