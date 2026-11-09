---
name: contig-read-stream-encoding
description: Use when reconstructing or reimplementing the NanoSpring lossless compression
  pipeline for ONT FASTQ reads and you need to reproduce the per-contig stream encoding
  stage after overlap detection and consensus building.
when_to_use_negative:
- You need to compress read identifiers or quality scores — NanoSpring focuses only
  on the base sequences in the FASTQ file
- Reads are short with low error rates (e.g., short-read data) — the approximate-assembly
  encoding assumes long nanopore reads with ~5% or better basecaller error rates
- Only trivial compression is needed — general purpose compressors like gzip may suffice
  if a 3-6x ratio improvement is not required
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: NanoSpring
  role: Primary compressor that performs the per-contig stream encoding and bundles
    compressed streams into a .NanoSpring tar archive
  repo: https://github.com/qm2/NanoSpring
- name: fast-lzma2
  role: LZMA2-based lossless library used to compress the encoded streams
  repo: https://github.com/conor42/fast-lzma2
- name: libbsc
  role: Block-sorting lossless compression library used to compress the encoded streams
  repo: https://github.com/IlyaGrebnov/libbsc
- name: minimap2
  role: Embeddable aligner used upstream for read overlap detection that groups reads
    into contigs prior to stream encoding
- name: gzip
  role: General-purpose baseline compressor for comparing achieved bits-per-base ratios
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:a835aaa37e4c733e5ebdac0bade89c4a7896510d9a8b73e37e30347acae40498
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - NanoSpring
  - fast-lzma2
  - gzip
  - libbsc
  - minimap2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# contig-read-stream-encoding

## Summary

Encodes nanopore sequencing reads assigned to each approximate-assembly contig into ordered per-contig streams (consensus, positions, errors, orientation flag, read index), with unaligned reads kept in a separate lone-reads stream, so the streams can be entropy-compressed into a single .NanoSpring archive.

## When to use

Use when reconstructing or reimplementing the NanoSpring lossless compression pipeline for ONT FASTQ reads and you need to reproduce the per-contig stream encoding stage after overlap detection and consensus building.

## When NOT to use

- You need to compress read identifiers or quality scores — NanoSpring focuses only on the base sequences in the FASTQ file
- Reads are short with low error rates (e.g., short-read data) — the approximate-assembly encoding assumes long nanopore reads with ~5% or better basecaller error rates
- Only trivial compression is needed — general purpose compressors like gzip may suffice if a 3-6x ratio improvement is not required

## Inputs

- ONM nanopore sequencing reads in FASTQ format (gzipped FASTQ also supported); only base sequences are compressed, not read identifiers or quality scores
- Per-contig read assignments and consensus sequences from the NanoSpring approximate assembly stage

## Outputs

- Ordered per-contig encoded streams: consensus sequence, start/error positions, error types, erroneous bases, reverse-complement flags, and 32-bit read indices
- A separate lone-reads stream for unaligned reads
- A single tar archive (conventionally .NanoSpring) containing the LZMA2 (fast-lzma2) and BSC (libbsc) compressed streams
- Decompressed read sequences file (conventionally .reads) after decompression

## How to apply

After building NanoSpring from source (cmake ..; make -j) and compressing a FASTQ with ./NanoSpring -c -i file.fastq -o file.NanoSpring using defaults (20 threads, kmer 23, 60 hash functions, overlap sketch threshold 6, minimap2 k 20 / w 50), encode every read aligned to a contig relative to its consensus sequence: store the read's start position on the consensus, its errors, a reverse-complement boolean flag, and a 32-bit unsigned integer read index. Reads that could not be aligned to any contig are written to a separate lone-reads stream so they remain losslessly recoverable. The streams (consensus sequence, start/error position, error type, erroneous base, reverse-complement flag, read index) are then compressed with LZMA2 (fast-lzma2) and BSC (libbsc) and bundled into a tar archive with the .NanoSpring extension. Rationale: encoding reads as small deltas against a contig consensus exploits the low (~5% or better) error rates of deep-learning-based nanopore basecallers, achieving 0.35-0.65 bits per base on recent high-quality datasets.

## Related tools

- **NanoSpring** (Primary compressor that performs the per-contig stream encoding and bundles compressed streams into a .NanoSpring tar archive) — https://github.com/qm2/NanoSpring
- **fast-lzma2** (LZMA2-based lossless library used to compress the encoded streams) — https://github.com/conor42/fast-lzma2
- **libbsc** (Block-sorting lossless compression library used to compress the encoded streams) — https://github.com/IlyaGrebnov/libbsc
- **minimap2** (Embeddable aligner used upstream for read overlap detection that groups reads into contigs prior to stream encoding)
- **gzip** (General-purpose baseline compressor for comparing achieved bits-per-base ratios)

## Examples

```
./NanoSpring -c -i file.fastq -o file.NanoSpring && ./NanoSpring -d -i file.NanoSpring -o file.reads && cmp file.reads <(cat file.fastq | sed -n '2~4p')
```

## Evaluation signals

- The compressed .NanoSpring tar archive is created and contains the different compressed streams
- Roundtrip is lossless: cmp file.reads <(cat file.fastq | sed -n '2~4p') reports no differences
- Total bits per base, computed as (compressed archive size in bits) / (total number of bases), falls within the reported 0.35-0.65 bits per base range on recent high-quality nanopore datasets
- Each contig's encoded streams contain exactly the fields expected: consensus sequence, start/error position, error type, erroneous base, reverse-complement flag, and read index, with unaligned reads routed to the lone-reads stream
- Compression is 3-6x smaller than gzip on the same FASTQ file for recent basecaller versions

## Limitations

- NanoSpring does not compress read identifiers or quality values — only base sequences in the FASTQ file
- Building from source on older architectures may require removing the -msse4.1 flag from target_compile_options and the -mavx2 flag for libbsc in CMakeLists.txt, and compiling minimap2 with sse2only
- The compression ratio is not at fundamental limits; the assembly algorithm needs improvement to get closer to the fundamental limits
- Older basecaller versions with higher error rates compress worse (the 3x+ advantage over gzip holds for recent basecaller versions)

## Evidence

- [results] For each read in a contig, NanoSpring stores the start position on the consensus sequence, the errors in the read, a reverse-complement flag (bool), and a 32-bit unsigned integer read index.: [evidence span withheld — anchor status: not_found]
- [intro] the error rate has fallen dramatically in the recent years with the advent of deep learning based basecallers which achieve median error rate close to 5% or better5, suggesting that a similar: "the error rate has fallen dramatically in the recent years with the advent of deep learning based basecallers which achieve median error rate close to 5% or better"
- [readme] Note that the compressed files are tar archives consisting of the different compressed streams, although we recommend using the ``NanoSpring extension: "the compressed files are tar archives consisting of the different compressed streams, although we recommend using the ``NanoSpring extension"
- [readme] Compare decompressed file to reads in original file to verify compression was lossless.: "Compare decompressed file to reads in original file to verify compression was lossless."
- [abstract] NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base: "NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base"
