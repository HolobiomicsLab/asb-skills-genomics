---
name: wallclock-runtime-profiling
description: Use when comparing runtime or memory footprints of compression tools
  on nanopore FASTQ datasets, or when reproducing the paper's Tables 3-4 timing/memory
  results with 20 threads.
when_to_use_negative:
- The goal is compression ratio (bits per base) rather than runtime/memory resource
  usage
- Input data is not FASTQ nanopore reads, e.g. quality scores or read identifiers
  that NanoSpring does not compress
- The comparison targets reference-based compressors like RENANO rather than the reference-free
  tools benchmarked here
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NanoSpring
  role: Benchmark subject and runtime reference; compresses/decompresses nanopore
    FASTQ read sequences with configurable thread count (-t) and decompression memory
    target
  repo: https://github.com/qm2/NanoSpring
- name: CoLoRd
  role: State-of-the-art comparison compressor for decompression time and peak memory
    baselines
- name: gzip
  role: General-purpose compression baseline for runtime and memory comparison
- name: ENANO
  role: Specialized nanopore compressor baseline for timing/memory comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:9c4b3b39058b340af18e81264349b060a294fd4d3975d6ac39ee58c52c5c745e
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# wallclock-runtime-profiling

## Summary

Benchmark wall-clock time and peak memory usage of FASTQ compressors/decompressors (NanoSpring, CoLoRd, gzip, ENANO) on the paper's nanopore datasets at a fixed thread count (20 threads) to reproduce Tables 3-4 and expose resource-usage trade-offs.

## When to use

Use when comparing runtime or memory footprints of compression tools on nanopore FASTQ datasets, or when reproducing the paper's Tables 3-4 timing/memory results with 20 threads.

## When NOT to use

- The goal is compression ratio (bits per base) rather than runtime/memory resource usage
- Input data is not FASTQ nanopore reads, e.g. quality scores or read identifiers that NanoSpring does not compress
- The comparison targets reference-based compressors like RENANO rather than the reference-free tools benchmarked here

## Inputs

- Nanopore sequencing FASTQ datasets from Table 1 of the paper (e.g. hs2, 26x human dataset)
- Installed compression tools: NanoSpring, gzip, ENANO, CoLoRd

## Outputs

- Table of wall-clock compression times per dataset and tool (Table 3 reproduction)
- Table of decompression times and peak memory usage per dataset and tool (Table 4 reproduction)
- Lossless verification results for each decompressed archive

## How to apply

Obtain the nanopore FASTQ datasets listed in Table 1 of the paper (including the 26x human dataset hs2). Install NanoSpring via bioconda or build from source with cmake, and set up Gzip, ENANO, and CoLoRd as comparison compressors. For each dataset-tool pair, run compression with 20 threads (e.g. './NanoSpring -c -i file.fastq -o file.NanoSpring -t 20') and decompression with 20 threads ('./NanoSpring -d -i file.NanoSpring -o file.reads -t 20'), recording wall-clock time and peak memory for each step. Verify each round trip is lossless with 'cmp file.reads <(cat file.fastq | sed -n '2~4p')' before trusting the timing numbers, since a failed decompression invalidates the measurement. Tabulate compression time, decompression time, and peak memory across all four tools to reproduce Tables 3 and 4 and evaluate claims such as NanoSpring being >4x faster than CoLoRd at decompression with 20 threads.

## Related tools

- **NanoSpring** (Benchmark subject and runtime reference; compresses/decompresses nanopore FASTQ read sequences with configurable thread count (-t) and decompression memory target) — https://github.com/qm2/NanoSpring
- **CoLoRd** (State-of-the-art comparison compressor for decompression time and peak memory baselines)
- **gzip** (General-purpose compression baseline for runtime and memory comparison)
- **ENANO** (Specialized nanopore compressor baseline for timing/memory comparison)

## Examples

```
./NanoSpring -d -i file.NanoSpring -o file.reads -t 20; /usr/bin/time -v ./NanoSpring -d -i file.NanoSpring -o file.reads -t 20
```

## Evaluation signals

- Every decompressed output passes the lossless check: 'cmp file.reads <(cat file.fastq | sed -n '2~4p')' reports no differences
- Timing and peak memory values are recorded for all four tools (NanoSpring, gzip, ENANO, CoLoRd) across all Table 1 datasets
- Decompression results are consistent with the finding that NanoSpring needs ~11 minutes and ~5.6 GB peak memory for the 26x human dataset (hs2) with 20 threads
- Relative behavior matches the paper's claim of >4x faster decompression for NanoSpring vs CoLoRd with 20 threads
- Thread count (-t 20) and other defaults (k=23, 60 hash functions, overlap-sketch-threshold 6) are held constant across runs so comparisons are fair

## Limitations

- NanoSpring's --decompression-memory flag only approximately controls peak memory and may have no effect at very low settings or with many threads when another decompression stage dominates
- Very low --decompression-memory values might slightly reduce decompression speed
- NanoSpring compresses only base sequences and ignores read identifiers and quality values, so runtime comparisons exclude those streams
- Timing/memory results depend on hardware and thread count; the paper's numbers were obtained with 20 threads

## Evidence

- [abstract] NanoSpring is faster and more memory efficient than CoLoRd during decompression: "significantly faster at decompression when using multiple threads (> 4[Formula: see text] faster decompression with 20 threads)"
- [readme] Lossless verification command from README: "cmp file.reads <(cat file.fastq | sed -n '2~4p')"
- [readme] Decompression memory control is approximate: "attempt to set peak memory usage for decompression in GB (default 5 GB) by using disk-based sort for writing reads in the correct order. This is only approximate"
- [discussion] NanoSpring compresses only read sequences: "NanoSpring, which focuses only on the base sequences in the FASTQ file"
- [methods] Compression/decompression commands with thread count: "./NanoSpring -c -i file.fastq -o file.NanoSpring"
