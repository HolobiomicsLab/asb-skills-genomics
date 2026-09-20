---
name: minhash-sketching-of-sequencing-reads
description: Use when you need to identify nanopore reads that overlap a query sequence
  using k-mer MinHash sketches — e.g., to reproduce or extend NanoSpring's overlap-finding
  stage for approximate-assembly-based compression of long, error-prone ONT reads.
when_to_use_negative:
- Reads are short and low-error (e.g., Illumina-style) — tools like SPRING and PgRC
  assume short reads and the minhash overlap approach targets long nanopore reads.
- You need reference-based compression — NanoSpring's minhash indexing is part of
  its reference-free approximate assembly approach.
- You need to compress read identifiers or quality values — NanoSpring only compresses
  the base sequences in the FASTQ file.
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: NanoSpring
  role: Implements the MinHash sketching (k=23, n=60) and BBHash-based lookup returning
    reads matching ≥ t=6 sketch values
  repo: https://github.com/qm2/NanoSpring
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:a549b1449ee5ce4e408ad96af2dd1fd73b4977f3faf83f20b09bbbe18331c11c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - NanoSpring
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# minhash-sketching-of-sequencing-reads

## Summary

Index nanopore FASTQ read sequences by computing MinHash sketches over k-mers (default k=23, n=60 hash functions in NanoSpring) and building static lookup tables so that reads overlapping a query sequence can be found by matching at least t (default 6) of the n sketch values. This enables the approximate-assembly overlap step of NanoSpring's reference-free compression.

## When to use

Use when you need to identify nanopore reads that overlap a query sequence using k-mer MinHash sketches — e.g., to reproduce or extend NanoSpring's overlap-finding stage for approximate-assembly-based compression of long, error-prone ONT reads.

## When NOT to use

- Reads are short and low-error (e.g., Illumina-style) — tools like SPRING and PgRC assume short reads and the minhash overlap approach targets long nanopore reads.
- You need reference-based compression — NanoSpring's minhash indexing is part of its reference-free approximate assembly approach.
- You need to compress read identifiers or quality values — NanoSpring only compresses the base sequences in the FASTQ file.

## Inputs

- FASTQ file of nanopore (ONT) read base sequences (gzipped FASTQ also supported by NanoSpring)
- Query read sequence or read sketch to look up overlaps for

## Outputs

- n=60 BBHash minimal-perfect-hash lookup tables (serialized, e.g., minhash_index.bin)
- Set of reads sharing at least t=6 of the 60 MinHash sketch values with the query
- 60-value MinHash sketch per read

## How to apply

Parse the input FASTQ and extract all k-mers of length k=23 (NanoSpring's default minhash kmer size) from each read. Compute n=60 hash functions (the default num-hash) over each read's k-mer set to produce a 60-value MinHash sketch per read. Build one static lookup table per hash function using BBHash (minimal perfect hashing), mapping each MinHash value to the reads sharing it, and serialize the tables (e.g., to minhash_index.bin). For a query, compute its sketch and return all reads matching the query sketch for at least t=6 of the 60 values (the default overlap-sketch-threshold), since sketch agreement above this threshold indicates a probable overlap despite ONT basecalling errors of roughly 5% or better.

## Related tools

- **NanoSpring** (Implements the MinHash sketching (k=23, n=60) and BBHash-based lookup returning reads matching ≥ t=6 sketch values) — https://github.com/qm2/NanoSpring

## Examples

```
./NanoSpring -c -i file.fastq -k 23 -n 60 --overlap-sketch-thr 6 -o file.NanoSpring
```

## Evaluation signals

- minhash_index.bin exists and is loadable
- A self-query: using a read's own sketch as the query returns that read, since it matches all 60 values and trivially satisfies the t=6 threshold
- Sketches contain exactly n=60 values computed from k=23-mers of each read
- Tuning flags (-k, -n, --overlap-sketch-thr) change the index consistently (e.g., -k 20 -n 64 --overlap-sketch-thr 8 yields 64 tables with a threshold of 8)
- Returned overlap candidates are consistent with approximate read overlaps on known-overlapping read pairs

## Limitations

- Only approximate: sketch matching with threshold t=6 of 60 can miss true overlaps or admit false ones, especially under very low MinHash parameter settings
- NanoSpring does not compress read identifiers or quality values, so the sketch index covers base sequences only
- Very low overlap-sketch-threshold or large thread counts can affect downstream memory/speed characteristics (README notes such approximations may have no effect at extreme settings)

## Evidence

- [readme] kmer size and hash count defaults: "-k [ --kmer ] arg (=23)        kmer size for the minhash (default 23)
  -n [ --num-hash ] arg (=60)    number of hash functions for minhash"
- [readme] overlap sketch threshold default: "--overlap-sketch-thr arg (=6)  the overlap sketch threshold for minhash (default 6)"
- [intro] tool scope: "NanoSpring is available as an open-source tool on GitHub, requires only a FASTQ file as input for compression, and does not compress read identifiers or quality values"
- [methods] compression invocation: "./NanoSpring -c -i file.fastq -o file.NanoSpring"
