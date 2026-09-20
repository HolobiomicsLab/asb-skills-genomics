---
name: minimap2-read-alignment
description: Use when building contigs from overlapping nanopore reads without a reference
  genome — e.g., during approximate-assembly compression of FASTQ base sequences —
  and candidate overlaps must be precisely aligned before graph edge insertion.
when_to_use_negative:
- The data are short, low-error reads — compressors like SPRING and PgRC operate under
  that assumption instead
- A reference-based compressor is required (e.g., RENANO) — NanoSpring's minimap2
  alignments target an internally built consensus, not an external reference
- Quality values or read identifiers need compressing — NanoSpring compresses only
  the base sequences
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0080
tools:
- name: minimap2
  role: Aligns candidate overlapping reads to the current consensus sequence of the
    consensus graph before edge insertion
  repo: https://github.com/qm2/NanoSpring
- name: NanoSpring
  role: Orchestrates MinHash sketching, consensus-graph construction, contig extraction,
    and final compression of the FASTQ base sequences
  repo: https://github.com/qm2/NanoSpring
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:37f5eb333c1140f0ac3047e962ca758db263850619f32c5744bd16f9595feacd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - NanoSpring
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# minimap2-read-alignment

## Summary

Aligns candidate overlapping nanopore reads onto the current consensus sequence of a directed acyclic graph using minimap2, the step within NanoSpring's approximate-assembly compression that adds read edges to the consensus graph from which contigs are greedily extracted.

## When to use

Use when building contigs from overlapping nanopore reads without a reference genome — e.g., during approximate-assembly compression of FASTQ base sequences — and candidate overlaps must be precisely aligned before graph edge insertion.

## When NOT to use

- The data are short, low-error reads — compressors like SPRING and PgRC operate under that assumption instead
- A reference-based compressor is required (e.g., RENANO) — NanoSpring's minimap2 alignments target an internally built consensus, not an external reference
- Quality values or read identifiers need compressing — NanoSpring compresses only the base sequences

## Inputs

- FASTQ file of nanopore read sequences (gzipped input also supported)
- Candidate overlapping reads identified via MinHash sketching
- Current consensus sequence of the directed acyclic consensus graph

## Outputs

- Aligned read-to-consensus mappings inserted as edges in the consensus graph
- Contigs built from the greedy highest-weight consensus path
- Compressed .NanoSpring archive (tar of compressed streams)

## How to apply

First sketch the FASTQ reads with MinHash (default kmer size 23, 60 hash functions, overlap sketch threshold 6) to find candidate overlapping reads for contig extension. Then align each candidate read to the current consensus sequence of the graph with minimap2, using kmer size 20, window size 50, and max-chain-iter 400 (override via --minimap-k, --minimap-w, --max-chain-iter). Insert the aligned reads into the directed acyclic consensus graph, capping the graph at an edge threshold of 4,000,000 edges (--edge-thr), and extract the greedy highest-weight consensus path to build contigs. The rationale: nanopore basecaller error rates near 5% or better make approximate assembly of overlapping reads feasible, and the resulting contigs serve as the basis for NanoSpring's reference-free compression of base sequences only.

## Related tools

- **minimap2** (Aligns candidate overlapping reads to the current consensus sequence of the consensus graph before edge insertion) — https://github.com/qm2/NanoSpring
- **NanoSpring** (Orchestrates MinHash sketching, consensus-graph construction, contig extraction, and final compression of the FASTQ base sequences) — https://github.com/qm2/NanoSpring

## Examples

```
./NanoSpring -c -i file.fastq --minimap-k 25 --minimap-w 100 -o file.NanoSpring
```

## Evaluation signals

- minimap2 parameters match the configured values (default minimap-k=20, minimap-w=50, max-chain-iter=400) as echoed in the CLI options
- The consensus graph never exceeds the edge threshold (default 4,000,000 edges via --edge-thr)
- Round trip is lossless: `cmp file.reads <(cat file.fastq | sed -n '2~4p')` reports no differences between decompressed reads and original sequence lines
- Contigs are recovered as greedy highest-weight paths from the directed acyclic consensus graph, confirming alignment-driven edge insertion succeeded
- Installation validity is confirmed by running the test script from the build directory

## Limitations

- Older CPU architectures may need the -msse4.1 flag removed from target_compile_options, adjusted -mavx2 flags for libbsc, or minimap2 compiled with `make sse2only=1`
- Very low --decompression-memory settings can slightly reduce decompression speed
- Compression ratio is not yet at fundamental limits; the paper's future work is improving the assembly algorithm

## Evidence

- [readme] minimap2 alignment parameters: "--minimap-k arg (=20)          kmer size for the minimap2 (default 20)"
- [readme] minimap2 window override example: "Compressing with kmer size of minimap2 to be 25 and the window size for the minimap2 to be 100."
- [intro] minimap2 as downstream tool ignoring quality scores: "since quality scores are often ignored by downstream tools like minimap2"
- [abstract] approximate assembly context: "We present NanoSpring, a reference-free compressor for nanopore sequencing reads, relying on an approximate assembly approach."
