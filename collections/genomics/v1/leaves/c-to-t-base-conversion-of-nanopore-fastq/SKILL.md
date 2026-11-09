---
name: c-to-t-base-conversion-of-nanopore-fastq
description: Use when preprocessing raw nanopore FASTQ reads from enzymatic methyl-seq
  (EMseq/nanoEM) data so that C-to-T-converted reads can be aligned to a C-to-T converted
  reference genome.
when_to_use_negative:
- Input is direct nanopore WGS data read without base conversion (e.g. for nanopolish
  signal-level methylation calling) — converting reads there would destroy the native
  signal context
- Data are already base-converted short-read bisulfite/enzymatic methyl-seq reads
  processed by tools like Trim Galore/Bismark
- You are converting the reference genome, not reads — that step uses convert_ref.py
  producing output.fa
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: Python3
  role: Implementation language of the src/convert_reads.py conversion script
  repo: https://github.com/yos-sk/nanoEM
- name: pysam
  role: Library used by convert_reads.py to read and write FASTQ read sets
  repo: https://github.com/yos-sk/nanoEM
- name: nanopore sequencing
  role: Source of the raw long reads (PromethION) that undergo C-to-T/G-to-A conversion
- name: EMseq (NEBNext Enzymatic Methyl-seq)
  role: Wet-lab enzymatic conversion whose in-silico counterpart this script reproduces
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:7af9a05e64f205eda33723e59ab50f433a6fedf62dcdc6256134aad1c979685b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yos-sk/nanoEM
  repo_url: https://github.com/yos-sk/nanoEM
  tools:
  - EMseq (NEBNext Enzymatic Methyl-seq)
  - Python3
  - nanopore sequencing
  - pysam
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# c-to-t-base-conversion-of-nanopore-fastq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In-silico bismark-like base conversion of nanoEM nanopore reads: all C/Gs in sequencing reads are converted to T/As (C-to-T) with a companion G-to-A converted FASTQ, mirroring the EMseq enzymatic conversion where unmethylated cytosines are read as thymine prior to alignment against a converted reference.

## When to use

Use when preprocessing raw nanopore FASTQ reads from enzymatic methyl-seq (EMseq/nanoEM) data so that C-to-T-converted reads can be aligned to a C-to-T converted reference genome.

## When NOT to use

- Input is direct nanopore WGS data read without base conversion (e.g. for nanopolish signal-level methylation calling) — converting reads there would destroy the native signal context
- Data are already base-converted short-read bisulfite/enzymatic methyl-seq reads processed by tools like Trim Galore/Bismark
- You are converting the reference genome, not reads — that step uses convert_ref.py producing output.fa

## Inputs

- raw nanoEM nanopore FASTQ read file (e.g. from PromethION 1D pass reads, 3.4-7.6 kb N50)
- Python3 environment with pysam installed

## Outputs

- *_CT.fq.gz — C-to-T converted gzip-compressed FASTQ
- *_GA.fq.gz — G-to-A converted gzip-compressed FASTQ

## How to apply

Load the raw nanoEM nanopore FASTQ with the pipeline's conversion script src/convert_reads.py (Python3 with pysam). Apply the C-to-T substitution to read sequences, mimicking the EMseq enzymatic conversion in which unmethylated cytosines are read as thymine. Generate the companion G-to-A converted FASTQ representing the opposite strand orientation. Write both converted read sets as gzip-compressed FASTQ, emitting *_CT.fq.gz and *_GA.fq.gz as declared by the pipeline; these two files are then mapped with minimap2 (map-ont preset, --eqx) against output.fa, the converted reference. Preserve read names and the same read set as the input FASTQ so downstream best_align.py can reconcile the two alignments per read.

## Related tools

- **Python3** (Implementation language of the src/convert_reads.py conversion script) — https://github.com/yos-sk/nanoEM
- **pysam** (Library used by convert_reads.py to read and write FASTQ read sets) — https://github.com/yos-sk/nanoEM
- **nanopore sequencing** (Source of the raw long reads (PromethION) that undergo C-to-T/G-to-A conversion)
- **EMseq (NEBNext Enzymatic Methyl-seq)** (Wet-lab enzymatic conversion whose in-silico counterpart this script reproduces)

## Examples

```
python src/convert_reads.py fastq
```

## Evaluation signals

- *_CT.fq.gz and *_GA.fq.gz exist as gzip-compressed FASTQ files after the run
- Sequence content of converted files reflects C-to-T (and G-to-A) substitution relative to the input reads
- The converted files contain the same read set (read names/number of records) as the input FASTQ
- Converted reads align to the converted reference (output.fa) with minimap2 map-ont --eqx, producing 1.sorted.bam and 2.sorted.bam without systematic unmapped reads

## Limitations

- nanoEM cannot distinguish between 5-methylcytosine and 5-hydroxymethylcytosine on the same DNA molecule, so conversion treats both identically
- Conversion maps mostly C-T converted bases onto generally error-prone nanopore data, requiring the companion best_align.py step to choose the best of the CT/GA alignments
- No software version numbers are stated for the pipeline scripts in the article or README

## Evidence

- [methods] Base conversion of nanoEM data: "### Base conversion of nanoEM data

OUTPUT:
- *_CT.fq.gz: C-to-T converted fastq
- *_GA.fq.gz: G-to-A converted fastq"
- [results] Bismark-like read conversion: [evidence span withheld — anchor status: not_found]
- [abstract] Enzymatic basis of conversion: "we combined recently developed enzymatic base conversion, where an unmethylated cytosine (C) should be converted to thymine (T), with nanopore sequencing"
- [readme] README invocation: "python src/convert_reads.py fastq"
- [methods] Downstream alignment of converted reads: "minimap2 -t 8 --split-prefix temp_sam1 -ax map-ont output.fa  *_CT.fq.gz --eqx"
