---
name: scaffold-graph-traversal
description: Use when a draft assembly FASTA (e.g. a Flye assembly) plus long-read
  FASTQ (e.g. ONT SRR10028109) are available and mapping-supported contig joins have
  been inferred, and the scaffold graph must be traversed to produce ordered, oriented
  scaffold sequences.
when_to_use_negative:
- The input is already a scaffolded assembly with no unjoined contigs to merge — traversal
  cannot create new joins without graph edges.
- Only short paired-end reads are available — ntLink requires long-read data; short-read
  scaffolders are the appropriate choice.
- The goal is assembly correction of misassemblies rather than joining contigs — Tigmint-long
  with ntLink mappings (Alternate Protocol 2) is the intended downstream application.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ntLink
  role: Computes minimizer sketches, maps long reads to contigs, infers joins and
    gap sizes, and writes the scaffold graph that is then traversed; overlap=True
    step trims overlapping joined sequences.
  repo: https://github.com/bcgsc/ntLink
- name: ABySS
  role: abyss-scaffold (ABySS v2.3.0+) traverses the ntLink scaffold graph to produce
    the merged scaffold sequences.
  repo: https://github.com/BirolLab/abyss
- name: btllib
  role: Provides ntHash2-style recursive spaced-seed hashing used to compute the ordered
    minimizer sketches underlying the mappings.
  repo: https://github.com/BirolLab/btllib
- name: QUAST
  role: Aligns the resulting scaffolds against the reference (e.g. C. elegans GCA_000002985.3)
    to report NG50, genome fraction, and misassembly counts for validation.
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:9476e7d10c3acb760a70fa9caa3a941b24529d41551ce4eaa8b9d91685239c8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/ntLink
  tools:
  - ABySS
  - QUAST
  - btllib
  - ntLink
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# scaffold-graph-traversal

## Summary

Traverse a long-read-derived scaffold graph — whose nodes are oriented contigs and whose edges are minimizer-mapping-supported joins — using abyss-scaffold (ABySS v2.3.0+) as integrated into ntLink, to merge draft contigs into scaffolds and optionally trim overlaps at joins.

## When to use

Use when a draft assembly FASTA (e.g. a Flye assembly) plus long-read FASTQ (e.g. ONT SRR10028109) are available and mapping-supported contig joins have been inferred, and the scaffold graph must be traversed to produce ordered, oriented scaffold sequences.

## When NOT to use

- The input is already a scaffolded assembly with no unjoined contigs to merge — traversal cannot create new joins without graph edges.
- Only short paired-end reads are available — ntLink requires long-read data; short-read scaffolders are the appropriate choice.
- The goal is assembly correction of misassemblies rather than joining contigs — Tigmint-long with ntLink mappings (Alternate Protocol 2) is the intended downstream application.

## Inputs

- Draft assembly FASTA (e.g. celegans_flye.fa, Flye draft of C. elegans)
- Long-read FASTA/FASTQ, gzipped or plain (e.g. SRR10028109.fq.gz ONT reads)
- Scaffold graph in GFA format (scaffold_graph.gfa) produced by ntLink, with nodes as oriented contigs and edges as mapping-supported joins

## Outputs

- Scaffolded assembly FASTA with suffix *ntLink.scaffolds.fa (e.g. celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa)
- Intermediate ntLink outputs: read-to-contig mappings in PAF-like format and scaffold_graph.gfa

## How to apply

First build the scaffold graph with ntLink: compute ordered minimizer sketches of the target assembly and reads with btllib ntHash2-style hashing (k=32, w=100 in the protocol example), map read minimizers to contigs, and emit scaffold_graph.gfa where nodes are oriented contigs of at least z=1000 bp and edges are joins supported by at least n=1 read (gap bounds g=20 bp minimum, G=-1 for no maximum). Then hand the graph to abyss-scaffold (requires ABySS v2.3.0+) to traverse paths through the graph and output merged scaffold sequences. Enable the overlap=True step so overlapping joined sequences are identified and trimmed at scaffold joins rather than naively concatenated with estimated gaps. The rationale is that minimizer-based mappings of reads spanning multiple contigs supply both the join evidence and the gap-size estimates, avoiding full read alignments.

## Related tools

- **ntLink** (Computes minimizer sketches, maps long reads to contigs, infers joins and gap sizes, and writes the scaffold graph that is then traversed; overlap=True step trims overlapping joined sequences.) — https://github.com/bcgsc/ntLink
- **ABySS** (abyss-scaffold (ABySS v2.3.0+) traverses the ntLink scaffold graph to produce the merged scaffold sequences.) — https://github.com/BirolLab/abyss
- **btllib** (Provides ntHash2-style recursive spaced-seed hashing used to compute the ordered minimizer sketches underlying the mappings.) — https://github.com/BirolLab/btllib
- **QUAST** (Aligns the resulting scaffolds against the reference (e.g. C. elegans GCA_000002985.3) to report NG50, genome fraction, and misassembly counts for validation.)

## Examples

```
ntLink scaffold target=celegans_flye.fa reads=SRR10028109.fq.gz k=32 w=100 t=5 overlap=True
```

## Evaluation signals

- The scaffold FASTA exists and carries the *ntLink.scaffolds.fa suffix.
- abyss-fac on draft vs. scaffolds shows NG50/N50 increase and number of sequences decrease after traversal.
- QUAST against the reference shows increased NG50 and genome fraction without a large increase in misassemblies, matching Table 3 values for the ntLink-scaffolded Flye C. elegans assembly.
- Every scaffold graph edge is supported by at least a=1 anchored long-read mapping.
- The PAF-like read-to-contig mappings can be regenerated from the sketches.

## Limitations

- Joins are restricted to contigs of at least z=1000 bp; shorter contigs are excluded from the graph.
- Gap size estimates come from minimizer-based mappings; a minimum gap of g=20 bp is enforced and G=-1 (no maximum) by default, so unrealistic gaps can pass unless bounds are tightened.
- Without overlap detection (overlap=True), overlapping adjacent contigs joined by traversal would be duplicated at scaffold joins rather than trimmed.

## Evidence

- [abstract] Instead of using read alignments to identify candidate joins, ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds: "Instead of using read alignments to identify candidate joins, ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds"
- [methods] scaffold graph definition: "4. Output a scaffold graph, where the nodes are oriented contigs and the edges are joins suggested by the long read data"
- [abstract] overlap feature: "Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations"
