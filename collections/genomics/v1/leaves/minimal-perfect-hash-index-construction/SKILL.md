---
name: minimal-perfect-hash-index-construction
description: 'Use when you need to index reads by their MinHash sketches so that a
  query sequence can efficiently retrieve all reads matching at least t of n hash
  functions (NanoSpring defaults: k=23, n=60, t=6).'
when_to_use_negative:
- Reads are short and low-error (e.g. Illumina) — compressors like SPRING and PgRC
  assume short reads and the minhash overlap model targets long, higher-error nanopore
  reads
- You only need sequence compression without overlap/assembly lookup — plain archival
  compressors or the base NanoSpring stages may suffice without materializing the
  index
- Quality scores or read identifiers need compression — NanoSpring focuses only on
  the base sequences in the FASTQ file
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NanoSpring
  role: Builds the n=60 MinHash sketches per read and the BBHash minimal perfect hash
    lookup tables serialized to minhash_index.bin; query returns reads matching >=
    6 of 60 sketch values
  repo: https://github.com/qm2/NanoSpring
- name: gzip
  role: Baseline general-purpose compressor against which NanoSpring's compression
    ratio is compared (3-6x lower bits per base)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:4271db574d0ad53815673532e08b27744a52f0867a9ae507dda615b96a84c79e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - NanoSpring
  - gzip
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# minimal-perfect-hash-index-construction

## Summary

Builds a MinHash lookup index for nanopore read overlap detection by mapping each of n=60 MinHash values per read into BBHash minimal perfect hash tables, enabling fast retrieval of reads sharing at least t=6 sketch values with a query.

## When to use

Use when you need to index reads by their MinHash sketches so that a query sequence can efficiently retrieve all reads matching at least t of n hash functions (NanoSpring defaults: k=23, n=60, t=6).

## When NOT to use

- Reads are short and low-error (e.g. Illumina) — compressors like SPRING and PgRC assume short reads and the minhash overlap model targets long, higher-error nanopore reads
- You only need sequence compression without overlap/assembly lookup — plain archival compressors or the base NanoSpring stages may suffice without materializing the index
- Quality scores or read identifiers need compression — NanoSpring focuses only on the base sequences in the FASTQ file

## Inputs

- FASTQ file of nanopore basecalled reads (gzipped FASTQ also supported)
- minhash parameters: kmer size k=23, number of hash functions n=60, overlap sketch threshold t=6

## Outputs

- minhash_index.bin: 60 serialized BBHash lookup tables mapping MinHash values to read identifiers
- Query results: list of reads matching a query sketch for at least t=6 of 60 hash functions

## How to apply

Parse the input FASTQ and extract all k-mers of length k=23 (NanoSpring's default minhash kmer size) from each read. Compute n=60 hash functions over each read's k-mer set to produce a 60-value MinHash sketch per read. For each of the 60 hash functions, build a static lookup table with BBHash (minimal perfect hashing) that maps each MinHash value to the list of reads sharing it, and serialize the tables to minhash_index.bin. During lookup, compute the 60-value sketch of the query and return all reads whose sketch matches for at least t=6 of the 60 hash functions (the default --overlap-sketch-thr). The rationale is that overlapping nanopore reads share enough k-mers to hit the sketch threshold, enabling approximate-assembly graph construction without a reference.

## Related tools

- **NanoSpring** (Builds the n=60 MinHash sketches per read and the BBHash minimal perfect hash lookup tables serialized to minhash_index.bin; query returns reads matching >= 6 of 60 sketch values) — https://github.com/qm2/NanoSpring
- **gzip** (Baseline general-purpose compressor against which NanoSpring's compression ratio is compared (3-6x lower bits per base))

## Examples

```
./NanoSpring -c -i file.fastq -k 23 -n 60 --overlap-sketch-thr 6 -o file.NanoSpring
```

## Evaluation signals

- minhash_index.bin exists, is loadable, and contains 60 lookup tables (one per hash function)
- Querying the index with a read's own sketch returns that read (self-match trivially satisfies the t=6 of 60 threshold)
- Changing -k, -n, or --overlap-sketch-thr on the command line (e.g. -k 20 -n 64 --overlap-sketch-thr 8) changes index construction and query behavior as expected
- Downstream overlap detection/assembly stages succeed, implying retrieved read sets cover true overlaps
- Round-trip: compressing then decompressing with ./NanoSpring -d reproduces the original read sequences bit-for-bit (verified with cmp against sed -n '2~4p' of the FASTQ)

## Limitations

- The overlap sketch threshold (default 6) trades recall for speed; very divergent or short overlaps may fall below t=6 of 60 matches
- BBHash tables are static — the index must be rebuilt if reads are added or parameters (k, n) change
- NanoSpring is targeted at nanopore reads from recent basecallers (median error rate close to 5% or better); higher error rates may break sketch matching

## Evidence

- [readme] kmer size for the minhash (default 23): "-k [ --kmer ] arg (=23)        kmer size for the minhash (default 23)"
- [readme] number of hash functions for minhash (default 60): "-n [ --num-hash ] arg (=60)    number of hash functions for minhash (default
                                 60)"
- [readme] the overlap sketch threshold for minhash (default 6): "--overlap-sketch-thr arg (=6)  the overlap sketch threshold for minhash
                                 (default 6)"
- [abstract] NanoSpring, a reference-free compressor for nanopore sequencing reads, relying on an approximate assembly approach: "We present NanoSpring, a reference-free compressor for nanopore sequencing reads, relying on an approximate assembly approach."
- [methods] git clone https://github.com/qm2/NanoSpring.git: "git clone https://github.com/qm2/NanoSpring.git"
- [intro] the error rate has fallen dramatically in the recent years with the advent of deep learning based basecallers which achieve median error rate close to 5%: "the error rate has fallen dramatically in the recent years with the advent of deep learning based basecallers which achieve median error rate close to 5% or better5"
