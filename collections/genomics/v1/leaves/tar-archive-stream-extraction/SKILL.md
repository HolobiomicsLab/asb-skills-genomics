---
name: tar-archive-stream-extraction
description: Use when a .NanoSpring (or similar tar-structured) compressed output
  must be decomposed into its constituent compressed streams to attribute total compressed
  size to individual algorithmic components.
when_to_use_negative:
- Input is a plain FASTQ or a general-purpose compressed file (gzip/lzma), not a .NanoSpring
  tar archive of streams
- The goal is total compressed size or compression ratio rather than per-stream attribution
- NanoSpring output was produced with modified stream structure or a different compressor
  whose archive layout differs
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3697
tools:
- name: NanoSpring
  role: Produces the .NanoSpring tar archive whose individual compressed streams (consensus,
    error, lone-read) are extracted and measured; run with default 20 threads
  repo: https://github.com/qm2/NanoSpring
- name: gzip
  role: Baseline general-purpose compressor whose output is compared against NanoSpring's
    per-stream sizes (3-6x worse compression)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:bb08b59cc662ff3024a53deb7675194ea7300ab5bb12e4318e9adb6034883c52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - NanoSpring
  - gzip
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# tar-archive-stream-extraction

## Summary

Untar a NanoSpring .NanoSpring compressed archive — which is a tar of separate compressed streams — and measure the byte size of each stream (consensus, error, lone-read) so per-stream bits-per-base contributions to total compressed size can be computed across FASTQ datasets.

## When to use

Use when a .NanoSpring (or similar tar-structured) compressed output must be decomposed into its constituent compressed streams to attribute total compressed size to individual algorithmic components.

## When NOT to use

- Input is a plain FASTQ or a general-purpose compressed file (gzip/lzma), not a .NanoSpring tar archive of streams
- The goal is total compressed size or compression ratio rather than per-stream attribution
- NanoSpring output was produced with modified stream structure or a different compressor whose archive layout differs

## Inputs

- NanoSpring compressed archive (.NanoSpring, tar archive of compressed streams)
- Source FASTQ file used for compression (to obtain total base count)

## Outputs

- Per-stream byte sizes for consensus, error, and lone-read streams
- Bits-per-base per stream per dataset (e.g. stream_bits_per_base.csv)

## How to apply

After compressing each FASTQ dataset (e.g. snail, banana, hs1, hs4) with './NanoSpring -c -i file.fastq -o file.NanoSpring' using the default 20 threads, note that the compressed file is a tar archive of the different compressed streams, so untar it with standard tar utilities to obtain the individual consensus, error, and lone-read stream files. Measure each stream's byte size with standard file-size utilities (e.g. stat, ls -l, du). Divide each stream's size by the total number of bases in the source FASTQ, converting bytes to bits (x8), to obtain bits-per-base per stream. Tabulate the three streams' bits-per-base for every dataset in a single CSV so stream contributions can be compared across datasets. The rationale is that NanoSpring's approximate-assembly design encodes a consensus sequence, error streams relative to the consensus, and lone reads separately, so stream sizes reveal where compression gains come from.

## Related tools

- **NanoSpring** (Produces the .NanoSpring tar archive whose individual compressed streams (consensus, error, lone-read) are extracted and measured; run with default 20 threads) — https://github.com/qm2/NanoSpring
- **gzip** (Baseline general-purpose compressor whose output is compared against NanoSpring's per-stream sizes (3-6x worse compression))

## Examples

```
./NanoSpring -c -i file.fastq -o file.NanoSpring -t 20 && tar -xf file.NanoSpring -C streams/ && stat -c '%s %n' streams/*
```

## Evaluation signals

- The untarred directory contains identifiable consensus, error, and lone-read stream files, and the sum of their byte sizes (plus tar overhead) approximately equals the original .NanoSpring archive size
- Bits-per-base per stream sums to the dataset's reported total bits-per-base (0.35-0.65 bits/base for recently basecalled high quality nanopore datasets)
- Error stream computes to roughly 0.5 bits/base for most datasets, consistent with the paper's reported breakdown
- Lone-read stream is large for snail, banana and hs1 but negligible for hs4, matching the paper's Figure 3 pattern
- Losslessness verified by './NanoSpring -d' round-trip and 'cmp file.reads <(cat file.fastq | sed -n '2~4p')' before trusting stream sizes

## Limitations

- NanoSpring compresses only read sequences — read identifiers and quality values are not compressed, so stream attribution covers base sequences only
- Tar archive may add small per-file overhead, so per-stream sizes are approximate rather than exact contributions
- The breakdown pattern is dataset-dependent: lone-read contribution varies widely across datasets, so conclusions do not transfer between datasets

## Evidence

- [readme] Note that the compressed files are tar archives consisting of the different compressed streams: "Note that the compressed files are tar archives consisting of the different compressed streams, although we recommend using the ``NanoSpring extension"
- [abstract] NanoSpring compresses only base sequences: "NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base"
- [intro] NanoSpring is available as an open-source tool that requires only a FASTQ file as input and does not compress read identifiers or quality values: "NanoSpring is available as an open-source tool on GitHub, requires only a FASTQ file as input for compression, and does not compress read identifiers or quality values"
