---
name: benchmark-resource-usage-profiling
description: Use when benchmarking a compressor (NanoSpring or similar) across dataset
  or parameter variants and you must report bits/base together with compression time
  and peak memory per run, e.g., comparing Guppy fast/hac/sup basecalled reads.
when_to_use_negative:
- When only compression ratio matters and resource usage is not part of the reported
  comparison — a single compression run suffices
- When the input is not FASTQ read sequences (NanoSpring ignores read identifiers
  and quality values, so benchmarking it on other file types is out of scope)
- When comparing against reference-based compressors requiring a reference genome
  — NanoSpring is reference-free and the comparison setup differs
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3275
tools:
- name: NanoSpring
  role: Compressor being benchmarked; run in compress mode with default 20 threads
    on each basecaller-mode FASTQ, timed and memory-profiled per run
  repo: https://github.com/qm2/NanoSpring
- name: Guppy
  role: ONT basecaller used to produce the fast/hac/sup FASTQ variants that define
    the benchmark conditions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:dd276da214d9ebf070bd3f121d1e818367e662fa2cf08f6484174d91b5c616dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - Guppy
  - NanoSpring
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3275
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# benchmark-resource-usage-profiling

## Summary

Profiling discipline for measuring compression runtime and peak memory usage of a sequence compressor (e.g., NanoSpring on nanopore FASTQ data) alongside compression ratio, so algorithmic gains can be judged against practical resource costs as in the paper's Table 5 basecaller-mode comparison.

## When to use

Use when benchmarking a compressor (NanoSpring or similar) across dataset or parameter variants and you must report bits/base together with compression time and peak memory per run, e.g., comparing Guppy fast/hac/sup basecalled reads.

## When NOT to use

- When only compression ratio matters and resource usage is not part of the reported comparison — a single compression run suffices
- When the input is not FASTQ read sequences (NanoSpring ignores read identifiers and quality values, so benchmarking it on other file types is out of scope)
- When comparing against reference-based compressors requiring a reference genome — NanoSpring is reference-free and the comparison setup differs

## Inputs

- FASTQ files of the same dataset under each condition (e.g., S. aureus basecalled with Guppy 6.1.2 fast, hac, and sup modes)
- NanoSpring compressor executable (built from source or installed via bioconda)
- Machine with fixed hardware/OS for all timed runs

## Outputs

- Per-run measurements: compressed size in bits/base, compression time, peak memory usage
- Summary table (e.g., table5_s_aureus.csv) with one row per condition and columns for mean error rate, bits/base, compression time, and peak memory
- .NanoSpring compressed archives per condition

## How to apply

Run the compressor under identical conditions for every variant — for NanoSpring use the documented invocation ('NanoSpring -c -i input.fastq -o output.NanoSpring') with the default 20 threads and default parameters (k=23, 60 hash functions, overlap sketch threshold 6) so runs differ only in the factor being compared. Wrap each run with time and peak-memory measurement (e.g., /usr/bin/time -v) and record wall-clock compression time and maximum resident set size alongside the compressed size converted to bits/base (compressed bytes * 8 divided by total base count of the FASTQ). For NanoSpring specifically, note that decompression speed scales with threads (>4x faster decompression with 20 threads) and decompression peak memory can be tuned via the --decompression-memory option (default 5 GB), so profile decompression separately if resource usage is a reported metric. Assemble per-variant rows into a table (e.g., table5_s_aureus.csv with columns for mean error rate, bits/base, compression time, peak memory) and sanity-check values against published references (0.69 bits/base fast, 0.48 hac, 0.41 sup on S. aureus).

## Related tools

- **NanoSpring** (Compressor being benchmarked; run in compress mode with default 20 threads on each basecaller-mode FASTQ, timed and memory-profiled per run) — https://github.com/qm2/NanoSpring
- **Guppy** (ONT basecaller used to produce the fast/hac/sup FASTQ variants that define the benchmark conditions)

## Examples

```
/usr/bin/time -v NanoSpring -c -i s_aureus_sup.fastq -o s_aureus_sup.NanoSpring -t 20
```

## Evaluation signals

- Reproduced bits/base values fall close to the paper's reported 0.69 (fast), 0.48 (hac), and 0.41 (sup) for the S. aureus dataset
- Each condition uses identical NanoSpring parameters and thread count (default -t 20), so differences in time/memory are attributable to the variant, not settings
- Compressed archives decompress losslessly (cmp file.reads <(cat file.fastq | sed -n '2~4p') passes), confirming the benchmarked runs were valid compressions
- bits/base is computed consistently as compressed bytes * 8 / total bases of the input FASTQ across all conditions
- Compression time and peak memory are recorded for every run and match the expected ballpark for the machine used (NanoSpring is competitive in resource usage with CoLoRd per the paper)

## Limitations

- NanoSpring compresses only read sequences, not read identifiers or quality values, so bits/base benchmarks are not directly comparable to whole-FASTQ compressors
- Peak memory and runtime depend on hardware; values are only comparable across conditions run on the same machine and thread count
- NanoSpring's decompression --decompression-memory setting is only approximate and may have no effect at very low settings or with large thread counts
- On older CPU architectures the -msse4.1/-mavx2 compile flags may need adjustment, which can affect measured performance

## Evidence

- [abstract] significantly faster at decompression when using multiple threads (> 4[Formula: see text] faster decompression with 20 threads): "significantly faster at decompression when using multiple threads (> 4[Formula: see text] faster decompression with 20 threads)"
- [methods] ./NanoSpring -c -i file.fastq -o file.NanoSpring: "./NanoSpring -c -i file.fastq -o file.NanoSpring"
- [readme] attempt to set peak memory usage for decompression in GB (default 5 GB) by using disk-based sort: "--decompression-memory arg (=5) attempt to set peak memory usage for 
                                  decompression in GB (default 5 GB) by using 
                                  disk-based sort"
- [discussion] NanoSpring, which focuses only on the base sequences in the FASTQ file: "NanoSpring, which focuses only on the base sequences in the FASTQ file"
