---
name: unix-pipeline-stream-splitting
description: Use when building or benchmarking viral sequencing pipelines where a
  mapper's SAM output feeds a consensus caller and you want to eliminate intermediary
  files, compare piped vs. two-step disk I/O, or archive BAM while still streaming.
when_to_use_negative:
- The reference genome is ultra-long (e.g. a human chromosome), since ViralConsensus
  memory consumption scales linearly with reference genome length and may require
  prohibitively large memory
- The downstream tool cannot accept input via standard input, so an intermediary alignment
  file is unavoidable
- You need sorted/indexed BAM with random access, which requires a materialized file
  rather than an on-the-fly stream
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0797
tools:
- name: ViralConsensus
  role: Consensus caller that consumes the mapper's SAM stream on-the-fly via standard
    input ('-i -'), avoiding intermediary files
  repo: https://github.com/niemasd/ViralConsensus
- name: minimap2
  role: Short-read aligner (-t 1 -a -x sr) producing the SAM stream that is piped
    into ViralConsensus
- name: samtools
  role: Used with tee in the streaming variant to archive the BAM ('samtools view
    -b -@ 1 > reads.bam') alongside consensus calling
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:e07081670dc01c9b83b79adc5c6f02965c081277c1f40b58010b925bc3df189f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niemasd/ViralConsensus
  tools:
  - ViralConsensus
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# unix-pipeline-stream-splitting

## Summary

A skill for streaming read-mapper output (SAM from minimap2) directly into a downstream consumer such as ViralConsensus via standard input, optionally splitting the stream with tee, to avoid writing intermediary SAM/BAM files and thereby reduce disk I/O and wall-clock time.

## When to use

Use when building or benchmarking viral sequencing pipelines where a mapper's SAM output feeds a consensus caller and you want to eliminate intermediary files, compare piped vs. two-step disk I/O, or archive BAM while still streaming.

## When NOT to use

- The reference genome is ultra-long (e.g. a human chromosome), since ViralConsensus memory consumption scales linearly with reference genome length and may require prohibitively large memory
- The downstream tool cannot accept input via standard input, so an intermediary alignment file is unavoidable
- You need sorted/indexed BAM with random access, which requires a materialized file rather than an on-the-fly stream

## Inputs

- Paired-end gzipped FASTQ reads (reads.fastq.gz)
- Single-sequence reference genome FASTA (reference.fas)

## Outputs

- Consensus genome FASTA (consensus.fas) identical in format between piped and two-step modes
- Benchmark table of wall-clock seconds and disk bytes written per mode (benchmark_results.tsv)
- Optionally an archived BAM (reads.bam) in the tee-based streaming variant

## How to apply

Run minimap2 with parameters -t 1 -a -x sr on a single-sequence reference (reference.fas) and paired-end gzipped reads (reads.fastq.gz), and pipe the SAM stream via standard input directly into viral_consensus using '-i -' (e.g. '-i - -r reference.fas -o consensus.fas'). As a baseline, run the two-step mode: write the minimap2 SAM output to a file, then invoke viral_consensus on that alignment file. Time each mode with /usr/bin/time and capture bytes written to disk to quantify the I/O savings. For pipelines that must also archive alignments, replace the simple pipe with tee to split the mapper stream down two paths — one into viral_consensus and one into 'samtools view -b -@ 1 > reads.bam' — since tee (not a plain pipe) is required to fan a stream out to multiple consumers. Tabulate wall-clock seconds and disk bytes written per mode (e.g. into benchmark_results.tsv) to demonstrate that the piped mode writes fewer bytes to disk while producing an identical-format consensus FASTA.

## Related tools

- **ViralConsensus** (Consensus caller that consumes the mapper's SAM stream on-the-fly via standard input ('-i -'), avoiding intermediary files) — https://github.com/niemasd/ViralConsensus
- **minimap2** (Short-read aligner (-t 1 -a -x sr) producing the SAM stream that is piped into ViralConsensus)
- **samtools** (Used with tee in the streaming variant to archive the BAM ('samtools view -b -@ 1 > reads.bam') alongside consensus calling) — https://github.com/samtools/htslib

## Examples

```
minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas
```

## Evaluation signals

- Piped mode writes fewer bytes to disk than the two-step (SAM-file) mode, as recorded by /usr/bin/time disk-write accounting
- Consensus FASTA from piped mode is identical-format (and ideally identical in sequence) to the two-step mode output
- Wall-clock time tabulated in benchmark_results.tsv shows the piped mode is not slower than the two-step mode
- In the tee variant, both consumers succeed: viral_consensus emits consensus.fas and samtools produces a valid BAM (reads.bam)

## Limitations

- ViralConsensus memory consumption scales linearly with reference genome length, so ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory
- A plain pipe serves only one consumer; tee must be used to send the stream down two (or more) paths
- The web app (WebAssembly port) works well only for reasonably small datasets; larger datasets need the command-line tool

## Evidence

- [abstract] Piping directly from mapper via stdin: "unlike existing methods, ViralConsensus can pipe data directly from a read mapper via standard input and performs viral consensus calling on-the-fly"
- [methods] Pipe command pattern: "minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas"
- [methods] Avoid unnecessary disk I/O: "data can be piped directly from the read mapper to ViralConsensus to avoid unnecessary disk I/O"
- [methods] tee for stream splitting: "`tee` should be used instead to send the stream down two (or more) paths"
- [readme] Memory limitation caveat: "memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory"
