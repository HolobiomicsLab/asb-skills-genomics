---
name: read-overlap-threshold-querying
description: Use when building or querying a read-overlap index for nanopore FASTQ
  reads without a reference, e.g. to find reads that overlap a query sequence via
  MinHash sketches and a match threshold.
when_to_use_negative:
- The input is not read base sequences but quality values or read identifiers — NanoSpring
  does not compress or index those
- The goal is reference-based overlap detection with high-accuracy alignment (e.g.
  minimap2 mapping) rather than fast approximate MinHash matching
- Reads are short and low-error (e.g. Illumina) — NanoSpring and its overlap approach
  target long, higher-error nanopore reads
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: NanoSpring
  role: Provides the MinHash indexing stage whose defaults (-k 23, -n 60, --overlap-sketch-thr
    6) and BBHash lookup tables this skill reconstructs
  repo: https://github.com/qm2/NanoSpring
- name: minimap2
  role: Used downstream in the NanoSpring pipeline for the overlap/chaining stage
    that consumes the MinHash index candidates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:8b85a7bfba3064c126ec31e4d20adc3040e462363a69b683d275b44643601901
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - NanoSpring
  - minimap2
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# read-overlap-threshold-querying

## Summary

Reconstructs NanoSpring's MinHash-based read-overlap index: k-mer extraction (k=23), n=60 hash-function sketches, BBHash minimal-perfect-hash lookup tables, and threshold queries returning reads matching at least t=6 of 60 sketch values.

## When to use

Use when building or querying a read-overlap index for nanopore FASTQ reads without a reference, e.g. to find reads that overlap a query sequence via MinHash sketches and a match threshold.

## When NOT to use

- The input is not read base sequences but quality values or read identifiers — NanoSpring does not compress or index those
- The goal is reference-based overlap detection with high-accuracy alignment (e.g. minimap2 mapping) rather than fast approximate MinHash matching
- Reads are short and low-error (e.g. Illumina) — NanoSpring and its overlap approach target long, higher-error nanopore reads

## Inputs

- FASTQ file of nanopore reads (gzipped FASTQ also supported)
- Query sequence (e.g. a read's bases) to look up overlaps for
- Parameters: kmer size k (default 23), number of hash functions n (default 60), overlap sketch threshold t (default 6)

## Outputs

- n=60 BBHash static lookup tables mapping MinHash values to read identifiers (serialized as minhash_index.bin)
- For each query: the set of reads matching at least t=6 of the 60 sketch values (overlap candidates)

## How to apply

Parse the FASTQ file and take only the base sequences of each ONT read. Extract all k-mers of length k=23 from each read and compute n=60 hash functions over the k-mer set to yield a 60-value MinHash sketch per read. Build 60 static lookup tables with BBHash (minimal perfect hashing), one per hash function, mapping each MinHash value to the reads sharing it, then serialize the tables (e.g. minhash_index.bin). At query time, compute the sketch of the query sequence and return all reads matching the sketch for at least t=6 of the 60 hash functions (the overlap-sketch-thr default). These parameters mirror NanoSpring's CLI defaults: -k 23, -n 60, --overlap-sketch-thr 6; the index feeds the approximate-assembly stage that groups overlapping reads for reference-free compression.

## Related tools

- **NanoSpring** (Provides the MinHash indexing stage whose defaults (-k 23, -n 60, --overlap-sketch-thr 6) and BBHash lookup tables this skill reconstructs) — https://github.com/qm2/NanoSpring
- **minimap2** (Used downstream in the NanoSpring pipeline for the overlap/chaining stage that consumes the MinHash index candidates)

## Examples

```
./NanoSpring -c -i file.fastq -k 20 -n 64 --overlap-sketch-thr 8 -o file.NanoSpring
```

## Evaluation signals

- minhash_index.bin exists and is loadable
- Querying with a read's own sketch returns that read (self-query trivially satisfies the t=6 threshold)
- CLI-equivalent parameters match NanoSpring defaults: kmer=23, num-hash=60, overlap-sketch-thr=6
- Each of the 60 lookup tables maps every MinHash value from its hash function to the reads sharing it (minimal perfect hash coverage)
- Queries report only reads matching >= 6 of the 60 sketch values, not all sharing a single hash value

## Limitations

- Threshold t=6 is a heuristic overlap threshold; lowering it increases sensitivity but raises false-overlap candidates, raising it can miss true overlaps
- MinHash matching is approximate — matches depend on k=23 k-mer exactness and will degrade for reads with higher error rates
- NanoSpring indexes only base sequences; read identifiers and quality values are never compressed or indexed

## Evidence

- [readme] kmer size for minhash defaults and hash-function count: "-n [ --num-hash ] arg (=60)    number of hash functions for minhash (default"
- [readme] overlap sketch threshold default: "--overlap-sketch-thr arg (=6)  the overlap sketch threshold for minhash"
- [intro] NanoSpring scope: FASTQ base sequences only: "NanoSpring is available as an open-source tool on GitHub, requires only a FASTQ file as input for compression, and does not compress read identifiers or quality values"
- [readme] compression target context for the overlap stage: "NanoSpring - Tool for compression of nanopore genomic reads in FASTQ format (gzipped input also supported). Compresses only the read sequences"
