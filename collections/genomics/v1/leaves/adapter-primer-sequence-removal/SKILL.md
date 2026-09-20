---
name: adapter-primer-sequence-removal
description: Use when starting a ViMOP-style untargeted viral nanopore analysis on
  raw ONT FASTQ reads that still carry adapters and primers at both ends and a fixed
  number of bases must be removed per end via --trim_length.
when_to_use_negative:
- Input FASTQ already had adapters/primers removed during basecalling — cutting --trim_length
  bases again would remove biological sequence
- Adapter/primer sequences vary in length or position, requiring adapter-aware scanning
  rather than a fixed number of bases trimmed from both ends
- The goal is quality-based end trimming (e.g. seqtk trimfq with the Phred algorithm)
  rather than removal of a known fixed primer length
edam_operation: http://edamontology.org/operation_3238
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0196
tools:
- name: ViMOP
  role: Pipeline that runs read trimming as its first workflow step, exposing --trim_length
    to set the number of bases trimmed from both ends
  repo: https://github.com/opr-group-bnitm/vimop
- name: seqtk
  role: Fast, lightweight FASTA/FASTQ processing tool invoked to cut the specified
    number of bases from the 3' and 5' ends of each read
  repo: https://github.com/lh3/seqtk
- name: Nextflow
  role: Workflow engine that executes the ViMOP pipeline containing the trimming step
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:947f3658233f6804f550e361573a347b2e47e0b37ef85909f39ed50e7796a4cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Nextflow
  - ViMOP
  - seqtk
  edam_operation: http://edamontology.org/operation_3238
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# adapter-primer-sequence-removal

## Summary

The first workflow step of ViMOP, which cuts a user-defined number of bases (--trim_length) from both the 3' and 5' ends of raw untargeted nanopore FASTQ reads using seqtk, to strip adapter and primer sequences before taxonomic classification and assembly.

## When to use

Use when starting a ViMOP-style untargeted viral nanopore analysis on raw ONT FASTQ reads that still carry adapters and primers at both ends and a fixed number of bases must be removed per end via --trim_length.

## When NOT to use

- Input FASTQ already had adapters/primers removed during basecalling — cutting --trim_length bases again would remove biological sequence
- Adapter/primer sequences vary in length or position, requiring adapter-aware scanning rather than a fixed number of bases trimmed from both ends
- The goal is quality-based end trimming (e.g. seqtk trimfq with the Phred algorithm) rather than removal of a known fixed primer length

## Inputs

- Raw nanopore FASTQ file(s) (optionally gzipped) from the directory passed via --fastq, possibly organized in barcode01, barcode02, ... per-sample subdirectories
- Integer parameter --trim_length: number of bases to trim from both ends of each read

## Outputs

- Trimmed FASTQ file(s) (trimmed_reads.fastq) preserving original read headers and the base qualities of the retained internal portion
- Per-sample trimmed read sets ready for Centrifuge classification and de novo assembly

## How to apply

Load the raw nanopore FASTQ file(s) passed via --fastq, which may be organized in barcode01, barcode02, ... subdirectories treated as separate samples. Run the seqtk-based end-trimming step that removes exactly --trim_length bases from both read ends, since the reads begin with adapter sequences and primers. Preserve the original read headers and the base qualities of the retained internal portion in the trimmed FASTQ output. Feed the trimmed reads into the downstream ViMOP steps (Centrifuge classification, host/contaminant removal, de novo assembly). Choose --trim_length to match the known length of the primers/adapters used during sequencing, as an incorrect value leaves primer bases or removes biological sequence.

## Related tools

- **ViMOP** (Pipeline that runs read trimming as its first workflow step, exposing --trim_length to set the number of bases trimmed from both ends) — https://github.com/opr-group-bnitm/vimop
- **seqtk** (Fast, lightweight FASTA/FASTQ processing tool invoked to cut the specified number of bases from the 3' and 5' ends of each read) — https://github.com/lh3/seqtk
- **Nextflow** (Workflow engine that executes the ViMOP pipeline containing the trimming step)

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq ./fastq/ --trim_length 20
```

## Evaluation signals

- Each trimmed read is exactly 2 x trim_length bases shorter than its input counterpart
- The trimmed output parses as valid 4-line FASTQ with the same number of records as the input
- Original read headers and the base qualities of the retained internal portion are unchanged between input and trimmed FASTQ
- Downstream Centrifuge classification no longer attributes leading/trailing primer bases, and reads align cleanly against the viral references

## Limitations

- Trimming is a fixed base-count cut from both ends, so it cannot adapt to reads with variable adapter lengths, and short reads may be over-trimmed
- If --trim_length does not match the actual primer/adapter length, residual primer bases remain or real sequence is lost
- ViMOP users are advised to carefully review outputs (e.g. alignment .bam files), as the pipeline is designed for small and medium-size RNA viruses and drafts for large DNA viruses like mpox may be inaccurate

## Evidence

- [methods] Read trimming: "At the beginning the pipeline trims the ends of the reads to remove adapter sequences and primers."
- [readme] trim_length parameter: "`--trim_length` sets the number of bases trimmed from both ends."
- [readme] seqtk scope: [evidence span withheld — anchor status: external_doc]
- [readme] input organization: "A directory with fastq files is passed to `--fastq`. The directory can contain subdirectories called barcode01, barcode02, ... In this case, the different barcodes are treated as different samples"
