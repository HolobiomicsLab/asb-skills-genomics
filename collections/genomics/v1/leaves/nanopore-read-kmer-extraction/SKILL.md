---
name: nanopore-read-kmer-extraction
description: Use when compressing or indexing ONT nanopore read sequences from a FASTQ
  file and a k-mer–based MinHash sketch is needed to find reads overlapping a query
  sequence.
when_to_use_negative:
- Input is not a FASTQ base-sequence file — NanoSpring ignores read identifiers and
  quality scores, so this skill is wrong for quality-score or header compression tasks
- Reads are short, low-error short-read data — NanoSpring's approximate-assembly approach
  targets long nanopore reads made feasible by deep-learning basecallers reaching
  ~5% or better median error rates
- A reference genome is available and a reference-based compressor (e.g. RENANO) is
  preferred
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: NanoSpring
  role: Computes the k-mer sets, the n=60 MinHash sketches, and the BBHash lookup
    tables; exposes -k, -n and --overlap-sketch-thr parameters during compression
  repo: https://github.com/qm2/NanoSpring
- name: gzip
  role: Baseline general-purpose compressor NanoSpring's k-mer/MinHash-driven approach
    is compared against (3-6x better compression)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:f59bcfbbf796d11094551bd999e3a6121980ba224462e6de5b18c8561394e780
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - NanoSpring
  - gzip
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# nanopore-read-kmer-extraction

## Summary

Extract all k-mers of length k=23 from the base sequences of nanopore FASTQ reads and compute n=60 MinHash hash values per read, as the first stages of NanoSpring's MinHash overlap-detection pipeline used by its approximate-assembly compressor.

## When to use

Use when compressing or indexing ONT nanopore read sequences from a FASTQ file and a k-mer–based MinHash sketch is needed to find reads overlapping a query sequence.

## When NOT to use

- Input is not a FASTQ base-sequence file — NanoSpring ignores read identifiers and quality scores, so this skill is wrong for quality-score or header compression tasks
- Reads are short, low-error short-read data — NanoSpring's approximate-assembly approach targets long nanopore reads made feasible by deep-learning basecallers reaching ~5% or better median error rates
- A reference genome is available and a reference-based compressor (e.g. RENANO) is preferred

## Inputs

- FASTQ file of nanopore read sequences (plain or gzipped, e.g. file.fastq or file.fastq.gz)
- k-mer size parameter k (default 23)
- number of hash functions n (default 60)
- overlap sketch threshold t (default 6)

## Outputs

- List of all 23-mers per read
- 60-value MinHash sketch per read
- n=60 BBHash lookup tables serialized to minhash_index.bin mapping MinHash values to sharing reads

## How to apply

Parse the input FASTQ (gzipped input is also supported) and read the base sequences of all nanopore reads, ignoring quality values and read identifiers since NanoSpring compresses only sequences. Extract every k-mer of length k=23 (the default minhash kmer size set via the -k/--kmer option) from each read. Over each read's 23-mer set, compute n=60 hash functions (the -n/--num-hash default) to produce a 60-value MinHash sketch per read. These sketches feed the downstream BBHash-based lookup tables, where a query read matches another read if it agrees on at least t=6 of the 60 sketch values (--overlap-sketch-thr default 6). Adjust k, n, or the threshold only with matching values at both index-build and query time, as demonstrated in the README's custom-parameter compression example.

## Related tools

- **NanoSpring** (Computes the k-mer sets, the n=60 MinHash sketches, and the BBHash lookup tables; exposes -k, -n and --overlap-sketch-thr parameters during compression) — https://github.com/qm2/NanoSpring
- **gzip** (Baseline general-purpose compressor NanoSpring's k-mer/MinHash-driven approach is compared against (3-6x better compression))

## Examples

```
./NanoSpring -c -i file.fastq -k 23 -n 60 --overlap-sketch-thr 6 -o file.NanoSpring
```

## Evaluation signals

- minhash_index.bin exists, is loadable, and contains 60 lookup tables (one per hash function)
- Querying the index with a read's own 60-value sketch returns that read, trivially satisfying the t=6-of-60 threshold
- Default parameters match the README: kmer size 23, number of hash functions 60, overlap sketch threshold 6
- Decompressed reads verify lossless compression: cmp file.reads <(cat file.fastq | sed -n '2~4p') shows no differences

## Limitations

- NanoSpring compresses only the base sequences in the FASTQ file and does not compress read identifiers or quality values
- Compression is not yet at fundamental limits; future work includes improving the assembly algorithm
- On older architectures the -msse4.1 and -mavx2 compile flags in CMakeLists.txt may need to be removed or adjusted

## Evidence

- [readme] kmer size for the minhash (default 23): "-k [ --kmer ] arg (=23)        kmer size for the minhash (default 23)"
- [readme] number of hash functions for minhash (default 60): "-n [ --num-hash ] arg (=60)    number of hash functions for minhash (default
                                 60)"
- [readme] overlap sketch threshold for minhash (default 6): "--overlap-sketch-thr arg (=6)  the overlap sketch threshold for minhash
                                 (default 6)"
- [intro] lossless reference-free compressor for nanopore sequencing reads: "In this work, we present NanoSpring, which is a lossless reference-free compressor for nanopore sequencing reads."
- [readme] Compresses only the read sequences, ignoring quality values and read identifiers: "Compresses only the read sequences (i.e., ignores quality values and read identifiers)."
