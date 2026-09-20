---
name: wall-clock-runtime-measurement
description: Use when comparing pipeline execution modes (piped/on-the-fly vs intermediary-file)
  and you need quantitative evidence — wall-clock seconds and bytes written to disk
  — that streaming avoids unnecessary disk I/O.
when_to_use_negative:
- The goal is consensus accuracy or variant-calling fidelity rather than runtime and
  disk-I/O benchmarking of pipeline wiring
- The reference genome is ultra-long (e.g. human chromosomes) — memory scales linearly
  with reference length, so runtime benchmarking on such inputs may be dominated by
  memory constraints rather than I/O mode
- Only a single execution mode is available (no intermediary-file counterpart to compare
  against), so no disk-I/O reduction can be measured
tools:
- name: minimap2
  role: Short-read mapper whose SAM output (-t 1 -a -x sr) is timed in both piped
    and intermediary-file modes
- name: ViralConsensus
  role: On-the-fly consensus caller that accepts the mapper stream via standard input
    (-i -), enabling the zero-intermediary-file execution mode being benchmarked
  repo: https://github.com/niemasd/ViralConsensus
- name: samtools
  role: Used in the tee-based streaming variant to archive the mapper stream as BAM
    ('samtools view -b -@ 1 > reads.bam') while ViralConsensus consumes the other
    path
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:b59b5e69ee6aa79ff968f292d5013f2fac423da111f62d4435a1f3894ae8a745
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niemasd/ViralConsensus
  tools:
  - ViralConsensus
  - minimap2
  - samtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# wall-clock-runtime-measurement

## Summary

Benchmark the wall-clock time and disk bytes written of alternative pipeline wiring modes (e.g. streaming a read mapper's SAM output directly into ViralConsensus via a pipe versus writing an intermediary alignment file), using /usr/bin/time so that disk-I/O savings claims can be quantified rather than asserted.

## When to use

Use when comparing pipeline execution modes (piped/on-the-fly vs intermediary-file) and you need quantitative evidence — wall-clock seconds and bytes written to disk — that streaming avoids unnecessary disk I/O.

## When NOT to use

- The goal is consensus accuracy or variant-calling fidelity rather than runtime and disk-I/O benchmarking of pipeline wiring
- The reference genome is ultra-long (e.g. human chromosomes) — memory scales linearly with reference length, so runtime benchmarking on such inputs may be dominated by memory constraints rather than I/O mode
- Only a single execution mode is available (no intermediary-file counterpart to compare against), so no disk-I/O reduction can be measured

## Inputs

- Paired-end reads in FASTQ (reads.fastq.gz)
- Single-sequence reference genome in FASTA (reference.fas)
- minimap2 SAM alignment output (streamed via stdin or written to an intermediary file)

## Outputs

- Consensus genome FASTA (consensus.fas) from each execution mode
- Benchmark table (e.g. benchmark_results.tsv) with wall-clock seconds and disk bytes written per mode
- Optionally an archived BAM (reads.bam) from the tee-based streaming variant

## How to apply

Set up both variants of the workflow over identical inputs (e.g. paired-end reads.fastq.gz and a single-sequence reference.fas). For the piped mode, run minimap2 with parameters -t 1 -a -x sr on the reference and reads and pipe the SAM stream via standard input directly into viral_consensus (-i - -r reference.fas -o consensus.fas), wrapping the command in /usr/bin/time to record wall-clock time and capture disk bytes written. For the two-step mode, write the same minimap2 SAM output to a file first, then invoke viral_consensus on that alignment file, recording the same metrics (optionally splitting the stream with tee so one path feeds viral_consensus and the other feeds 'samtools view -b -@ 1 > reads.bam' to emulate pipelines that also archive the BAM). Tabulate wall-clock seconds and disk bytes written per mode (e.g. into benchmark_results.tsv), then validate that the piped mode writes fewer bytes to disk than the two-step mode while producing a consensus FASTA of identical format.

## Related tools

- **minimap2** (Short-read mapper whose SAM output (-t 1 -a -x sr) is timed in both piped and intermediary-file modes)
- **ViralConsensus** (On-the-fly consensus caller that accepts the mapper stream via standard input (-i -), enabling the zero-intermediary-file execution mode being benchmarked) — https://github.com/niemasd/ViralConsensus
- **samtools** (Used in the tee-based streaming variant to archive the mapper stream as BAM ('samtools view -b -@ 1 > reads.bam') while ViralConsensus consumes the other path) — https://github.com/samtools/htslib

## Examples

```
/usr/bin/time -v minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas
```

## Evaluation signals

- Piped mode writes fewer bytes to disk than the two-step mode for the same reads and reference
- Both modes produce a consensus FASTA in identical format (same reference coordinate span and FASTA structure)
- Wall-clock seconds and bytes-written are recorded for every mode in benchmark_results.tsv with no missing rows
- Timing is captured via /usr/bin/time on every run rather than estimated
- The tee-based variant still delivers the full mapper stream to viral_consensus while also producing a valid reads.bam

## Limitations

- ViralConsensus memory consumption scales linearly with reference genome length, so ultra-long references (e.g. human chromosomes) may require prohibitively large memory and confound runtime comparisons
- The measured I/O advantage applies to pipelines where an intermediary alignment file would otherwise be written; pipelines that must archive the alignment anyway still write the BAM (via tee) and will not realize the full savings
- The web app port suits reasonably small datasets only; benchmarking on larger datasets requires the command-line tool

## Evidence

- [methods] Piped invocation from methods: "minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas"
- [methods] Rationale: avoiding disk I/O by piping: "data can be piped directly from the read mapper to ViralConsensus to avoid unnecessary disk I/O"
- [readme] On-the-fly streaming via standard input: "ViralConsensus can pipe data directly from a read mapper via standard input and performs viral consensus calling on-the-fly"
- [methods] tee for multi-path streaming: "`tee` should be used instead to send the stream down two (or more) paths"
