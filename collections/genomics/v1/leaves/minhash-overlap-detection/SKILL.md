---
name: minhash-overlap-detection
description: Use when building contigs or consensus sequences from long, error-prone
  nanopore reads without a reference — e.g., to find candidate overlapping reads for
  greedy contig extension, or to parameterize read overlap search in NanoSpring-style
  compression.
when_to_use_negative:
- Input is short, low-error Illumina-style read data — NanoSpring's MinHash overlap
  approach targets long ONT reads, and compressors like SPRING or PgRC assume short
  reads instead.
- The goal is read-level overlap for de novo assembly of a published genome rather
  than compression — NanoSpring's consensus graph is an intermediate for lossless
  sequence compression, not a polished assembly.
- You need to compress quality values or read identifiers — NanoSpring compresses
  only base sequences from the FASTQ.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: NanoSpring
  role: Implements the MinHash sketching and index search that finds candidate overlapping
    reads for contig extension; exposes --kmer, --num-hash, and --overlap-sketch-thr
    parameters
  repo: https://github.com/qm2/NanoSpring
- name: minimap2
  role: Confirms candidate MinHash overlaps by aligning reads to the current consensus
    (kmer size 20, window size 50, max-chain-iter 400) before graph insertion
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:18940440ce29eded8f0b56a884948d5efb2fee8695e557d3bec9b53ca00d4fed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - NanoSpring
  - minimap2
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# minhash-overlap-detection

## Summary

Detects overlapping nanopore reads by sketching sequences with MinHash and querying a MinHash index for candidates that overlap a growing consensus, as done in NanoSpring's approximate-assembly compression pipeline. Candidate overlaps are then confirmed with minimap2 alignment before insertion into a consensus graph.

## When to use

Use when building contigs or consensus sequences from long, error-prone nanopore reads without a reference — e.g., to find candidate overlapping reads for greedy contig extension, or to parameterize read overlap search in NanoSpring-style compression.

## When NOT to use

- Input is short, low-error Illumina-style read data — NanoSpring's MinHash overlap approach targets long ONT reads, and compressors like SPRING or PgRC assume short reads instead.
- The goal is read-level overlap for de novo assembly of a published genome rather than compression — NanoSpring's consensus graph is an intermediate for lossless sequence compression, not a polished assembly.
- You need to compress quality values or read identifiers — NanoSpring compresses only base sequences from the FASTQ.

## Inputs

- FASTQ file of nanopore read sequences (gzipped FASTQ also supported); identifiers and quality values are not used
- Current consensus sequence of the directed acyclic consensus graph (query against the MinHash index)

## Outputs

- Candidate overlapping reads retrieved from the MinHash index
- minimap2 alignments of candidates to the current consensus, added as edges in the consensus graph
- Greedy highest-weight consensus paths (contigs) used for approximate-assembly compression

## How to apply

Sketch each nanopore read with MinHash using the NanoSpring defaults: kmer size 23 (--kmer), 60 hash functions (--num-hash), and an overlap sketch threshold of 6 (--overlap-sketch-thr). Greedily query the MinHash index with the current consensus sequence of the directed acyclic consensus graph to retrieve candidate reads that plausibly overlap the contig being extended, which avoids all-vs-all read comparison. Confirm each candidate overlap by aligning the read to the consensus with minimap2 using kmer size 20, window size 50, and max-chain-iter 400. Insert aligned reads into the consensus graph (bounded by --edge-thr, default 4,000,000 edges) and extract the greedy highest-weight consensus path as the contig. Sketch threshold and hash-count tradeoffs determine sensitivity to the ~5% error rate of deep-learning basecalled ONT reads: lowering --kmer or raising --num-hash/--overlap-sketch-thr shifts recall vs. speed.

## Related tools

- **NanoSpring** (Implements the MinHash sketching and index search that finds candidate overlapping reads for contig extension; exposes --kmer, --num-hash, and --overlap-sketch-thr parameters) — https://github.com/qm2/NanoSpring
- **minimap2** (Confirms candidate MinHash overlaps by aligning reads to the current consensus (kmer size 20, window size 50, max-chain-iter 400) before graph insertion)

## Examples

```
./NanoSpring -c -i file.fastq -o file.NanoSpring -t 20 -k 23 -n 60 --overlap-sketch-thr 6
```

## Evaluation signals

- Decompression round-trip is lossless: `cmp file.reads <(cat file.fastq | sed -n '2~4p')` reports no differences.
- MinHash parameters match intended settings: --kmer 23, --num-hash 60, --overlap-sketch-thr 6 (defaults) appear in the run log/invocation.
- Candidate overlaps survive minimap2 alignment with the specified parameters (minimap-k 20, minimap-w 50, max-chain-iter 400) and are added to the consensus graph.
- The consensus graph stays within the edge limit (--edge-thr default 4,000,000) before extracting the greedy highest-weight consensus path.
- Compressed size falls near the paper's 0.35-0.65 bits per base range for recently basecalled high-quality nanopore datasets (3-6x better than gzip).

## Limitations

- Compression ratio is not at fundamental limits; the paper notes future work on improving the assembly algorithm to get closer to the fundamental limits.
- Very low --decompression-memory settings or many threads can slightly reduce speed; the memory cap is approximate and may have no effect at very low settings.
- Older CPU architectures may need -msse4.1/-mavx2 flags removed from CMakeLists.txt or minimap2 built with sse2only, or the build will fail.

## Evidence

- [abstract] NanoSpring, a reference-free compressor for nanopore sequencing reads, relying on an approximate assembly approach: "We present NanoSpring, a reference-free compressor for nanopore sequencing reads, relying on an approximate assembly approach."
- [readme] minhash kmer size default 23: "-k [ --kmer ] arg (=23)        kmer size for the minhash (default 23)"
- [readme] minhash overlap sketch threshold parameter: "--overlap-sketch-thr arg (=6)  the overlap sketch threshold for minhash
                                 (default 6)"
- [readme] minimap2 chaining parameter used to confirm overlaps: "--max-chain-iter arg (=400)    the max number of partial chains during
                                 chaining for minimap2 (default 400)"
- [readme] example invocation tuning the minhash parameters: "./NanoSpring -c -i file.fastq -k 20 -n 64 --overlap-sketch-thr 8 -o file.NanoSpring"
- [intro] basecaller error rate motivating approximate assembly: "the error rate has fallen dramatically in the recent years with the advent of deep learning based basecallers which achieve median error rate close to 5% or better5"
