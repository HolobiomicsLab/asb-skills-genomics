---
name: fastq-sequence-parsing
description: Use when input is a nanopore FASTQ file (possibly gzipped) and the task
  requires reading or extracting raw base sequences — e.g. for k-mer extraction, MinHash
  sketching, or read-sequence compression benchmarks.
when_to_use_negative:
- When the task requires compressed read identifiers or quality values — NanoSpring
  compresses only the base sequences in the FASTQ file and ignores quality values
  and read identifiers
- When input is already a NanoSpring archive (tar of compressed streams) — it must
  first be decompressed with '-d' before sequence parsing
- When the data are not sequencing reads in FASTQ format but e.g. compressed quality-score
  streams (as in ENANO's focus)
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: NanoSpring
  role: Consumes the FASTQ file and compresses only the read sequences; decompression
    restores the sequence lines for lossless verification against the parsed FASTQ
  repo: https://github.com/qm2/NanoSpring
- name: gzip
  role: Baseline handling of gzipped FASTQ input and general-purpose compression comparison
    on raw FASTQ
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:0c7d769f6e3aab06b7e4fbad51d2a9e0f6abcb44e3b9153683202ba167bba450
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - CoLoRd
  - ENANO
  - NanoSpring
  - gzip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# fastq-sequence-parsing

## Summary

Parse nanopore sequencing reads from FASTQ files and isolate the base sequences (line 2 of every 4-line record), since downstream tools like NanoSpring compress only sequences while ignoring read identifiers and quality values.

## When to use

Use when input is a nanopore FASTQ file (possibly gzipped) and the task requires reading or extracting raw base sequences — e.g. for k-mer extraction, MinHash sketching, or read-sequence compression benchmarks.

## When NOT to use

- When the task requires compressed read identifiers or quality values — NanoSpring compresses only the base sequences in the FASTQ file and ignores quality values and read identifiers
- When input is already a NanoSpring archive (tar of compressed streams) — it must first be decompressed with '-d' before sequence parsing
- When the data are not sequencing reads in FASTQ format but e.g. compressed quality-score streams (as in ENANO's focus)

## Inputs

- Nanopore FASTQ file (e.g. file.fastq), optionally gzipped (file.fastq.gz)
- 4-line FASTQ records containing header, base-sequence line, separator, and quality line

## Outputs

- Array of base sequences (one per read), stripped of identifiers and quality scores
- Decompressed sequence-only file (file.reads) that matches the original reads exactly
- k-mer sets of length k=23 per read for downstream MinHash indexing

## How to apply

Read the FASTQ file record by record (4 lines per read: header, sequence, '+', quality); when only base sequences are needed, extract every 2nd of 4 lines, e.g. via `sed -n '2~4p'`. NanoSpring requires only a FASTQ file as input and compresses only read sequences, so identifiers and quality lines can be dropped for its workflow. Once sequences are extracted, they can be decomposed into k-mers of length k=23 for MinHash sketching with n=60 hash functions, or fed to the compressor via './NanoSpring -c -i file.fastq -o file.NanoSpring' (gzipped FASTQ is also supported). Correctness is judged by losslessness: decompress with './NanoSpring -d -i file.NanoSpring -o file.reads -t 20' and compare the output against `sed -n '2~4p'` of the original FASTQ using `cmp`, which must show no difference.

## Related tools

- **NanoSpring** (Consumes the FASTQ file and compresses only the read sequences; decompression restores the sequence lines for lossless verification against the parsed FASTQ) — https://github.com/qm2/NanoSpring
- **gzip** (Baseline handling of gzipped FASTQ input and general-purpose compression comparison on raw FASTQ)

## Examples

```
./NanoSpring -c -i file.fastq -o file.NanoSpring; cmp file.reads <(cat file.fastq | sed -n '2~4p')
```

## Evaluation signals

- Decompressed file must be byte-identical to the sequence lines of the original: `cmp file.reads <(cat file.fastq | sed -n '2~4p')` returns no differences
- Number of extracted sequences equals the number of FASTQ records (total lines / 4)
- Extracted sequence lines contain only A/C/G/T/N characters and match the headers/quality lines 1:1 in record order
- Compression output is a tar archive of compressed streams and reported bits-per-base falls in the 0.35-0.65 range for recently basecalled high-quality datasets
- Installation verified by running the test script from the build directory

## Limitations

- Quality scores and read identifiers are not handled — they are ignored by NanoSpring and must be preserved separately if the full FASTQ is needed
- Only base sequences are counted when computing bits-per-base, so results are not directly comparable to tools that compress the entire FASTQ
- Older architectures may require removing -msse4.1/-mavx2 flags during build

## Evidence

- [readme] NanoSpring - Tool for compression of nanopore genomic reads in FASTQ format (gzipped input also supported). Compresses only the read sequences (i.e., ignores quality values and read identifiers).: "NanoSpring - Tool for compression of nanopore genomic reads in FASTQ format (gzipped input also supported). Compresses only the read sequences (i.e., ignores quality values and read identifiers)."
- [readme] Compare decompressed file to reads in original file to verify compression was lossless.: "Compare decompressed file to reads in original file to verify compression was lossless."
- [abstract] NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base: "NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base"
- [intro] NanoSpring is available as an open-source tool on GitHub, requires only a FASTQ file as input for compression, and does not compress read identifiers or quality values: "NanoSpring is available as an open-source tool on GitHub, requires only a FASTQ file as input for compression, and does not compress read identifiers or quality values"
- [readme] kmer size for the minhash (default 23): "kmer size for the minhash (default 23)"
