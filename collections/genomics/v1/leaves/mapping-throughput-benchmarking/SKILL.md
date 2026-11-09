---
name: mapping-throughput-benchmarking
description: Use when evaluating how storage format of raw nanopore signals (FAST5
  vs POD5 vs SLOW5/BLOW5) affects RawHash2 mapping elapsed time, or when reproducing
  throughput comparisons reported in signal-mapping papers.
when_to_use_negative:
- The goal is accuracy/F1 evaluation of mapping quality rather than elapsed-time/throughput
  comparison
- Only one signal file format is available, so no cross-format timing comparison can
  be made
- The read sets differ between runs (e.g., different read counts or signal lengths),
  since timing differences would not be attributable to file format
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: RawHash2
  role: 'Mapper under benchmark: indexes the reference genome (ref.ind) and maps raw
    signals from each file format with -t 32 to measure elapsed time'
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash
  role: Predecessor mechanism whose throughput RawHash2 improves on; the same format-based
    benchmarking approach applies
  repo: https://github.com/CMU-SAFARI/RawHash
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:9328007b73bcc87aa6a0dff6324cb91d6b1b4b82b2c3e1ebcb95f8b5fd645c1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Mapping throughput benchmarking

## Summary

Benchmarks RawHash2 raw nanopore signal mapping by recording total wall-clock elapsed time of identical mapping runs (same index, same preset, same thread count) across signal file formats (FAST5, POD5, SLOW5/BLOW5), isolating file-reading/decompression overhead from mapping compute.

## When to use

Use when evaluating how storage format of raw nanopore signals (FAST5 vs POD5 vs SLOW5/BLOW5) affects RawHash2 mapping elapsed time, or when reproducing throughput comparisons reported in signal-mapping papers.

## When NOT to use

- The goal is accuracy/F1 evaluation of mapping quality rather than elapsed-time/throughput comparison
- Only one signal file format is available, so no cross-format timing comparison can be made
- The read sets differ between runs (e.g., different read counts or signal lengths), since timing differences would not be attributable to file format

## Inputs

- The same raw nanopore read set in three storage formats: FAST5, POD5, and SLOW5/BLOW5
- Reference genome FASTA (e.g., ref.fasta)
- R9.4 6-mer pore model file under extern/kmer_models/legacy

## Outputs

- ref.ind: RawHash2 index of the reference genome
- mapping.paf: mapping results in PAF format for each format run
- format_timing.tsv: table with one row per input format containing recorded total elapsed times
- Computed POD5-over-FAST5 and SLOW5-over-FAST5 speedup factors

## How to apply

First build a single reference index so all format runs share it: index the reference genome with rawhash2 using the R9.4 6-mer pore model under extern/kmer_models/legacy with 32 threads to produce ref.ind. Then run the identical mapping command (-t 32, same ref.ind, same preset) on each version of the same read set — FAST5, then POD5, then SLOW5/BLOW5 — redirecting output to mapping.paf and recording the total wall-clock elapsed time for each run. Aggregate the three elapsed-time measurements into format_timing.tsv with one row per input format, and compute throughput speedup of POD5 and SLOW5 relative to FAST5. The rationale is that RawHash2 accepts all four formats, so format is the only varying factor; large speedups indicate file-reading/decompression overhead dominates (FAST5 uses VBZ compression). Validation requires that POD5-over-FAST5 and SLOW5-over-FAST5 speedup factors reproduce the ordering reported in the paper (POD5 and SLOW5 map faster than FAST5).

## Related tools

- **RawHash2** (Mapper under benchmark: indexes the reference genome (ref.ind) and maps raw signals from each file format with -t 32 to measure elapsed time) — https://github.com/CMU-SAFARI/RawHash
- **RawHash** (Predecessor mechanism whose throughput RawHash2 improves on; the same format-based benchmarking approach applies) — https://github.com/CMU-SAFARI/RawHash

## Examples

```
./rawhash2 -t 32 ref.ind reads.fast5 > mapping.paf
```

## Evaluation signals

- All three runs use the identical index (ref.ind), identical preset, and identical thread count (-t 32), differing only in input file format
- format_timing.tsv contains exactly one row per format (FAST5, POD5, SLOW5) with a recorded total elapsed time
- Computed speedup factors show POD5 and SLOW5 mapping faster than FAST5, matching the ordering reported in Section 3.5 and Supplementary Tables S2 and S4
- Each run produces a mapping.paf output, confirming the mapping completed successfully before its timing is counted

## Limitations

- FAST5 read files may be compressed with VBZ compression, so measured elapsed time includes decompression overhead that is not part of the mapping algorithm itself
- Timing results depend on hardware and thread count (32 threads in the reference workflow); speedup factors must be recomputed on the target machine
- RawHash2 support for POD5 and SLOW5 is new; legacy RawHash may not support all formats for equivalent comparisons

## Evidence

- [methods] RawHash can be used to map reads from FAST5, POD5, SLOW5, or BLOW5 files to a reference genome in sequence format.: "RawHash can be used to map reads from **FAST5, POD5, SLOW5, or BLOW5** files to a reference genome in sequence format."
- [methods] Below is an example that generates an index file ref.ind for the reference genome ref.fasta using a certain k-mer model located under extern and 32 threads.: "Below is an example that generates an index file `ref.ind` for the reference genome `ref.fasta` using a certain k-mer model located under `extern` and `32` threads."
- [methods] your reads in fast5 files are compressed with the VBZ compression from Nanopore: "your reads in fast5 files are compressed with the [VBZ compression](https://github.com/nanoporetech/vbz_compression) from Nanopore"
- [abstract] better throughput (on average by 4.0× and up to 9.9×) than RawHash: "better throughput (on average by 4.0× and up to 9.9×) than RawHash"
- [methods] git clone --recursive https://github.com/CMU-SAFARI/RawHash.git rawhash2: "git clone --recursive https://github.com/CMU-SAFARI/RawHash.git rawhash2"
