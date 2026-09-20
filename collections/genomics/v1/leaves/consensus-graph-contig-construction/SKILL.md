---
name: consensus-graph-contig-construction
description: Use when building contigs from long, noisy nanopore FASTQ reads without
  a reference (e.g. inside a lossless compressor), by finding candidate overlapping
  reads with MinHash sketching and aligning them to a growing consensus graph.
when_to_use_negative:
- The task requires compressing quality values or read identifiers — NanoSpring compresses
  only the base sequences of the FASTQ
- Input is short, low-error read data (e.g. Illumina) where compressors like SPRING
  or PgRC that assume short reads are appropriate
- A high-quality reference genome is available and a reference-based compressor such
  as RENANO would achieve better compression
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: NanoSpring
  role: 'Hosts the consensus-graph contig construction: MinHash sketching, consensus
    graph building, and greedy consensus path extraction during compression'
  repo: https://github.com/qm2/NanoSpring
- name: minimap2
  role: Aligns candidate overlapping reads to the current consensus sequence before
    they are added to the directed acyclic consensus graph
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:4e0e717ffd34892d7e3f1f675112d12ae8d376e2039780dd1831a09ddbd912e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - NanoSpring
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# consensus-graph-contig-construction

## Summary

Reconstructs NanoSpring's approximate-assembly stage, where contigs are built by greedily extending a directed acyclic consensus graph with minimap2-aligned reads found via MinHash overlap search, enabling reference-free lossless compression of ONT read sequences.

## When to use

Use when building contigs from long, noisy nanopore FASTQ reads without a reference (e.g. inside a lossless compressor), by finding candidate overlapping reads with MinHash sketching and aligning them to a growing consensus graph.

## When NOT to use

- The task requires compressing quality values or read identifiers — NanoSpring compresses only the base sequences of the FASTQ
- Input is short, low-error read data (e.g. Illumina) where compressors like SPRING or PgRC that assume short reads are appropriate
- A high-quality reference genome is available and a reference-based compressor such as RENANO would achieve better compression

## Inputs

- FASTQ file of nanopore (ONT) read sequences (gzipped input also supported); identifiers and quality values are not used
- Optional tuning parameters: --kmer, --num-hash, --overlap-sketch-thr, --minimap-k, --minimap-w, --max-chain-iter, --edge-thr

## Outputs

- Contigs formed as greedy highest-weight consensus paths through the directed acyclic consensus graph
- Compressed archive (e.g. file.NanoSpring, a tar archive of compressed streams) for later lossless decompression

## How to apply

Sketch each nanopore read with MinHash using the default kmer size 23, 60 hash functions, and overlap sketch threshold 6 to retrieve candidate reads overlapping the current consensus sequence of the graph. Align candidates to that consensus with minimap2 using kmer size 20, window size 50, and max-chain-iter 400, then add the aligned reads to the directed acyclic consensus graph, capping it at the default edge threshold of 4,000,000 edges. Iteratively extract the greedy highest-weight consensus path to emit contigs, since the falling basecaller error rate (~5% or better) is what makes an approximate assembly of ONT reads viable. Run compression with the default 20 threads (./NanoSpring -c -i file.fastq -o file.NanoSpring); note the tool compresses only base sequences, not identifiers or quality values.

## Related tools

- **NanoSpring** (Hosts the consensus-graph contig construction: MinHash sketching, consensus graph building, and greedy consensus path extraction during compression) — https://github.com/qm2/NanoSpring
- **minimap2** (Aligns candidate overlapping reads to the current consensus sequence before they are added to the directed acyclic consensus graph)

## Examples

```
./NanoSpring -c -i file.fastq -o file.NanoSpring
```

## Evaluation signals

- Losslessness check: cmp file.reads <(cat file.fastq | sed -n '2~4p') shows no differences between decompressed and original sequence lines
- Consensus graph stays within the --edge-thr cap (default 4,000,000 edges); contigs are emitted as greedy highest-weight consensus paths
- On recently basecalled high-quality nanopore datasets, overall compression lands around 0.35-0.65 bits per base
- MinHash overlap stage uses kmer size 23, 60 hash functions, and overlap sketch threshold 6; minimap2 stage uses k=20, w=50, max-chain-iter 400

## Limitations

- Compression ratio is not at the fundamental limits; the authors note 'Future work includes research on the assembly algorithm to get closer to the fundamental limits'
- Edge threshold (default 4,000,000 edges) bounds graph size and can be raised (e.g. 8000000) at the cost of memory
- Only base sequences are compressed; identifiers and quality values are ignored, and very low --decompression-memory settings may slightly reduce decompression speed

## Evidence

- [readme] contig build params: "-k [ --kmer ] arg (=23)        kmer size for the minhash (default 23)"
- [readme] edge threshold: "--edge-thr arg (=4000000)      the max number of edges allowed in a consensus graph (default 4000000)"
- [readme] minimap2 params: "--minimap-k arg (=20)          kmer size for the minimap2 (default 20)"
- [intro] motivation: "the error rate has fallen dramatically in the recent years with the advent of deep learning based basecallers which achieve median error rate close to 5% or better5"
- [abstract] scope: "NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base"
