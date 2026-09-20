---
name: peak-memory-measurement
description: Use when comparing resource usage (runtime and peak memory) of sequence-compression
  tools on the same FASTQ inputs, e.g. benchmarking NanoSpring vs CoLoRd decompression
  with 20 threads.
when_to_use_negative:
- When the goal is compression ratio (bits per base) rather than runtime/memory resource
  usage
- When NanoSpring's read identifier or quality-score compression matters — NanoSpring
  only compresses base sequences, so resource comparisons against tools compressing
  full FASTQ are not apples-to-apples
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: NanoSpring
  role: Tool under benchmark; compress/decompress FASTQ read sequences with 20 threads
    while recording time and peak memory
  repo: https://github.com/qm2/NanoSpring
- name: CoLoRd
  role: State-of-the-art comparison compressor/decompressor for runtime and memory
    benchmarks
- name: gzip
  role: General-purpose compressor baseline for resource-usage tables
- name: ENANO
  role: Nanopore-specific comparison compressor for resource-usage tables
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:35da9c347302300c05fd25990b8bb65294bd906920842e37f44113103c24bbd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - CoLoRd
  - ENANO
  - NanoSpring
  - gzip
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# peak-memory-measurement

## Summary

Benchmarking skill for recording wall-clock time and peak memory usage of compression/decompression tools (NanoSpring, CoLoRd, gzip, ENANO) run with 20 threads on nanopore FASTQ datasets, to reproduce Tables 3-4 of the NanoSpring paper.

## When to use

Use when comparing resource usage (runtime and peak memory) of sequence-compression tools on the same FASTQ inputs, e.g. benchmarking NanoSpring vs CoLoRd decompression with 20 threads.

## When NOT to use

- When the goal is compression ratio (bits per base) rather than runtime/memory resource usage
- When NanoSpring's read identifier or quality-score compression matters — NanoSpring only compresses base sequences, so resource comparisons against tools compressing full FASTQ are not apples-to-apples

## Inputs

- Nanopore sequencing FASTQ datasets listed in Table 1 of the paper (e.g. 26x human hs2)
- Installed compressors: NanoSpring, gzip, ENANO, CoLoRd
- Compressed archives produced by each tool (.NanoSpring, gzip, etc.) for the decompression benchmark

## Outputs

- Wall-clock compression and decompression times per tool per dataset
- Peak memory usage (GB) per tool per dataset
- Tabulated benchmark reproducing Tables 3-4 (with hs2 ~11 min, ~5.6 GB for NanoSpring decompression)

## How to apply

Run each compressor and decompressor with a fixed thread count (20 threads, e.g. './NanoSpring -c -i file.fastq -o file.NanoSpring -t 20' and './NanoSpring -d -i file.NanoSpring -o file.reads -t 20') on each Table 1 nanopore FASTQ dataset, recording wall-clock time and peak memory for every run. Keep parameters identical across tools so timing/memory comparisons are attributable to the algorithm, not configuration. Note that NanoSpring's peak decompression memory is tunable via the --decompression-memory option (default 5 GB, approximate and using disk-based sorting to write reads in correct order). Verify losslessness of each run (e.g. 'cmp file.reads <(cat file.fastq | sed -n '2~4p')') so resource measurements correspond to valid compressed archives. Tabulate compression/decompression times and peak memory for all four tools across all datasets, expecting e.g. ~11 minutes and ~5.6 GB peak memory for NanoSpring decompressing the 26x human dataset (hs2), and >4x faster NanoSpring vs CoLoRd decompression with 20 threads.

## Related tools

- **NanoSpring** (Tool under benchmark; compress/decompress FASTQ read sequences with 20 threads while recording time and peak memory) — https://github.com/qm2/NanoSpring
- **CoLoRd** (State-of-the-art comparison compressor/decompressor for runtime and memory benchmarks)
- **gzip** (General-purpose compressor baseline for resource-usage tables)
- **ENANO** (Nanopore-specific comparison compressor for resource-usage tables)

## Examples

```
./NanoSpring -d -i file.NanoSpring -o file.reads -t 20; /usr/bin/time -v ./NanoSpring -d -i hs2.fastq -o hs2.reads -t 20
```

## Evaluation signals

- Every decompressed output passes the lossless check: 'cmp file.reads <(cat file.fastq | sed -n '2~4p')' returns no difference
- Timing and peak-memory values are recorded for all four tools across all Table 1 datasets with the same 20-thread setting
- NanoSpring decompression of hs2 is around 11 minutes and 5.6 GB peak memory
- NanoSpring decompression is >4x faster than CoLoRd with 20 threads
- NanoSpring's observed peak decompression memory is consistent with the --decompression-memory setting (default 5 GB, approximate)

## Limitations

- The --decompression-memory setting is only approximate and might have no effect at very low settings, with large thread counts, or when another decompressor stage is the biggest memory contributor; very low values may slightly reduce speed
- NanoSpring compresses only read sequences, not read identifiers or quality values, so end-to-end resource comparisons against full-FASTQ compressors are not directly equivalent
- Older architectures may require removing -msse4.1/-mavx2 flags from the build, which could affect measured performance

## Evidence

- [abstract] significantly faster at decompression when using multiple threads (> 4[Formula: see text] faster decompression with 20 threads): "significantly faster at decompression when using multiple threads (> 4[Formula: see text] faster decompression with 20 threads)"
- [readme] number of threads (default 20): "  -t [ --num-threads ] arg (=20) number of threads (default 20)"
- [readme] --decompression-memory arg (=5) attempt to set peak memory usage for decompression in GB: "--decompression-memory arg (=5) attempt to set peak memory usage for decompression in GB (default 5 GB) by using disk-based sort for writing reads in the correct order."
- [results] NanoSpring is faster and more memory efficient than CoLoRd during decompression, requiring around 11 minutes and 5.6 GB peak memory for a 26x human dataset (hs2).: [evidence span withheld — anchor status: not_found]
