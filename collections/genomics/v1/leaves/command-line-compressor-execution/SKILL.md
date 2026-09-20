---
name: command-line-compressor-execution
description: Use when you need to reproduce lossless compression ratios on nanopore
  FASTQ read sequences, e.g. benchmarking NanoSpring against Gzip (~2.2 bpb), ENANO
  (~1.9 bpb), or CoLoRd on the sa/zymo/snail/banana/hs1-hs4 datasets.
when_to_use_negative:
- You need to compress read identifiers or quality scores — NanoSpring only compresses
  the base sequences of the FASTQ file
- Input reads are short with low error rates — existing compressors like SPRING and
  PgRC target short reads; nanopore long reads need this specialized workflow
- You require reference-based compression — this skill covers the reference-free mode;
  reference-based tools like RENANO are out of scope
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3366
tools:
- name: NanoSpring
  role: Primary reference-free compressor; compresses only read sequences from FASTQ
    using approximate assembly, invoked via ./NanoSpring -c/-d
  repo: https://github.com/qm2/NanoSpring
- name: gzip
  role: General-purpose compression baseline (~2.2 bits/base on read sequences)
- name: ENANO
  role: Nanopore-specific comparison compressor using a context-based model followed
    by arithmetic coding for read sequences (~1.9 bits/base)
- name: CoLoRd
  role: State-of-the-art reference-free comparison compressor run in balanced mode
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:7971694490e073bad84adbd6f72b0c93cf84861a3e5ef73aab26e2973b308ea1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - CoLoRd
  - ENANO
  - NanoSpring
  - gzip
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# command-line-compressor-execution

## Summary

Executes FASTQ read-sequence compressors (NanoSpring, gzip, ENANO, CoLoRd) from the command line on nanopore datasets to produce compressed archives whose sizes yield bits-per-base compression ratios as reported in Table 2 of the NanoSpring paper.

## When to use

Use when you need to reproduce lossless compression ratios on nanopore FASTQ read sequences, e.g. benchmarking NanoSpring against Gzip (~2.2 bpb), ENANO (~1.9 bpb), or CoLoRd on the sa/zymo/snail/banana/hs1-hs4 datasets.

## When NOT to use

- You need to compress read identifiers or quality scores — NanoSpring only compresses the base sequences of the FASTQ file
- Input reads are short with low error rates — existing compressors like SPRING and PgRC target short reads; nanopore long reads need this specialized workflow
- You require reference-based compression — this skill covers the reference-free mode; reference-based tools like RENANO are out of scope

## Inputs

- Nanopore FASTQ (or FASTQ.gz) files, e.g. sa, zymo, snail, banana, hs1, hs2, hs3, hs4 datasets
- Built NanoSpring executable and comparison compressors (gzip/pigz, ENANO, CoLoRd)

## Outputs

- Compressed archives (file.NanoSpring tar archive of compressed streams; gzip/ENANO/CoLoRd archives)
- Decompressed read-sequence files (file.reads) for lossless verification
- Per-dataset, per-compressor bits-per-base values, e.g. saved as table2_bits_per_base.tsv

## How to apply

Build or install NanoSpring (conda `nanospring` or cmake build from github.com/qm2/NanoSpring), then compress each FASTQ with default settings (20 threads, kmer size 23, 60 hash functions, overlap sketch threshold 6) via './NanoSpring -c -i file.fastq -o file.NanoSpring'. Run comparison tools in parallel: gzip/pigz on the raw FASTQ, ENANO, and reference-free CoLoRd in balanced mode. Treat each .NanoSpring output as a tar archive of compressed streams and count only base-sequence bytes (identifiers and quality scores are not compressed), then divide total compressed bits by the total number of sequenced bases to obtain bits per base. Validate each run by decompressing with './NanoSpring -d -i file.NanoSpring -o file.reads -t 20' and confirming losslessness with 'cmp file.reads <(cat file.fastq | sed -n '2~4p')'.

## Related tools

- **NanoSpring** (Primary reference-free compressor; compresses only read sequences from FASTQ using approximate assembly, invoked via ./NanoSpring -c/-d) — https://github.com/qm2/NanoSpring
- **gzip** (General-purpose compression baseline (~2.2 bits/base on read sequences))
- **ENANO** (Nanopore-specific comparison compressor using a context-based model followed by arithmetic coding for read sequences (~1.9 bits/base))
- **CoLoRd** (State-of-the-art reference-free comparison compressor run in balanced mode)

## Examples

```
./NanoSpring -c -i file.fastq -o file.NanoSpring && ./NanoSpring -d -i file.NanoSpring -o file.reads -t 20 && cmp file.reads <(cat file.fastq | sed -n '2~4p')
```

## Evaluation signals

- Decompressed file passes losslessness check: 'cmp file.reads <(cat file.fastq | sed -n '2~4p')' returns no differences
- Bits-per-base values for recently basecalled high-quality human datasets (hs2, hs3, hs4) fall in the reported 0.35-0.65 bits/base range (0.7 bpb or below on hs2/hs3, 0.35 bpb on hs4)
- NanoSpring compression is 3-6x better than gzip and matches per-dataset values in Table 2 of the paper
- Compression ratios are competitive with reference-free CoLoRd while decompression with 20 threads is >4x faster

## Limitations

- NanoSpring archives are tar files of compressed streams — care is needed when attributing archive bytes to base-sequence compression only
- Building from source on older architectures may require removing the -msse4.1 flag from CMakeLists.txt (and -mavx2 for libbsc), or building minimap2 with 'make sse2only=1'
- Compression ratio is not at the fundamental limits; the assembly algorithm would need improvement to get closer to theoretical limits
- Decompression memory can be tuned via --decompression-memory (default 5 GB), but very low values may slightly reduce speed and settings are approximate

## Evidence

- [intro] NanoSpring is available as an open-source tool on GitHub, requires only a FASTQ file as input for compression, and does not compress read identifiers or quality values: "NanoSpring is available as an open-source tool on GitHub, requires only a FASTQ file as input for compression, and does not compress read identifiers or quality values"
- [abstract] For recently basecalled high quality nanopore datasets, NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base: "For recently basecalled high quality nanopore datasets, NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base"
- [methods] Compress FASTQ files with NanoSpring via './NanoSpring -c -i file.fastq -o file.NanoSpring': "./NanoSpring -c -i file.fastq -o file.NanoSpring"
- [readme] Note that the compressed files are tar archives consisting of the different compressed streams, although we recommend using the .NanoSpring extension: "Note that the compressed files are tar archives consisting of the different compressed streams, although we recommend using the ``NanoSpring extension as in the examples shown below."
- [readme] Compare decompressed file to reads in original file to verify compression was lossless: "Compare decompressed file to reads in original file to verify compression was lossless."
