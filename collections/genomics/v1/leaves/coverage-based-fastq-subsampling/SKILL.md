---
name: coverage-based-fastq-subsampling
description: Use when you need to characterize how a compression tool's bits/base
  performance depends on sequencing coverage, by downsampling a high-coverage FASTQ
  to discrete coverage levels before compression.
when_to_use_negative:
- You need lossless compression of read identifiers or quality scores — NanoSpring
  compresses only the base sequences in the FASTQ
- Input reads are short, low-error Illumina-style reads — the approximate-assembly
  approach targets long, higher-error nanopore reads, and tools like SPRING/PgRC assume
  short reads
- The dataset is already at or below the lowest target coverage, so no meaningful
  subsampling ladder can be constructed
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NanoSpring
  role: Reference-free lossless compressor applied to each coverage subsample to measure
    bits/base; run with default parameters (20 threads, kmer 23, 60 hash functions,
    overlap sketch threshold 6)
  repo: https://github.com/qm2/NanoSpring
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:84ffb2db63397013ffc79b575523ca9ae13045e5e21c5d86c02b18e5d4d45339
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - NanoSpring
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# coverage-based-fastq-subsampling

## Summary

Randomly subsample a deeply sequenced nanopore FASTQ (e.g. banana at 177x or CHM13/hs3 at 126x) to a ladder of lower coverages (12x, 23x, 46x, 69x) so that compression ratio (bits/base) can be measured as a function of sequencing depth, as done for the NanoSpring Figure 4 curve.

## When to use

Use when you need to characterize how a compression tool's bits/base performance depends on sequencing coverage, by downsampling a high-coverage FASTQ to discrete coverage levels before compression.

## When NOT to use

- You need lossless compression of read identifiers or quality scores — NanoSpring compresses only the base sequences in the FASTQ
- Input reads are short, low-error Illumina-style reads — the approximate-assembly approach targets long, higher-error nanopore reads, and tools like SPRING/PgRC assume short reads
- The dataset is already at or below the lowest target coverage, so no meaningful subsampling ladder can be constructed

## Inputs

- High-coverage nanopore FASTQ file (e.g. CHM13 at 126x, banana at 177x)
- Target coverage ladder values (e.g. 12x, 23x, 46x, 69x, 126x) and reference genome size for base-count calculation

## Outputs

- Subsampled FASTQ files at each coverage level
- Bits-per-base values per coverage level per dataset (e.g. coverage_bits_per_base.csv)
- Losslessly decompressed read sequences (.reads files) for verification

## How to apply

Randomly select reads from the full-coverage FASTQ (CHM13 at 126x, banana at 177x) until the target coverage levels of 12x, 23x, 46x, and 69x (plus the unsampled 126x/177x points) are reached, using total base count relative to genome size to verify each level. Compress each subsample with NanoSpring using default parameters ('./NanoSpring -c -i <fastq> -o <out>.NanoSpring'; 20 threads, kmer 23, 60 hash functions, overlap sketch threshold 6). Record the byte size of each .NanoSpring tar archive and normalize by the total input bases to compute bits per base. The reported trend on subsampled CHM13 is 0.83, 0.68, 0.60, 0.57 and 0.56 bits/base at 12x, 23x, 46x, 69x and 126x, showing compression improves with coverage because the approximate-assembly approach finds more overlaps at higher redundancy.

## Related tools

- **NanoSpring** (Reference-free lossless compressor applied to each coverage subsample to measure bits/base; run with default parameters (20 threads, kmer 23, 60 hash functions, overlap sketch threshold 6)) — https://github.com/qm2/NanoSpring

## Examples

```
./NanoSpring -c -i chm13_12x.fastq -o chm13_12x.NanoSpring && ./NanoSpring -d -i chm13_12x.NanoSpring -o chm13_12x.reads
```

## Evaluation signals

- CHM13 subsample compression ratios reproduce the reported trend 0.83, 0.68, 0.60, 0.57, 0.56 bits/base at 12x, 23x, 46x, 69x, 126x
- Bits/base monotonically decreases with increasing coverage for both banana and CHM13 curves
- Total bases in each subsample FASTQ match the intended coverage level times genome size
- Decompression with './NanoSpring -d' followed by cmp against the original read lines (sed -n '2~4p') confirms losslessness for every subsample
- Each compressed output is a valid tar archive of compressed streams with the .NanoSpring extension

## Limitations

- NanoSpring compresses only read sequences — quality values and read identifiers are not compressed, so bits/base excludes those streams
- Compression ratio is not at fundamental limits; future work on the assembly algorithm is needed to get closer to the theoretical limits
- Very low coverage subsamples reduce the number of read overlaps available to the approximate assembly, weakening compression
- For older architectures, the -msse4.1/-mavx2 compile flags in CMakeLists.txt may need to be removed to build NanoSpring

## Evidence

- [results] subsampled CHM13 coverage trend: [evidence span withheld — anchor status: not_found]
- [results] compression improves with coverage: [evidence span withheld — anchor status: not_found]
- [methods] compress command: "./NanoSpring -c -i file.fastq -o file.NanoSpring"
- [readme] lossless verification: "Compare decompressed file to reads in original file to verify compression was lossless."
- [readme] sequences only scope: "Compresses only the read sequences (i.e., ignores quality values and read identifiers)."
