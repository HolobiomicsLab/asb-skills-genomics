---
name: contig-orientation-inference
description: Use when you have a draft assembly FASTA plus long-read FASTA/FASTQ and
  need to scaffold contigs (order/orient them and estimate gaps) without running a
  full read alignment step.
when_to_use_negative:
- You already have high-quality full read alignments (e.g. minimap2 PAF) and an alignment-based
  scaffolder is preferred — ntLink's advantage is using lightweight minimizer-based
  mappings instead of read alignments.
- The draft contigs are shorter than z=1000 bp, since ntLink restricts scaffold graph
  nodes to contigs of at least 1000 bp.
- You need misassembly correction rather than scaffolding — that is a separate downstream
  application (Tigmint-long using ntLink mappings), not contig join inference.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0193
tools:
- name: ntLink
  role: Scaffolding tool whose minimizer-based mappings infer contig ordering, orientation,
    and gap sizes; computes minimizer sketches and builds the scaffold graph
  repo: https://github.com/bcgsc/ntLink
- name: btllib
  role: Dependency library providing ntHash2-style recursive spaced-seed hashing used
    to compute ordered minimizer sketches (btllib 1.7.1 or higher)
  repo: https://github.com/BirolLab/btllib
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:41e2a309acba122733cd84995548160894d87f0931a0bca3c6b281f79a01eb3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/ntLink
  tools:
  - btllib
  - ntLink
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0193
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# contig-orientation-inference

## Summary

Infers how draft assembly contigs should be ordered and oriented into scaffolds using ntLink's lightweight, minimizer-based mappings of long reads instead of full read alignments; the same mappings also estimate gap sizes between joined contigs.

## When to use

Use when you have a draft assembly FASTA plus long-read FASTA/FASTQ and need to scaffold contigs (order/orient them and estimate gaps) without running a full read alignment step.

## When NOT to use

- You already have high-quality full read alignments (e.g. minimap2 PAF) and an alignment-based scaffolder is preferred — ntLink's advantage is using lightweight minimizer-based mappings instead of read alignments.
- The draft contigs are shorter than z=1000 bp, since ntLink restricts scaffold graph nodes to contigs of at least 1000 bp.
- You need misassembly correction rather than scaffolding — that is a separate downstream application (Tigmint-long using ntLink mappings), not contig join inference.

## Inputs

- Draft assembly FASTA (target assembly contigs)
- Long-read sequencing data in FASTA/FASTQ (gzipped or uncompressed)
- ntLink parameters: k=32 (default), w=100 (default; w=250 in article example), n=1 minimum supported read, gap bounds g=20 bp and G=-1, z=1000 bp minimum contig length

## Outputs

- PAF-like long-read to contig mappings
- scaffold_graph.gfa (scaffold graph with oriented-contig nodes and mapping-suggested-join edges)
- Scaffolded assembly file with suffix *ntLink.scaffolds.fa

## How to apply

First compute ordered minimizer sketches of the target assembly and long reads with ntLink/btllib ntHash2-style recursive spaced-seed hashing (defaults k=32, w=100; the article example uses w=250). Map long-read minimizers to the target contigs, optionally in sensitive mode, producing PAF-like read-to-contig mappings. Scan the mappings for contig pairs whose joins are suggested by long reads spanning multiple contigs, enforcing a minimum edge weight of n=1 supporting read and gap bounds of g=20 bp minimum and G=-1 (no maximum). Build the scaffold graph whose nodes are oriented contigs restricted to at least z=1000 bp and whose edges are the mapping-suggested joins, writing scaffold_graph.gfa; the post-scaffolding output carries the suffix *ntLink.scaffolds.fa. The rationale is that long reads spanning multiple contigs supply the joining and gap-estimation evidence, avoiding computationally heavier read alignments.

## Related tools

- **ntLink** (Scaffolding tool whose minimizer-based mappings infer contig ordering, orientation, and gap sizes; computes minimizer sketches and builds the scaffold graph) — https://github.com/bcgsc/ntLink
- **btllib** (Dependency library providing ntHash2-style recursive spaced-seed hashing used to compute ordered minimizer sketches (btllib 1.7.1 or higher)) — https://github.com/BirolLab/btllib

## Examples

```
ntLink scaffold target.fa longreads.fastq w=250
```

## Evaluation signals

- Every scaffold graph edge is supported by at least the minimum number (a=1) of anchored long-read mappings.
- The PAF-like read-to-contig mappings can be regenerated from the minimizer sketches of the assembly and reads.
- scaffold_graph.gfa exists with nodes restricted to contigs of at least z=1000 bp and edges encoding mapping-suggested joins.
- The final scaffolded assembly file with suffix *ntLink.scaffolds.fa is produced and contiguity improves relative to the draft assembly (assessable with QUAST).
- Gap estimates between joined contigs fall within the enforced bounds (minimum g=20 bp; G=-1 means no maximum).

## Limitations

- Joins require long reads that span multiple contigs; with insufficient spanning long-read coverage, contig pairs cannot be linked.
- Minimum edge weight n=1 supported read is very permissive, so low-support joins may need additional validation.
- Contigs shorter than 1000 bp are excluded from the scaffold graph and thus cannot be oriented by this mechanism.

## Evidence

- [abstract] Instead of using read alignments to identify candidate joins, ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds: "Instead of using read alignments to identify candidate joins, ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds"
- [methods] 1. Compute ordered minimizer sketches of the input target assembly and long reads: "1. Compute ordered minimizer sketches of the input target assembly and long reads"
- [methods] 3. Find contig pairs, where joins are suggested by the long-read mapping evidence: "3. Find contig pairs, where joins are suggested by the long-read mapping evidence"
- [intro] ntLink is a lightweight, minimizer-based long-read scaffolding tool: "ntLink is a lightweight, minimizer-based long-read scaffolding tool"
