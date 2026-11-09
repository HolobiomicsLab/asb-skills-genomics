---
name: read-mapping-stream-piping
description: Use when constructing a viral sequencing pipeline where a read mapper
  (e.g. minimap2 with -t 1 -a -x sr) produces alignment records and you want to reduce
  disk I/O and wall-clock time by streaming records to consensus calling without writing
  an intermediary SAM/BAM file.
when_to_use_negative:
- Your reconstruction requires multiple reference sequences — ViralConsensus builds
  consensus from a single reference sequence
- The reference genome is ultra-long (e.g. a human chromosome) and you lack large-memory
  capacity, since memory consumption scales linearly with reference genome length
- You need to inspect or reprocess the full alignment records downstream (e.g. variant-level
  analysis with sorted, indexed BAMs) rather than just call the consensus
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ViralConsensus
  role: On-the-fly consensus caller that consumes the unsorted alignment stream from
    standard input ('-i -') and writes the FASTA consensus
  repo: https://github.com/niemasd/ViralConsensus
- name: minimap2
  role: Read mapper producing the SAM stream piped into ViralConsensus (parameters
    -t 1 -a -x sr)
- name: samtools
  role: Used with tee to archive the split mapper stream as BAM ('samtools view -b
    -@ 1 > reads.bam') when BAM archiving is also required
- name: htslib
  role: C library ViralConsensus depends on for reading SAM/BAM/CRAM alignment records
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:56e205b0c72aac4b0f1e6b0c5bab89a3dcd55d57740178c079ea1bad7b8a1e37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niemasd/ViralConsensus
  tools:
  - C++
  - ViralConsensus
  - htslib
  - minimap2
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# read-mapping-stream-piping

## Summary

Pipe the read mapper's SAM output stream directly into ViralConsensus via standard input so consensus calling happens on-the-fly, eliminating the intermediate SAM file on disk that two-step alignment-then-consensus workflows require.

## When to use

Use when constructing a viral sequencing pipeline where a read mapper (e.g. minimap2 with -t 1 -a -x sr) produces alignment records and you want to reduce disk I/O and wall-clock time by streaming records to consensus calling without writing an intermediary SAM/BAM file.

## When NOT to use

- Your reconstruction requires multiple reference sequences — ViralConsensus builds consensus from a single reference sequence
- The reference genome is ultra-long (e.g. a human chromosome) and you lack large-memory capacity, since memory consumption scales linearly with reference genome length
- You need to inspect or reprocess the full alignment records downstream (e.g. variant-level analysis with sorted, indexed BAMs) rather than just call the consensus

## Inputs

- Paired-end sequencing reads in FASTQ (e.g. reads.fastq.gz)
- Single-sequence reference genome in FASTA (e.g. reference.fas, such as NC_045512.2.fas for SARS-CoV-2)
- Optionally, an unsorted SAM/BAM/CRAM alignment stream or file via '-i' (file path or '-' for stdin)

## Outputs

- Consensus genome sequence in FASTA format (e.g. consensus.fas)
- Optionally, an archived alignment file (e.g. reads.bam) produced via the tee-split stream
- Benchmark metrics (wall-clock seconds, disk bytes written) when comparing piped vs two-step modes

## How to apply

Run the mapper (minimap2 -t 1 -a -x sr on the reference FASTA and paired-end FASTQ reads) and pipe the resulting SAM stream straight into viral_consensus with '-i -' (standard input), '-r reference.fas', and '-o consensus.fas'. ViralConsensus, written in C++ against htslib, streams unsorted alignment records one at a time without sorting, incrementing base and insertion counts at each reference position when the user-specified base quality threshold is met, then emits the most frequent base per position as a FASTA consensus. If you must also archive the alignments as BAM, use tee to split the mapper stream down two paths: one into viral_consensus and one into 'samtools view -b -@ 1 > reads.bam'. To quantify the benefit, benchmark piped mode against a two-step mode (mapper writes SAM to file, then viral_consensus reads it) with /usr/bin/time, recording wall-clock seconds and disk bytes written into benchmark_results.tsv.

## Related tools

- **ViralConsensus** (On-the-fly consensus caller that consumes the unsorted alignment stream from standard input ('-i -') and writes the FASTA consensus) — https://github.com/niemasd/ViralConsensus
- **minimap2** (Read mapper producing the SAM stream piped into ViralConsensus (parameters -t 1 -a -x sr))
- **samtools** (Used with tee to archive the split mapper stream as BAM ('samtools view -b -@ 1 > reads.bam') when BAM archiving is also required)
- **htslib** (C library ViralConsensus depends on for reading SAM/BAM/CRAM alignment records) — https://github.com/samtools/htslib

## Examples

```
minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas
```

## Evaluation signals

- The piped mode writes fewer bytes to disk than the two-step mode (which writes an intermediary SAM file), as tabulated in benchmark_results.tsv
- The consensus FASTA produced by the piped run is identical in format (and matches ViralConsensus v0.0.1 output) to that produced by the two-step run on the same inputs
- Wall-clock time of the piped run is reduced relative to the two-step run
- With tee, both downstream consumers succeed: viral_consensus yields the same consensus while 'samtools view -b -@ 1 > reads.bam' produces a valid BAM
- The alignment stream is consumed unsorted — no coordinate sorting step is inserted before consensus calling

## Limitations

- Memory consumption scales linearly with reference genome length, so ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory
- The approach requires a single reference sequence for reconstruction; multi-reference workflows are out of scope
- The web app (WebAssembly port) is only suitable for reasonably small datasets; larger datasets require the command-line tool
- If you also archive the stream as BAM, tee is required instead of a simple pipe so the stream is sent down two (or more) paths

## Evidence

- [abstract] Piping capability claim: "unlike existing methods, ViralConsensus can pipe data directly from a read mapper via standard input and performs viral consensus calling on-the-fly"
- [methods] Disk I/O rationale: "data can be piped directly from the read mapper to ViralConsensus to avoid unnecessary disk I/O"
- [readme] Pipeline suitability: "ViralConsensus can pipe data directly from a read mapper via standard input and performs viral consensus calling on-the-fly, making it an ideal tool for viral sequencing pipelines."
- [readme] Memory scaling caveat: "memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory"
- [methods] tee split for BAM archiving: "`tee` should be used instead to send the stream down two (or more) paths"
