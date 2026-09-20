---
name: minimizer-sketch-computation
description: Use when you have a draft genome assembly FASTA plus long-read sequencing
  data and need fast, alignment-free read-to-contig mappings (PAF-like) to infer contig
  ordering/orientation, gap sizes, or misassemblies.
when_to_use_negative:
- You already have accurate full read alignments (e.g. minimap2 PAF/BAM) and alignment-based
  scaffolding evidence is acceptable — minimizer mappings are a lightweight substitute,
  not a richer signal
- Input is short-read sequencing data — the method's joins rely on long reads spanning
  multiple contigs
- You need base-level alignment detail (e.g. variant calling) rather than contig-level
  mapping evidence
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: ntLink
  role: computes the ordered minimizer sketches, performs the minimizer-based mapping
    to contigs (PAF-like output), and builds the scaffold graph
  repo: https://github.com/bcgsc/ntLink
- name: btllib
  role: provides ntHash2-style recursive spaced-seed hashing underlying the minimizer
    sketches (v1.7.1 or higher)
  repo: https://github.com/BirolLab/btllib
- name: Tigmint-long
  role: downstream consumer of ntLink minimizer-based mappings for genome assembly
    correction
  repo: https://github.com/BirolLab/tigmint
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:261e55e2fff0e0fffae2edab135d6503aa03f2c7fc3de1eb454bcee32ab20e5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/ntLink
  tools:
  - Tigmint-long
  - btllib
  - ntLink
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# minimizer-sketch-computation

## Summary

Compute ordered minimizer sketches of a target draft assembly (FASTA) and long reads (FASTA/FASTQ) and use them to produce lightweight minimizer-based read-to-contig mappings, as implemented in ntLink with btllib ntHash2-style recursive spaced-seed hashing. These mappings replace read alignments as the evidence for scaffolding joins, gap estimates, and downstream applications such as misassembly detection.

## When to use

Use when you have a draft genome assembly FASTA plus long-read sequencing data and need fast, alignment-free read-to-contig mappings (PAF-like) to infer contig ordering/orientation, gap sizes, or misassemblies.

## When NOT to use

- You already have accurate full read alignments (e.g. minimap2 PAF/BAM) and alignment-based scaffolding evidence is acceptable — minimizer mappings are a lightweight substitute, not a richer signal
- Input is short-read sequencing data — the method's joins rely on long reads spanning multiple contigs
- You need base-level alignment detail (e.g. variant calling) rather than contig-level mapping evidence

## Inputs

- Draft genome assembly FASTA (target contigs)
- Long-read sequencing reads in FASTA or FASTQ (gzipped or uncompressed)
- Parameter choices: k-mer size k (default 32), window size w (default 100; article example w=250), minimum contig length z=1000, min supported reads n=1, gap bounds g=20 and G=-1

## Outputs

- Ordered minimizer sketches of the assembly contigs and long reads
- Long-read to contig mappings in PAF-like format
- Scaffold graph in GFA format (scaffold_graph.gfa) with oriented contigs as nodes and mapping-suggested joins as edges
- Scaffolded assembly (suffix *ntLink.scaffolds.fa) when run through the full pipeline

## How to apply

Load the target assembly FASTA and long-read FASTA/FASTQ (gzipped or plain) and compute ordered minimizer sketches of both using btllib ntHash2-style recursive spaced-seed hashing, with defaults k=32 and window w=100 (the article example uses w=250). Use the minimizers to map long reads to the target assembly contigs with ntLink's lightweight minimizer-based mapping, optionally in sensitive mode, producing read-to-contig mappings in PAF-like format. Scan the mappings for contig pairs whose joins are suggested by long reads spanning multiple contigs, enforcing the minimum edge weight n=1 supporting read and gap bounds g=20 bp (minimum) and G=-1 (no maximum), and restricting nodes to contigs of at least z=1000 bp. Build the scaffold graph with oriented contigs as nodes and mapping-suggested joins as edges (scaffold_graph.gfa), and validate that every edge is supported by at least a=1 anchored long-read mappings. The rationale is that minimizer mappings carry the same long-range contiguity evidence as full alignments at a fraction of the compute, and the same mappings feed gap-size estimation.

## Related tools

- **ntLink** (computes the ordered minimizer sketches, performs the minimizer-based mapping to contigs (PAF-like output), and builds the scaffold graph) — https://github.com/bcgsc/ntLink
- **btllib** (provides ntHash2-style recursive spaced-seed hashing underlying the minimizer sketches (v1.7.1 or higher)) — https://github.com/BirolLab/btllib
- **Tigmint-long** (downstream consumer of ntLink minimizer-based mappings for genome assembly correction) — https://github.com/BirolLab/tigmint

## Evaluation signals

- Every edge in scaffold_graph.gfa is supported by at least a=1 anchored long-read mapping
- The PAF-like read-to-contig mappings can be regenerated reproducibly from the same sketches and parameters
- Each scaffold graph edge is backed by reads spanning multiple contigs, and estimated gaps fall within the enforced bounds (g=20 bp minimum, G=-1 no maximum)
- The final scaffolded output carries the expected suffix *ntLink.scaffolds.fa
- All scaffold graph nodes correspond to oriented contigs of length >= z=1000 bp

## Limitations

- Joins are inferred from minimizer mappings rather than read alignments, so edge confidence is bounded by the minimum supported read count (n=1) and anchored mapping support (a=1)
- Contigs shorter than z=1000 bp are excluded from the scaffold graph
- ntLink assumes long-read data; the same long reads used to build the assembly may be used, but mapping quality depends on window size choices (k=32, w=100 default; article example w=250)

## Evidence

- [abstract] Instead of using read alignments to identify candidate joins, ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds: "Instead of using read alignments to identify candidate joins, ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds"
- [methods] 1. Compute ordered minimizer sketches of the input target assembly and long reads: "1. Compute ordered minimizer sketches of the input target assembly and long reads"
- [methods] 2. Use minimizers to map the long reads to the target assembly contigs: "2. Use minimizers to map the long reads to the target assembly contigs"
- [abstract] the lightweight minimizer-based mappings that enable ntLink scaffolding can also be utilized for other downstream applications, such as misassembly detection: "the lightweight minimizer-based mappings that enable ntLink scaffolding can also be utilized for other downstream applications, such as misassembly detection"
