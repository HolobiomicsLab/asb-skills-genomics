---
name: compression-stream-size-quantification
description: Use when you need a per-stream bits-per-base breakdown (consensus, error,
  lone-read) of NanoSpring-compressed nanopore FASTQ data, e.g. to reproduce the Figure
  3 stream-contribution analysis across Table 1 datasets.
when_to_use_negative:
- You only need total compression-ratio benchmarking against gzip, ENANO, or CoLoRd
  rather than a per-stream decomposition.
- The compressed output comes from a different compressor and is not a .NanoSpring
  tar archive with separable streams.
- You need size accounting for quality scores or read identifiers — NanoSpring compresses
  only the base sequences in the FASTQ file.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: NanoSpring
  role: Reference-free nanopore read compressor whose .NanoSpring output (a tar archive
    of consensus, error, and lone-read streams) is the object being decomposed and
    measured
  repo: https://github.com/qm2/NanoSpring
- name: gzip
  role: General-purpose compressor referenced as the baseline whose 3-6x worse bits-per-base
    contextualizes the stream breakdown
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:e9c57748782f14ffe77e3dd197b082d8064e618a344e8e60b9aa6a2d13a6a908
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - NanoSpring
  - gzip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# compression-stream-size-quantification

## Summary

Decomposes the total compressed size of NanoSpring output into its per-stream contributions (consensus sequence, error streams, lone-read streams) measured in bits-per-base, reproducing the Figure 3 analysis across the Table 1 nanopore FASTQ datasets.

## When to use

Use when you need a per-stream bits-per-base breakdown (consensus, error, lone-read) of NanoSpring-compressed nanopore FASTQ data, e.g. to reproduce the Figure 3 stream-contribution analysis across Table 1 datasets.

## When NOT to use

- You only need total compression-ratio benchmarking against gzip, ENANO, or CoLoRd rather than a per-stream decomposition.
- The compressed output comes from a different compressor and is not a .NanoSpring tar archive with separable streams.
- You need size accounting for quality scores or read identifiers — NanoSpring compresses only the base sequences in the FASTQ file.

## Inputs

- Table 1 nanopore FASTQ datasets (snail, banana, hs1, hs4), optionally gzipped
- NanoSpring .NanoSpring archive (tar of compressed consensus, error, and lone-read streams)

## Outputs

- stream_bits_per_base.csv with bits-per-base for the consensus, error, and lone-read streams per dataset
- Figure 3-style per-stream contribution table across all analyzed datasets

## How to apply

Compress each Table 1 FASTQ dataset (snail, banana, hs1, hs4) with './NanoSpring -c -i <file.fastq> -o <file.NanoSpring>' using the default 20 threads. The resulting .NanoSpring file is a tar archive of the different compressed streams, so untar it to obtain the individual consensus, error, and lone-read stream files. Measure each stream's byte size with standard file-size utilities and convert to bits-per-base by multiplying bytes by 8 and dividing by the total number of bases in the corresponding FASTQ. Tabulate consensus, error, and lone-read bits-per-base for every dataset into a single stream_bits_per_base.csv. Rationale: NanoSpring's approximate-assembly approach splits compressed size into a shared consensus, per-read error corrections, and lone (unaligned) reads, so per-stream sizes reveal where the bits go; optionally verify losslessness by comparing decompressed reads with the original FASTQ sequences.

## Related tools

- **NanoSpring** (Reference-free nanopore read compressor whose .NanoSpring output (a tar archive of consensus, error, and lone-read streams) is the object being decomposed and measured) — https://github.com/qm2/NanoSpring
- **gzip** (General-purpose compressor referenced as the baseline whose 3-6x worse bits-per-base contextualizes the stream breakdown)

## Examples

```
./NanoSpring -c -i hs4.fastq -o hs4.NanoSpring && tar -xf hs4.NanoSpring -C hs4_streams
```

## Evaluation signals

- Per-stream bits-per-base values should sum to NanoSpring's total, which is 0.35-0.65 bits per base for recently basecalled high quality nanopore datasets.
- Error streams should account for close to 0.5 bits/base for most datasets while the consensus sequence contributes less.
- Lone-read stream contribution should be large for snail, banana and hs1 but negligible for hs4.
- Lossless round-trip check passes: cmp file.reads <(cat file.fastq | sed -n '2~4p') reports no differences after decompression.
- Summing the individual stream byte sizes from the untarred archive reproduces the .NanoSpring archive's total size (minus tar overhead).

## Limitations

- NanoSpring compresses only the read sequences; quality values and read identifiers are ignored, so the breakdown covers base-sequence streams only.
- Bits-per-base figures apply to recently basecalled high-quality nanopore datasets; noisier basecalls shift the share between error and lone-read streams.
- The decomposition assumes default parameters (20 threads, kmer 23, 60 minhash functions, overlap sketch threshold 6); changing options like -k, -n, or --overlap-sketch-thr alters stream sizes.
- The compressed file layout (tar of streams) is a convention of the current release (e.g. v0.2 performance improvements); stream names or structure may change across versions.

## Evidence

- [readme] Note that the compressed files are tar archives consisting of the different compressed streams: "Note that the compressed files are tar archives consisting of the different compressed streams, although we recommend using the ```.NanoSpring``` extension"
- [abstract] For recently basecalled high quality nanopore datasets, NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base: "For recently basecalled high quality nanopore datasets, NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base"
- [readme] Compare decompressed file to reads in original file to verify compression was lossless.: "Compare decompressed file to reads in original file to verify compression was lossless."
- [readme] For compressing file.fastq using default 20 threads.: "For compressing file.fastq using default 20 threads.
```
./NanoSpring -c -i file.fastq -o file.NanoSpring"
