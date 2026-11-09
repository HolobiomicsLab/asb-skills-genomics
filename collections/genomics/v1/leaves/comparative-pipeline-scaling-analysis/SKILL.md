---
name: comparative-pipeline-scaling-analysis
description: Use when comparing runtime and peak-RSS scaling of two bioinformatics
  tools across controlled read-depth levels (e.g. 100/1K/10K/100K/1M mapped reads)
  on the same BAM input.
when_to_use_negative:
- The goal is consensus-sequence accuracy rather than performance scaling — accuracy
  requires a gold-standard comparison, not subsampled timing replicates
- Only a single depth level is available, so no depth-scaling relationship can be
  measured
- The pipelines under comparison cannot run on the same subsampled BAM inputs (e.g.
  one requires raw FASTQ mapper output), making the paired design invalid
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: ViralConsensus
  role: Consensus caller under test; runs directly on the subsampled BAM and is profiled
    for runtime and peak RSS
  repo: https://github.com/niemasd/ViralConsensus
- name: samtools
  role: Provides read subsampling, sorting, and indexing of the subsampled BAMs, and
    forms the iVar pipeline's prerequisite steps
  repo: https://github.com/samtools/htslib
- name: htslib
  role: Underlying C library for BAM/alignment I/O that ViralConsensus depends on;
    installation/compilation prerequisite
  repo: https://github.com/samtools/htslib
- name: iVar
  role: Baseline consensus pipeline (samtools sort/index + iVar consensus) against
    which ViralConsensus is benchmarked
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:57e05cc7c540203e1d6af2459f542ff8b4ab6c5889fbe183db87c84b3719891e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niemasd/ViralConsensus
  tools:
  - ViralConsensus
  - htslib
  - iVar
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# comparative-pipeline-scaling-analysis

## Summary

Benchmarks how two consensus-calling pipelines (ViralConsensus vs the iVar pipeline) scale in wall-clock runtime and peak memory as sequencing depth increases on SARS-CoV-2 amplicon data, using samtools-based read subsampling and /usr/bin/time -v profiling.

## When to use

Use when comparing runtime and peak-RSS scaling of two bioinformatics tools across controlled read-depth levels (e.g. 100/1K/10K/100K/1M mapped reads) on the same BAM input.

## When NOT to use

- The goal is consensus-sequence accuracy rather than performance scaling — accuracy requires a gold-standard comparison, not subsampled timing replicates
- Only a single depth level is available, so no depth-scaling relationship can be measured
- The pipelines under comparison cannot run on the same subsampled BAM inputs (e.g. one requires raw FASTQ mapper output), making the paired design invalid

## Inputs

- Coordinate-sorted, indexed BAM of SARS-CoV-2 amplicon sequencing reads (highest-depth sample)
- Viral reference FASTA (e.g. NC_045512.2.fas)
- ViralConsensus, iVar, and samtools/htslib installations

## Outputs

- benchmark_results.csv with columns tool, replicate, mapped-read count, runtime, and peak memory
- Runtime-vs-depth and peak-memory-vs-depth scaling curves for both tools
- Validation report of the 1M-read thresholds (ViralConsensus < 2 s, < 12 MB peak RSS)

## How to apply

Subsample the sorted/aligned reads of the highest-depth sample at five depths (100, 1K, 10K, 100K, 1M mapped reads) with 10 replicates per depth using samtools-based read subsampling. For each replicate, run ViralConsensus (viral_consensus -i <subsample.bam> -r <reference.fas> -o <consensus.fas>) under a time-and-memory profiler such as /usr/bin/time -v, recording wall-clock runtime and peak RSS. In parallel, run the iVar consensus pipeline (samtools indexing/sort + iVar consensus calling) under the identical profiler and environment. Aggregate the 100 paired measurements (5 depths x 10 replicates x 2 tools) into a results CSV with columns for tool, replicate, mapped-read count, runtime, and peak memory. The rationale is that ViralConsensus is written in C++ on htslib with a single-pass, constant-memory design, so its runtime should scale linearly with mapped reads while memory stays constant, whereas the samtools/iVar chain's memory grows with depth.

## Related tools

- **ViralConsensus** (Consensus caller under test; runs directly on the subsampled BAM and is profiled for runtime and peak RSS) — https://github.com/niemasd/ViralConsensus
- **samtools** (Provides read subsampling, sorting, and indexing of the subsampled BAMs, and forms the iVar pipeline's prerequisite steps) — https://github.com/samtools/htslib
- **htslib** (Underlying C library for BAM/alignment I/O that ViralConsensus depends on; installation/compilation prerequisite) — https://github.com/samtools/htslib
- **iVar** (Baseline consensus pipeline (samtools sort/index + iVar consensus) against which ViralConsensus is benchmarked)

## Examples

```
/usr/bin/time -v viral_consensus -i subsample.bam -r reference.fas -o consensus.fas
```

## Evaluation signals

- ViralConsensus runtime scales linearly with mapped-read count across the five depths
- ViralConsensus peak RSS stays constant (≈ under 12 MB) as depth grows from 100 to 1M reads
- ViralConsensus completes 1M reads in under 2 seconds (finding: called consensus from 1 million reads in under 2 s with peak memory under 12 MB)
- All 100 paired measurements (5 depths x 10 replicates x 2 tools) are present in benchmark_results.csv with populated runtime and peak-memory columns
- iVar pipeline's peak memory grows with depth, confirming the expected contrast between the two tools

## Limitations

- Memory consumption of ViralConsensus scales linearly with reference genome length, so ultra-long references (e.g. human chromosomes) may require prohibitively large memory
- The comparison reflects a SARS-CoV-2 amplicon dataset (~30 kb genome); results do not directly transfer to much larger reference genomes
- The web app (WebAssembly port) works well only for reasonably small datasets — command-line tooling is required for large-depth benchmarking

## Evidence

- [abstract] ViralConsensus is orders of magnitude faster and more memory-efficient than existing methods.: "ViralConsensus is orders of magnitude faster and more memory-efficient than existing methods."
- [readme] memory consumption scales linearly with reference genome length: "memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory"
- [methods] ViralConsensus is written in C++ and depends on htslib: "ViralConsensus is written in C++ and depends on htslib"
- [methods] consensus calling example command: "viral_consensus -i example/example.trimmed.unsorted.bam -r example/NC_045512.2.fas -o consensus.fas"
- [intro] pipe data directly from a read mapper via standard input: "ViralConsensus can pipe data directly from a read mapper via standard input and performs viral consensus calling on-the-fly, making it an ideal tool for viral sequencing pipelines."
