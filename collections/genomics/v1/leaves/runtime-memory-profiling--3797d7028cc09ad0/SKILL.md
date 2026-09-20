---
name: runtime-memory-profiling
description: Use when comparing runtime and peak memory usage of bioinformatics tools
  as input size scales (e.g. 100 to 1,000,000 mapped reads), or when verifying performance
  claims like 'under 2 seconds and 12 MB for 1M reads'.
when_to_use_negative:
- The question is about output accuracy/consensus correctness rather than computational
  efficiency — this skill measures runtime and peak RSS only
- The reference genome is ultra-long (e.g. a human chromosome), where ViralConsensus
  memory scales linearly with reference length and becomes the bottleneck, confounding
  depth-scaling measurements
- Inputs cannot be subsampled to controlled depths, so scaling with input size cannot
  be isolated
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: ViralConsensus
  role: 'Tool under benchmark: fast, memory-efficient viral consensus caller whose
    runtime/memory scaling is measured; invoked as viral_consensus -i <subsample.bam>
    -r <reference.fas> -o <consensus.fas>'
  repo: https://github.com/niemasd/ViralConsensus
- name: samtools
  role: Used for samtools-based read subsampling at controlled depths and for the
    sort/index steps of the iVar comparison pipeline
  repo: https://github.com/samtools/htslib
- name: htslib
  role: Core C library dependency that ViralConsensus is built on; must be installed
    in the library path before compiling and running the benchmarked tool
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:0ce33fbc43b033455e215bb9a474511a54dbb51caa2583dcc2571ebfe2c2ec38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niemasd/ViralConsensus
  tools:
  - ViralConsensus
  - htslib
  - samtools
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# runtime-memory-profiling

## Summary

Benchmark wall-clock runtime and peak memory (RSS) of computational tools across controlled input scales, using a time-and-memory profiler such as /usr/bin/time -v, to quantitatively compare efficiency claims (e.g. ViralConsensus vs the iVar pipeline on SARS-CoV-2 amplicon data).

## When to use

Use when comparing runtime and peak memory usage of bioinformatics tools as input size scales (e.g. 100 to 1,000,000 mapped reads), or when verifying performance claims like 'under 2 seconds and 12 MB for 1M reads'.

## When NOT to use

- The question is about output accuracy/consensus correctness rather than computational efficiency — this skill measures runtime and peak RSS only
- The reference genome is ultra-long (e.g. a human chromosome), where ViralConsensus memory scales linearly with reference length and becomes the bottleneck, confounding depth-scaling measurements
- Inputs cannot be subsampled to controlled depths, so scaling with input size cannot be isolated

## Inputs

- Sorted/aligned read alignment file (BAM) of a high-depth SARS-CoV-2 amplicon sequencing sample
- Reference FASTA (e.g. NC_045512.2.fas)
- Subsampled BAMs at controlled mapped-read depths (100, 1K, 10K, 100K, 1M)
- Tool configurations for each competitor (e.g. viral_consensus command line, samtools + iVar consensus pipeline)

## Outputs

- benchmark_results.csv with columns tool, replicate, mapped-read count, runtime, peak memory
- Per-replicate profiler logs of wall-clock runtime and peak RSS (e.g. from /usr/bin/time -v)
- Consensus FASTA files from each tool run, usable to confirm functional equivalence

## How to apply

Subsample the input at multiple depth levels (here: 100, 1K, 10K, 100K, and 1M mapped reads from a sorted/aligned SARS-CoV-2 amplicon BAM) with multiple replicates per depth (10) using samtools-based read subsampling to control for stochastic variation. For each replicate, run each tool under the same profiler (e.g. /usr/bin/time -v) and record wall-clock runtime and peak RSS alongside the exact mapped-read count. Aggregate all paired measurements (5 depths x 10 replicates x 2 tools = 100 rows) into a structured CSV (columns: tool, replicate, mapped-read count, runtime, peak memory) so scaling behavior can be modeled. Validate by checking scaling laws: ViralConsensus runtime should scale linearly with mapped reads while its memory stays constant (memory depends on reference genome length, ~30 kb for SARS-CoV-2, not read depth), whereas iVar's memory grows with depth.

## Related tools

- **ViralConsensus** (Tool under benchmark: fast, memory-efficient viral consensus caller whose runtime/memory scaling is measured; invoked as viral_consensus -i <subsample.bam> -r <reference.fas> -o <consensus.fas>) — https://github.com/niemasd/ViralConsensus
- **samtools** (Used for samtools-based read subsampling at controlled depths and for the sort/index steps of the iVar comparison pipeline) — https://github.com/samtools/htslib
- **htslib** (Core C library dependency that ViralConsensus is built on; must be installed in the library path before compiling and running the benchmarked tool) — https://github.com/samtools/htslib

## Examples

```
/usr/bin/time -v viral_consensus -i subsample_1000000.bam -r NC_045512.2.fas -o consensus.fas
```

## Evaluation signals

- ViralConsensus wall-clock runtime scales linearly with mapped-read count across the 100–1M read depths
- ViralConsensus peak memory stays roughly constant across depths (memory depends on reference genome length, not read count)
- ViralConsensus completes 1 million reads in under 2 seconds with peak memory under 12 MB
- benchmark_results.csv contains all 100 paired measurements (5 depths x 10 replicates x 2 tools) with non-missing runtime and peak-RSS values
- iVar pipeline runtime and memory both grow with sequencing depth, confirming a contrast rather than a flat artifact

## Limitations

- ViralConsensus memory consumption scales linearly with reference genome length, so ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large memory
- Comparative results hold for a SARS-CoV-2 (~30 kb) amplicon dataset; memory behavior may differ for other genome sizes
- Peak RSS from /usr/bin/time -v reflects the whole process tree, so pipeline stages like samtools sort contribute to the competitor's measured memory

## Evidence

- [results] ViralConsensus called a consensus sequence from an amplicon sequencing dataset with 1 million reads in under 2 seconds with peak memory usage under 12 MB: [evidence span withheld — anchor status: not_found]
- [readme] memory consumption scales linearly with reference genome length: "The caveat is that memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory"
- [methods] ViralConsensus is written in C++ and depends on htslib: "ViralConsensus is written in C++ and depends on htslib"
- [methods] viral_consensus -i example/example.trimmed.unsorted.bam -r example/NC_045512.2.fas -o consensus.fas: "viral_consensus -i example/example.trimmed.unsorted.bam -r example/NC_045512.2.fas -o consensus.fas"
- [abstract] ViralConsensus is orders of magnitude faster and more memory-efficient than existing methods.: "ViralConsensus is orders of magnitude faster and more memory-efficient than existing methods."
