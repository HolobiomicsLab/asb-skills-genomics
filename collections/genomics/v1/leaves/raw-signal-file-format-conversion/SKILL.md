---
name: raw-signal-file-format-conversion
description: Decision support for choosing or converting a raw nanopore signal storage
  format (FAST5, POD5, SLOW5/BLOW5) when mapping raw signals with RawHash/RawHash2
  and read- or file-reading overhead dominates total run time.
when_to_use_negative:
- The question is about mapping accuracy (F1) or algorithmic seeding/quantization
  performance rather than file-reading/compression overhead — use the RawHash2 accuracy
  or throughput benchmarks instead
- The signal files are in a format RawHash does not support; RawHash reads FAST5,
  POD5, SLOW5, or BLOW5 only
- Reads are already basecalled sequences — use a sequence mapper such as minimap2
  with map-ont, not raw-signal format benchmarking
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: RawHash2
  role: Mapping tool used to run the identical indexing and mapping commands on each
    signal file format and record elapsed time; adds support for POD5 and SLOW5 formats
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash
  role: Base mechanism that maps reads from FAST5, POD5, SLOW5, or BLOW5 files; defines
    the command-line workflow being benchmarked
  repo: https://github.com/CMU-SAFARI/RawHash
- name: vbz_compression
  role: VBZ compression scheme used inside FAST5 files; identified as the source of
    the decompression overhead that POD5/SLOW5 avoid
  repo: https://github.com/nanoporetech/vbz_compression
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:1792a0397c613dbcb050163afcdd3f7162b38226ea05c9f528d29918a41f367c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  - vbz_compression
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# raw-signal-file-format-conversion

## Summary

Benchmarks and switches the raw nanopore signal container format (FAST5 vs POD5 vs SLOW5/BLOW5) used as input to raw signal mappers such as RawHash2, quantifying the elapsed-time overhead caused by signal-file compression/decompression (e.g., VBZ in FAST5).

## When to use

Decision support for choosing or converting a raw nanopore signal storage format (FAST5, POD5, SLOW5/BLOW5) when mapping raw signals with RawHash/RawHash2 and read- or file-reading overhead dominates total run time.

## When NOT to use

- The question is about mapping accuracy (F1) or algorithmic seeding/quantization performance rather than file-reading/compression overhead — use the RawHash2 accuracy or throughput benchmarks instead
- The signal files are in a format RawHash does not support; RawHash reads FAST5, POD5, SLOW5, or BLOW5 only
- Reads are already basecalled sequences — use a sequence mapper such as minimap2 with map-ont, not raw-signal format benchmarking

## Inputs

- Same nanopore read set stored in three container formats: FAST5, POD5, and SLOW5/BLOW5
- Reference genome FASTA (e.g., ref.fasta)
- RawHash2-compatible pore model file (e.g., R9.4 6-mer model under extern/kmer_models/legacy)

## Outputs

- mapping.paf per input format (RawHash2 mapping results)
- format_timing.tsv with one row per input format and recorded total elapsed time
- POD5-over-FAST5 and SLOW5-over-FAST5 speedup factors

## How to apply

Index the reference genome once with rawhash2 using the appropriate pore model (e.g., the R9.4 6-mer model under extern/kmer_models/legacy) and a fixed thread count such as -t 32, producing ref.ind. Then run the identical rawhash2 mapping command (same index, same threads, same preset) on the same read set stored as FAST5, POD5, and SLOW5/BLOW5, redirecting PAF output to mapping.paf and recording wall-clock elapsed time per format. Aggregate the three elapsed-time measurements into a single table (e.g., format_timing.tsv, one row per format) and compute the speedup of POD5 and SLOW5 relative to FAST5. The rationale is that RawHash2 supports FAST5, POD5, SLOW5, and BLOW5 inputs, and the paper attributes much of the FAST5 overhead to compression/decompression of the signal files (VBZ compression in FAST5); converting to POD5 or SLOW5 reduces total elapsed time without changing the mapping algorithm. Validate that the computed speedups reproduce the paper's relative ordering (Section 3.5, Supplementary Tables S2 and S4): both POD5 and SLOW5 map faster than FAST5.

## Related tools

- **RawHash2** (Mapping tool used to run the identical indexing and mapping commands on each signal file format and record elapsed time; adds support for POD5 and SLOW5 formats) — https://github.com/CMU-SAFARI/RawHash
- **RawHash** (Base mechanism that maps reads from FAST5, POD5, SLOW5, or BLOW5 files; defines the command-line workflow being benchmarked) — https://github.com/CMU-SAFARI/RawHash
- **vbz_compression** (VBZ compression scheme used inside FAST5 files; identified as the source of the decompression overhead that POD5/SLOW5 avoid) — https://github.com/nanoporetech/vbz_compression

## Examples

```
rawhash2 -t 32 -x r94 reads.pod5 ref.ind > mapping.paf  # repeat identically on reads.fast5 and reads.blow5, timing each run
```

## Evaluation signals

- All three mapping runs use the same index (ref.ind), same -t 32 thread count, and same preset, so the only varying factor is the input container format
- The computed POD5-over-FAST5 and SLOW5-over-FAST5 speedup factors both exceed 1 and reproduce the ordering reported in Section 3.5 / Supplementary Tables S2 and S4
- format_timing.tsv contains exactly one row per input format (FAST5, POD5, SLOW5) with a recorded wall-clock elapsed time
- PAF output is produced for each format and can be compared for mapping consistency across formats

## Limitations

- FAST5 files may be compressed with VBZ from Nanopore, and decompression of these signal files is the dominant overhead being measured; results depend on the compression state of the input FAST5
- Speedups are throughput measurements for RawHash2 mapping and do not transfer to tools that cannot read POD5 or SLOW5/BLOW5 inputs

## Evidence

- [methods] RawHash supports multiple input formats: "RawHash can be used to map reads from **FAST5, POD5, SLOW5, or BLOW5** files to a reference genome in sequence format."
- [abstract] RawHash2 adds POD5 and SLOW5 support: "support for the R10.4 flow cell version and POD5 and SLOW5 file formats"
- [methods] Indexing command grounding: "Below is an example that generates an index file `ref.ind` for the reference genome `ref.fasta` using a certain k-mer model located under `extern` and `32` threads."
- [methods] VBZ compression in FAST5: "your reads in fast5 files are compressed with the [VBZ compression](https://github.com/nanoporetech/vbz_compression) from Nanopore"
- [readme] Clone instructions for RawHash2: "git clone --recursive https://github.com/CMU-SAFARI/RawHash.git rawhash2"
