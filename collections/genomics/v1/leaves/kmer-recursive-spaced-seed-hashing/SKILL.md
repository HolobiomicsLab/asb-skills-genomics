---
name: kmer-recursive-spaced-seed-hashing
description: Use when long-read FASTA/FASTQ and a draft assembly need lightweight,
  alignment-free read-to-contig mappings — e.g., to infer contig order/orientation
  for scaffolding or to detect misassemblies.
when_to_use_negative:
- When full base-level read alignments with per-base identity/accuracy are required
  (e.g., variant calling or precise alignment QC) — minimizer-based mappings are lightweight
  and approximate
- When the input draft contigs are shorter than the minimum length threshold (z=1000
  bp), since such contigs are excluded from the scaffold graph
- When the task is de novo assembly itself rather than scaffolding/correction of an
  existing draft — ntLink is a scaffolding/mapping tool, not an assembler (assemblers
  like Flye or ABySS do that)
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: ntLink
  role: 'Primary tool: computes ordered minimizer sketches and converts them into
    PAF-like read-to-contig mappings that drive scaffold join inference and gap estimation'
  repo: https://github.com/bcgsc/ntLink
- name: btllib
  role: Library dependency providing ntHash2-style recursive spaced-seed hashing used
    for the minimizer sketches (requires btllib 1.7.1 or higher)
  repo: https://github.com/BirolLab/btllib
- name: Tigmint-long
  role: Downstream consumer of the ntLink minimizer-based mappings for genome assembly
    correction (Alternate Protocol 2)
  repo: https://github.com/BirolLab/tigmint
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:84ba46fad0d73c304e6d8ae1cbc1845ac8aff1c12b94cd81ccdc1e558fc48f23
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
yes_rate: 0.8
n_claims: 15
---

# k-mer recursive spaced-seed hashing

## Summary

Computes ordered minimizer sketches of genome sequences (target assembly contigs and long reads) using btllib ntHash2-style recursive spaced-seed hashing (defaults k=32, w=100), enabling lightweight minimizer-based mappings instead of full read alignments for ntLink scaffolding.

## When to use

Use when long-read FASTA/FASTQ and a draft assembly need lightweight, alignment-free read-to-contig mappings — e.g., to infer contig order/orientation for scaffolding or to detect misassemblies.

## When NOT to use

- When full base-level read alignments with per-base identity/accuracy are required (e.g., variant calling or precise alignment QC) — minimizer-based mappings are lightweight and approximate
- When the input draft contigs are shorter than the minimum length threshold (z=1000 bp), since such contigs are excluded from the scaffold graph
- When the task is de novo assembly itself rather than scaffolding/correction of an existing draft — ntLink is a scaffolding/mapping tool, not an assembler (assemblers like Flye or ABySS do that)

## Inputs

- Draft assembly FASTA (target sequences)
- Long-read FASTA/FASTQ (gzipped or uncompressed)
- ntLink parameters: k-mer size k (default 32), window size w (default 100; article example w=250), minimum edge weight n (default 1), gap bounds g=20 and G=-1, minimum contig length z=1000

## Outputs

- Ordered minimizer sketches of the assembly and long reads
- Read-to-contig mappings in PAF-like format
- Scaffold graph GFA (scaffold_graph.gfa) with oriented contig nodes and mapping-supported edges
- Scaffolded assembly file with suffix *ntLink.scaffolds.fa

## How to apply

Load the target assembly FASTA and long-read FASTA/FASTQ (gzipped or plain) and compute ordered minimizer sketches of both using btllib ntHash2-style recursive spaced-seed hashing, with k-mer size k=32 and window size w=100 as defaults (the article example uses w=250). Use these ordered minimizer sketches in ntLink's lightweight minimizer-based mapping to map long-read minimizers to the target assembly contigs, optionally in sensitive mode, producing PAF-like read-to-contig mappings. The rationale is that minimizer-based mappings are far cheaper than read alignments yet still identify long reads spanning multiple contigs, which supply the evidence for scaffold joins and gap-size estimates; ntLink scans these mappings for contig pairs with supported joins (minimum edge weight n=1 read, gap bounds g=20 bp minimum and G=-1/no maximum), builds a scaffold graph restricted to contigs of at least z=1000 bp, and writes scaffold_graph.gfa. Recent ntLink versions extend this hashing backbone with overlap detection, gap-filling, and in-code scaffolding iterations.

## Related tools

- **ntLink** (Primary tool: computes ordered minimizer sketches and converts them into PAF-like read-to-contig mappings that drive scaffold join inference and gap estimation) — https://github.com/bcgsc/ntLink
- **btllib** (Library dependency providing ntHash2-style recursive spaced-seed hashing used for the minimizer sketches (requires btllib 1.7.1 or higher)) — https://github.com/BirolLab/btllib
- **Tigmint-long** (Downstream consumer of the ntLink minimizer-based mappings for genome assembly correction (Alternate Protocol 2)) — https://github.com/BirolLab/tigmint

## Evaluation signals

- Every scaffold graph edge is supported by at least the minimum number (a=1) of anchored long-read mappings
- The PAF-like read-to-contig mappings can be regenerated reproducibly from the same sketches/parameters
- The scaffold graph is emitted as scaffold_graph.gfa with nodes restricted to oriented contigs of at least 1000 bp
- The post-scaffolding output exists with the expected suffix *ntLink.scaffolds.fa
- Gap estimates for joins fall within the enforced bounds (minimum gap g=20 bp; G=-1 means no maximum)

## Limitations

- Mappings are minimizer-based rather than full alignments, so they do not provide base-level alignment detail
- Contigs shorter than z=1000 bp are excluded from the scaffold graph
- Minimum supported read threshold defaults to n=1, so a single spanning read can create a graph edge that may require further validation

## Evidence

- [abstract] Instead of using read alignments to identify candidate joins, ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds: "Instead of using read alignments to identify candidate joins, ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds"
- [methods] 1. Compute ordered minimizer sketches of the input target assembly and long reads: "1. Compute ordered minimizer sketches of the input target assembly and long reads"
- [intro] ntLink is a lightweight, minimizer-based long-read scaffolding tool: "ntLink is a lightweight, minimizer-based long-read scaffolding tool"
- [abstract] the lightweight minimizer-based mappings that enable ntLink scaffolding can also be utilized for other downstream applications, such as misassembly detection: "the lightweight minimizer-based mappings that enable ntLink scaffolding can also be utilized for other downstream applications, such as misassembly detection"
