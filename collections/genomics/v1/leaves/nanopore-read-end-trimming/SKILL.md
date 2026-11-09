---
name: nanopore-read-end-trimming
description: Use when raw Oxford Nanopore FASTQ reads contain PCR/tiling primer and
  adapter sequences at read ends and a fixed-length trim of both ends is needed before
  downstream classification or assembly.
when_to_use_negative:
- Input reads are already primer- and adapter-trimmed (e.g. ONT basecaller output
  with adapters removed) — a fixed-length double-end trim would unnecessarily remove
  biological sequence
- Primer positions vary per read (e.g. heterogeneous tiling schemes) so a fixed --trim_length
  cut from both ends is inappropriate and coordinate-aware primer trimming is required
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: seqtk
  role: Performs the fixed-length end trimming of both read ends (trimfq-style cut)
    on the nanopore FASTQ input
  repo: https://github.com/lh3/seqtk
- name: Nextflow
  role: Wraps the trimming step as the first process of the ViMOP pipeline, invoked
    with a single command
  repo: https://github.com/opr-group-bnitm/vimop
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:49da2809fbb4201fbb06a1075dadeee3a37d2d9b219a0d8b2c8646471a11f56c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/seqtk
  tools:
  - Nextflow
  - ViMOP
  - seqtk
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# nanopore-read-end-trimming

## Summary

Primer/adapter end trimming for untargeted nanopore FASTQ reads as implemented at the start of the ViMOP pipeline, using seqtk to cut a user-defined number of bases from both the 3' and 5' ends of every read.

## When to use

Use when raw Oxford Nanopore FASTQ reads contain PCR/tiling primer and adapter sequences at read ends and a fixed-length trim of both ends is needed before downstream classification or assembly.

## When NOT to use

- Input reads are already primer- and adapter-trimmed (e.g. ONT basecaller output with adapters removed) — a fixed-length double-end trim would unnecessarily remove biological sequence
- Primer positions vary per read (e.g. heterogeneous tiling schemes) so a fixed --trim_length cut from both ends is inappropriate and coordinate-aware primer trimming is required

## Inputs

- Raw Oxford Nanopore FASTQ file(s) from --fastq input directory (possibly gzipped, organized in barcode01, barcode02, ... sample subdirectories)
- User-specified --trim_length (number of bases to remove from both read ends)

## Outputs

- Trimmed FASTQ file(s) (e.g. trimmed_reads.fastq) with original headers and qualities of the retained portion
- Per-sample trimmed read sets for subsequent ViMOP steps (Centrifuge classification, filtering, assembly)

## How to apply

Load the raw nanopore FASTQ input passed via --fastq (optionally organized in barcode01, barcode02, ... subdirectories treated as separate samples). Apply a seqtk trimfq-style end trim that removes exactly --trim_length bases from each end of every read, stripping the primer and adapter sequences flanking the viral insert. Write the trimmed reads to a FASTQ file that preserves the original read headers and the base qualities of the retained internal portion. Rationale: in ViMOP this step is the mandatory first stage before Centrifuge classification and host/contaminant removal, because leftover primer bases would corrupt taxonomic assignment and downstream reference-guided assembly.

## Related tools

- **seqtk** (Performs the fixed-length end trimming of both read ends (trimfq-style cut) on the nanopore FASTQ input) — https://github.com/lh3/seqtk
- **Nextflow** (Wraps the trimming step as the first process of the ViMOP pipeline, invoked with a single command) — https://github.com/opr-group-bnitm/vimop

## Examples

```
seqtk trimfq -b 5 -e 5 in.fq > trimmed_reads.fastq  # trim_length=5 bases from each end
```

## Evaluation signals

- trimmed_reads.fastq exists and parses as valid 4-line FASTQ
- Each trimmed read is exactly 2 x trim_length bases shorter than its input counterpart
- Read headers are unchanged between input and trimmed output
- Base qualities of the retained internal portion are identical to the input FASTQ
- Number of reads in output equals number of reads in input (no reads dropped or duplicated)

## Limitations

- Fixed-length trimming cannot adapt to variable-length primer or adapter sequences; reads with inserts shorter than 2 x trim_length are over-trimmed or lost
- trim_length must be set correctly by the user for the primer scheme in use; an incorrect value leaves primer bases or removes viral sequence

## Evidence

- [methods] Read trimming method: "At the beginning the pipeline trims the ends of the reads to remove adapter sequences and primers."
- [readme] trim_length parameter: "`--trim_length` sets the number of bases trimmed from both ends."
- [readme] Input organization: "The directory can contain subdirectories called barcode01, barcode02, ... In this case, the different barcodes are treated as different samples"
