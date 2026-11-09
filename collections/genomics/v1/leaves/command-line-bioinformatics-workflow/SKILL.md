---
name: command-line-bioinformatics-workflow
description: Use when quantifying how a nanopore read compressor's bits/base ratio
  scales with sequencing coverage, by building a coverage ladder (e.g., 12x/23x/46x/69x/126x)
  from subsampled FASTQ files.
when_to_use_negative:
- The goal is compression of quality scores or read identifiers — NanoSpring compresses
  only base sequences in the FASTQ
- Input reads are short, low-error (e.g., Illumina) — specialized compressors like
  SPRING or PgRC assume short reads and are the appropriate choice
- A reference genome is available and reference-based compression is desired — a tool
  like RENANO targets that setting instead
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: NanoSpring
  role: Reference-free lossless compressor/decompressor for nanopore read sequences;
    CLI invoked with -c to compress and -d to decompress FASTQ subsamples at each
    coverage level
  repo: https://github.com/qm2/NanoSpring
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:5bfb930d5db36498b244e47fd4bbc19558907b1d89e4a120ab5f512ae202e252
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - NanoSpring
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# command-line-bioinformatics-workflow

## Summary

Reproduce NanoSpring's coverage-dependent compression curve by subsampling high-coverage nanopore FASTQs (e.g., CHM13 at 126x, banana at 177x), compressing each subsample with NanoSpring's CLI, and tabulating bits/base versus coverage.

## When to use

Use when quantifying how a nanopore read compressor's bits/base ratio scales with sequencing coverage, by building a coverage ladder (e.g., 12x/23x/46x/69x/126x) from subsampled FASTQ files.

## When NOT to use

- The goal is compression of quality scores or read identifiers — NanoSpring compresses only base sequences in the FASTQ
- Input reads are short, low-error (e.g., Illumina) — specialized compressors like SPRING or PgRC assume short reads and are the appropriate choice
- A reference genome is available and reference-based compression is desired — a tool like RENANO targets that setting instead

## Inputs

- High-coverage nanopore FASTQ file (e.g., CHM13 human genome at 126x coverage)
- High-coverage nanopore FASTQ file (e.g., banana genome at 177x coverage)

## Outputs

- Compressed .NanoSpring tar archive per coverage level
- coverage_bits_per_base.csv mapping coverage values to bits per base for each dataset
- Decompressed .reads files used for losslessness verification

## How to apply

Randomly select reads from the full-coverage FASTQ (CHM13 126x, banana 177x) to build a coverage ladder (12x, 23x, 46x, 69x, 126x). Compress each subsample with './NanoSpring -c -i <fastq> -o <out>.NanoSpring' using defaults (20 threads, kmer 23, 60 hash functions, overlap sketch threshold 6). Record the byte size of each compressed .NanoSpring tar archive, normalize by total input bases to get bits per base, and verify losslessness by decompressing ('./NanoSpring -d -i <out>.NanoSpring -o <out>.reads') and comparing sequences to the original FASTQ. Tabulate bits/base versus coverage as coverage_bits_per_base.csv; CHM13 should reproduce the reported 0.83, 0.68, 0.60, 0.57, 0.56 bits/base trend.

## Related tools

- **NanoSpring** (Reference-free lossless compressor/decompressor for nanopore read sequences; CLI invoked with -c to compress and -d to decompress FASTQ subsamples at each coverage level) — https://github.com/qm2/NanoSpring

## Examples

```
./NanoSpring -c -i chm13_12x.fastq -o chm13_12x.NanoSpring
```

## Evaluation signals

- CHM13 subsampled values reproduce the reported trend of 0.83, 0.68, 0.60, 0.57 and 0.56 bits/base at 12x, 23x, 46x, 69x and 126x coverage (monotonically improving with coverage)
- Decompressed .reads sequences match the original FASTQ sequences for every subsample (losslessness check via cmp/diff on the 2nd line of every 4-line record)
- bits/base computed as compressed .NanoSpring archive byte size divided by total number of bases in the input FASTQ falls in the ~0.5–0.9 range reported for high-quality nanopore data
- Compressed outputs are valid tar archives of the different compressed streams, using the .NanoSpring extension

## Limitations

- NanoSpring compresses only base sequences and ignores quality values and read identifiers, so bits/base here is not directly comparable to whole-FASTQ compression ratios
- On older CPU architectures the -msse4.1/-mavx2 flags in CMakeLists.txt may need to be removed or adjusted to build NanoSpring
- Compression ratio is acknowledged as not at fundamental limits; the assembly algorithm may need improvement to get closer to those limits

## Evidence

- [methods] Subsample the CHM13 (126x) FASTQ by randomly selecting reads to reach 12x, 23x, 46x, 69x, and 126x coverage: [evidence span withheld — anchor status: not_found]
- [readme] For compressing file.fastq using default 20 threads.: "./NanoSpring -c -i file.fastq -o file.NanoSpring"
- [methods] Decompress NanoSpring archives to reads: "Compare decompressed file to reads in original file to verify compression was lossless."
- [results] NanoSpring achieves 0.83, 0.68, 0.60, 0.57 and 0.56 bits/base: [evidence span withheld — anchor status: not_found]
- [readme] NanoSpring compresses only base sequences: "Compresses only the read sequences (i.e., ignores quality values and read identifiers)."
