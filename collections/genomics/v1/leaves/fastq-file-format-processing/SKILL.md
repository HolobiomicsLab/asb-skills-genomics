---
name: fastq-file-format-processing
description: Use when raw Oxford Nanopore FASTQ reads must have fixed-length primer
  and adapter sequences stripped from both ends before downstream taxonomic classification,
  filtering, or de novo/reference-guided assembly.
when_to_use_negative:
- Input reads are already demultiplexed/trimmed internal read portions without adapter
  or primer sequence at the termini
- Reads need quality-score-dependent or probabilistic trimming rather than a fixed
  number of bases cut from both ends
- The sample is not untargeted nanopore data processed by a primer-based protocol
  (e.g., amplicon-free cDNA or direct RNA runs with different end structure)
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: seqtk
  role: Performs the FASTQ end trimming, cutting a fixed number of bases from both
    the 3' and 5' ends of each read to remove adapter and primer sequences
  repo: https://github.com/lh3/seqtk
- name: ViMOP
  role: Pipeline that orchestrates the trimming step at the beginning of the workflow
    and exposes --trim_length as the user parameter
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Execution engine that runs the ViMOP trimming module within the single-command
    pipeline
  repo: https://www.nextflow.io/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:295a443295f6f906d16d1c8cd7fcbdbb8de59d8b31992b30eca21ce3d8a11cec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/seqtk
  tools:
  - Nextflow
  - ViMOP
  - seqtk
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# fastq-file-format-processing

## Summary

Primer/adapter-end trimming of raw nanopore FASTQ reads as implemented in ViMOP: seqtk removes a user-defined number of bases from both the 3' and 5' ends of every read before taxonomic classification and assembly.

## When to use

Use when raw Oxford Nanopore FASTQ reads must have fixed-length primer and adapter sequences stripped from both ends before downstream taxonomic classification, filtering, or de novo/reference-guided assembly.

## When NOT to use

- Input reads are already demultiplexed/trimmed internal read portions without adapter or primer sequence at the termini
- Reads need quality-score-dependent or probabilistic trimming rather than a fixed number of bases cut from both ends
- The sample is not untargeted nanopore data processed by a primer-based protocol (e.g., amplicon-free cDNA or direct RNA runs with different end structure)

## Inputs

- Raw nanopore FASTQ file(s) (optionally gzip-compressed), supplied via --fastq, optionally organized in barcode01/barcode02/... subdirectories
- User parameter --trim_length (number of bases trimmed from both read ends)

## Outputs

- Trimmed FASTQ file with original read headers preserved and each retained read exactly 2 x trim_length bases shorter than its input counterpart

## How to apply

Load the raw nanopore FASTQ files passed via --fastq; the directory may contain barcode01, barcode02, ... subdirectories that ViMOP treats as separate samples. Apply seqtk end trimming so that exactly --trim_length bases are removed from both ends of each read, since untargeted nanopore runs leave adapter and primer sequences at the read termini. Retain the original read headers and the base qualities of the internal portion and write the result to a FASTQ output. The trimmed reads then feed ViMOP's downstream steps (Centrifuge classification, host/contaminant removal, assembly). Rationale: trimming to a fixed user-specified length is simple and field-applicable, matching the known primer/adapter structure rather than relying only on quality-based trimming.

## Related tools

- **seqtk** (Performs the FASTQ end trimming, cutting a fixed number of bases from both the 3' and 5' ends of each read to remove adapter and primer sequences) — https://github.com/lh3/seqtk
- **ViMOP** (Pipeline that orchestrates the trimming step at the beginning of the workflow and exposes --trim_length as the user parameter) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Execution engine that runs the ViMOP trimming module within the single-command pipeline) — https://www.nextflow.io/

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq ./reads/ --trim_length 20
```

## Evaluation signals

- The trimmed_reads FASTQ output exists and parses as valid FASTQ (4 lines per record, matching headers and quality lengths)
- Every trimmed read is exactly 2 x trim_length bases shorter than its corresponding input read
- Read headers and the base qualities of the retained internal portion are identical to the original FASTQ records
- Read counts are preserved: number of records in the output equals the number in the input FASTQ
- Downstream ViMOP steps (Centrifuge classification, Krona plot) consume the trimmed reads without FASTQ parse errors

## Limitations

- Fixed-length trimming assumes the primer/adapter occupies exactly --trim_length bases at both ends; variable-length adapter carryover or short reads may be over- or under-trimmed
- Centrifuge classification accuracy is limited, but that concern applies to the following filtering step, not trimming itself; trimming is lossless with respect to read count
- Pipeline behavior can vary with Nextflow version (e.g., 'Cannot invoke method optional() on null object' bug with Nextflow 25.10.2, fixed only in 1.0.4), which can break module execution

## Evidence

- [methods] Trimming uses seqtk on read ends: "At the beginning the pipeline trims the ends of the reads to remove adapter sequences and primers."
- [readme] --trim_length parameter from README: "`--trim_length` sets the number of bases trimmed from both ends."
- [readme] Input FASTQ layout with barcodes: "A directory with fastq files is passed to `--fastq`. The directory can contain subdirectories called barcode01, barcode02, ..."
- [other] seqtk FASTQ processing example: [evidence span withheld — anchor status: external_doc]
