---
name: read-alignment-subsampling-benchmarking
description: Use when comparing runtime and memory scaling of consensus-calling or
  similar per-read tools across sequencing depths, by generating controlled read-count
  subsamples from a BAM and profiling each run.
when_to_use_negative:
- Reference genome is ultra-long (e.g. human chromosomes), where ViralConsensus memory
  consumption scales linearly with reference length and may be prohibitively large
- You need to benchmark end-to-end read preprocessing pipelines rather than comparing
  consensus-calling time/memory scaling
tools:
- name: ViralConsensus
  role: Consensus-calling tool being benchmarked; fast, memory-efficient C++ tool
    depending on htslib
  repo: https://github.com/niemasd/ViralConsensus
- name: samtools
  role: Read subsampling from the sorted BAM and the samtools indexing/sort step inside
    the iVar comparison pipeline
- name: htslib
  role: Dependency library used by ViralConsensus for reading alignment data
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:34f0e1916001dd4c3244a73d804c450df1ea2e8ac7eca49aa2e1826485f817a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niemasd/ViralConsensus
  tools:
  - ViralConsensus
  - htslib
  - samtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# read-alignment-subsampling-benchmarking

## Summary

A benchmarking skill that subsamples sorted/aligned amplicon reads (e.g. SARS-CoV-2) to controlled mapped-read depths and profiles wall-clock runtime and peak RSS of consensus-calling tools, as done to show ViralConsensus orders of magnitude faster and more memory-efficient than the iVar pipeline.

## When to use

Use when comparing runtime and memory scaling of consensus-calling or similar per-read tools across sequencing depths, by generating controlled read-count subsamples from a BAM and profiling each run.

## When NOT to use

- Reference genome is ultra-long (e.g. human chromosomes), where ViralConsensus memory consumption scales linearly with reference length and may be prohibitively large
- You need to benchmark end-to-end read preprocessing pipelines rather than comparing consensus-calling time/memory scaling

## Inputs

- Sorted, aligned reads (BAM) from the highest-depth amplicon sequencing sample
- Reference FASTA (e.g. NC_045512.2.fas for SARS-CoV-2)
- Profiler (e.g. /usr/bin/time -v) output streams

## Outputs

- Subsampled BAMs at 100, 1K, 10K, 100K, and 1M mapped reads (10 replicates per depth)
- benchmark_results.csv with columns: tool, replicate, mapped-read count, runtime, peak memory
- Consensus FASTA per replicate run

## How to apply

Subsample the sorted, aligned reads of the highest-depth sample with a samtools-based read subsampling approach at five depths (100, 1K, 10K, 100K, 1M mapped reads) with 10 replicates per depth. For each replicate, run ViralConsensus ('viral_consensus -i <subsample.bam> -r <reference.fas> -o <consensus.fas>') under a time-and-memory profiler such as '/usr/bin/time -v' and record wall-clock runtime and peak RSS; likewise run the iVar pipeline (samtools indexing/sort plus iVar consensus calling) under the same profiler. Aggregate the 100 paired measurements (5 depths x 10 replicates x 2 tools) into a benchmark_results.csv with tool, replicate, mapped-read count, runtime, and peak memory columns.

## Related tools

- **ViralConsensus** (Consensus-calling tool being benchmarked; fast, memory-efficient C++ tool depending on htslib) — https://github.com/niemasd/ViralConsensus
- **samtools** (Read subsampling from the sorted BAM and the samtools indexing/sort step inside the iVar comparison pipeline)
- **htslib** (Dependency library used by ViralConsensus for reading alignment data) — https://github.com/samtools/htslib

## Examples

```
/usr/bin/time -v viral_consensus -i subsample.bam -r reference.fas -o consensus.fas
```

## Evaluation signals

- ViralConsensus runtime scales linearly with mapped-read count across the 100, 1K, 10K, 100K, 1M depth series
- ViralConsensus peak RSS stays roughly constant with depth while iVar memory grows with depth
- ViralConsensus completes 1 million reads in under 2 seconds with peak memory under 12 MB
- benchmark_results.csv contains 100 paired measurements (5 depths x 10 replicates x 2 tools) with runtime and peak-RSS columns populated
- Each consensus output is a valid FASTA matching the reference genome length

## Limitations

- ViralConsensus memory consumption scales linearly with reference genome length, so ultra-long references (e.g. human chromosomes) may require prohibitively large memory
- Comparisons are specific to a single SARS-CoV-2 amplicon dataset; scaling conclusions may not transfer to other assay types or genomes
- ViralConsensus requires a single reference sequence being reconstructed

## Evidence

- [abstract] Performance claim: "ViralConsensus is orders of magnitude faster and more memory-efficient than existing methods."
- [readme] Runtime/memory targets: [evidence span withheld — anchor status: not_found]
- [intro] Reference-length memory scaling: "memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory"
- [methods] Consensus invocation: "viral_consensus -i example/example.trimmed.unsorted.bam -r example/NC_045512.2.fas -o consensus.fas"
