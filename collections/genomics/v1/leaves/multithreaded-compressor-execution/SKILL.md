---
name: multithreaded-compressor-execution
description: Use when comparing runtime and memory footprint of nanopore read compressors/decompressors
  on FASTQ benchmarks, especially multi-threaded (>4 thread) decompression speed comparisons
  like NanoSpring vs CoLoRd.
when_to_use_negative:
- When the goal is compression ratio (bits per base) rather than runtime/memory benchmarking
  — NanoSpring only compresses base sequences, not identifiers or quality scores
- When reads are short, low-error Illumina reads — specialized tools like SPRING and
  PgRC assume short reads and this nanopore workflow does not apply
- When only a single-thread or <=4-thread regime is of interest — the paper's key
  speed advantage (e.g. >4x faster decompression) is measured with 20 threads
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0080
tools:
- name: NanoSpring
  role: Primary compressor/decompressor benchmarked; invoked with -c/-d and -t 20
    for timing and memory measurements
  repo: https://github.com/qm2/NanoSpring
- name: CoLoRd
  role: State-of-the-art comparison compressor for compression ratio and decompression
    resource usage
- name: gzip
  role: General-purpose baseline compressor included in the timing/memory tables
- name: ENANO
  role: Comparison compressor focused on quality score compression, included in the
    benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:cc01bd54e9485fe27e8ec94ceb3142f6ac8a724bf0ddcf4fcd2367723bdbb767
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - CoLoRd
  - ENANO
  - NanoSpring
  - gzip
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# multithreaded-compressor-execution

## Summary

Benchmark NanoSpring and competing FASTQ compressors (gzip, ENANO, CoLoRd) by running compression and decompression with 20 threads, recording wall-clock time and peak memory to reproduce the paper's Tables 3-4 resource-usage comparison.

## When to use

Use when comparing runtime and memory footprint of nanopore read compressors/decompressors on FASTQ benchmarks, especially multi-threaded (>4 thread) decompression speed comparisons like NanoSpring vs CoLoRd.

## When NOT to use

- When the goal is compression ratio (bits per base) rather than runtime/memory benchmarking — NanoSpring only compresses base sequences, not identifiers or quality scores
- When reads are short, low-error Illumina reads — specialized tools like SPRING and PgRC assume short reads and this nanopore workflow does not apply
- When only a single-thread or <=4-thread regime is of interest — the paper's key speed advantage (e.g. >4x faster decompression) is measured with 20 threads

## Inputs

- Nanopore sequencing FASTQ files (optionally gzipped) from the datasets in Table 1 of the paper
- Installed NanoSpring executable (conda or built in the build directory)
- Comparison compressors: gzip, ENANO, CoLoRd

## Outputs

- Wall-clock compression time and peak memory per tool per dataset (Table 3 equivalent)
- Wall-clock decompression time and peak memory per tool per dataset (Table 4 equivalent)
- Lossless-verification result (cmp of decompressed sequences vs original FASTQ reads) per compression run

## How to apply

Install NanoSpring via bioconda or build it from source with cmake in the build directory (removing -msse4.1/-mavx2 flags on older architectures), and set up gzip, ENANO, and CoLoRd as comparators. For each Table 1 nanopore FASTQ dataset, compress with 20 threads ('./NanoSpring -c -i file.fastq -o file.NanoSpring -t 20') and decompress with 20 threads ('./NanoSpring -d -i file.NanoSpring -o file.reads -t 20'), recording wall-clock time and peak memory (e.g. /usr/bin/time -v) for every tool-dataset pair. Verify losslessness by comparing decompressed sequences to the original reads with 'cmp file.reads <(cat file.fastq | sed -n '2~4p')'. Tabulate times and peak memory into compression (Table 3) and decompression (Table 4) results; expect NanoSpring to decompress a 26x human dataset (hs2) in about 11 minutes at 5.6 GB peak memory, faster and leaner than CoLoRd.

## Related tools

- **NanoSpring** (Primary compressor/decompressor benchmarked; invoked with -c/-d and -t 20 for timing and memory measurements) — https://github.com/qm2/NanoSpring
- **CoLoRd** (State-of-the-art comparison compressor for compression ratio and decompression resource usage)
- **gzip** (General-purpose baseline compressor included in the timing/memory tables)
- **ENANO** (Comparison compressor focused on quality score compression, included in the benchmark)

## Examples

```
./NanoSpring -d -i file.NanoSpring -o file.reads -t 20
```

## Evaluation signals

- Every decompressed output passes the lossless check: 'cmp file.reads <(cat file.fastq | sed -n '2~4p')' reports no differences
- Timing and peak-memory values are recorded for all four tools across all Table 1 datasets (no missing cells)
- NanoSpring hs2 (26x human) decompression is around 11 minutes with ~5.6 GB peak memory at 20 threads
- With 20 threads, NanoSpring decompression is at least ~4x faster than CoLoRd
- Thread-count parameter is honored: runs use -t 20 (the NanoSpring default), so results should differ from single-thread baselines

## Limitations

- The --decompression-memory option (default 5 GB) only approximately caps peak memory and may have no effect with many threads or very low settings; very low values can slightly reduce speed
- Building from source on older architectures may require removing -msse4.1 and -mavx2 compile flags or building minimap2 with 'make sse2only=1'
- NanoSpring does not compress read identifiers or quality values, so comparisons with tools that compress the full FASTQ are not apples-to-apples
- Conda-installed NanoSpring must be invoked as 'NanoSpring' rather than './NanoSpring'

## Evidence

- [abstract] significantly faster at decompression when using multiple threads (> 4[Formula: see text] faster decompression with 20 threads): "significantly faster at decompression when using multiple threads (> 4[Formula: see text] faster decompression with 20 threads)"
- [other] For each dataset and each tool, compress the FASTQ file using 20 threads (NanoSpring invoked as './NanoSpring -c -i file.fastq -o file.NanoSpring -t 20'), recording wall-clock time and peak memory usage.: [evidence span withheld — anchor status: not_found]
- [readme] Compare decompressed file to reads in original file to verify compression was lossless.: "Compare decompressed file to reads in original file to verify compression was lossless."
- [readme] attempt to set peak memory usage for decompression in GB (default 5 GB) by using disk-based sort for writing reads in the correct order: "--decompression-memory arg (=5) attempt to set peak memory usage for decompression in GB"
