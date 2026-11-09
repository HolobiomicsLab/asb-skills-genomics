---
name: minimizer-based-long-read-scaffolding
description: Use when a draft assembly (e.g. from Flye or another long-read assembler)
  has fragmented contigs and long reads (FASTQ, e.g. ONT SRR10028109) are available
  to infer contig ordering, orientation, joins, and gap-filling.
when_to_use_negative:
- The input is already a set of long-read-to-contig alignments rather than raw reads
  plus a draft assembly — instead use ntLink as a standalone mapping tool (Alternate
  Protocol 1) or Tigmint-long correction (Alternate Protocol 2)
- The goal is correcting misassemblies rather than ordering/orienting contigs — use
  the Tigmint-long correction workflow built on ntLink mappings
- No long-read data are available, since ntLink infers joins from long-read mapping
  evidence
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ntLink
  role: 'Core scaffolder: computes ordered minimizer sketches, maps long reads to
    contigs, infers joins, and scaffolds/gap-fills the draft assembly'
  repo: https://github.com/bcgsc/ntLink
- name: abyss-fac (ABySS)
  role: Computes assembly contiguity and N-content statistics (ABySS v2.3.0+) on the
    scaffolded/gap-filled FASTA
  repo: https://github.com/BirolLab/abyss
- name: QUAST
  role: 'Evaluates gap quality with --split-scaffold, reporting separate Contig- vs
    Scaffold-level NG50/NGA50 and # N''s per 100 kbp'
  repo: https://github.com/ablab/quast
- name: btllib
  role: Required library dependency (1.7.1 or higher) for running ntLink
  repo: https://github.com/BirolLab/btllib
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:5a1b5ac7adbf9cadfba7dd7672aa3282e78347b0dd94e8c7395f95f293022e4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/ntLink
  tools:
  - QUAST
  - abyss-fac (ABySS)
  - btllib
  - ntLink
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# minimizer-based-long-read-scaffolding

## Summary

Scaffold a draft genome assembly using ntLink's lightweight minimizer-based long-read mappings rather than full read alignments, optionally with overlap detection, gap-filling, and in-code scaffolding iterations, to produce highly contiguous scaffolds while remaining computationally efficient.

## When to use

Use when a draft assembly (e.g. from Flye or another long-read assembler) has fragmented contigs and long reads (FASTQ, e.g. ONT SRR10028109) are available to infer contig ordering, orientation, joins, and gap-filling.

## When NOT to use

- The input is already a set of long-read-to-contig alignments rather than raw reads plus a draft assembly — instead use ntLink as a standalone mapping tool (Alternate Protocol 1) or Tigmint-long correction (Alternate Protocol 2)
- The goal is correcting misassemblies rather than ordering/orienting contigs — use the Tigmint-long correction workflow built on ntLink mappings
- No long-read data are available, since ntLink infers joins from long-read mapping evidence

## Inputs

- Draft assembly FASTA of contigs (e.g. celegans_flye.fa from Flye)
- Long-read sequencing reads in FASTQ (e.g. ONT reads SRR10028109.fq.gz)
- ntLink parameters such as k (e.g. 32), w (e.g. 100), and thread count t (e.g. 5); overlap=True for gap-filling

## Outputs

- Scaffolded (and optionally gap-filled) assembly FASTA
- Scaffold graph with oriented contig nodes and long-read-supported join edges
- Assembly statistics from abyss-fac (contiguity, N-content)
- QUAST --split-scaffold report with Contig vs Scaffold NG50/NGA50 and # N's per 100 kbp

## How to apply

Run ntLink in the mode matching the goal: Basic Protocol 1 uses overlap detection, Basic Protocol 2 adds gap-filling (requiring overlap=True so raw read sequence fills scaffold gaps, e.g. k=32 w=100 t=5 on C. elegans ONT data), and Basic Protocol 3 runs in-code scaffolding iterations with coordinate liftover. Internally, ntLink computes ordered minimizer sketches of the target assembly and long reads, uses the minimizers to map reads to contigs, finds contig pairs whose joins are suggested by the long-read mapping evidence, and outputs a scaffold graph whose nodes are oriented contigs and edges are joins. Assess the result with abyss-fac (ABySS v2.3.0+) for contiguity and N-content, and with QUAST --split-scaffold to separate Contig-level vs Scaffold-level NG50/NGA50 and count # N's per 100 kbp; efficient gap-filling makes Contig NG50/NGA50 approach Scaffold statistics while N content decreases.

## Related tools

- **ntLink** (Core scaffolder: computes ordered minimizer sketches, maps long reads to contigs, infers joins, and scaffolds/gap-fills the draft assembly) — https://github.com/bcgsc/ntLink
- **abyss-fac (ABySS)** (Computes assembly contiguity and N-content statistics (ABySS v2.3.0+) on the scaffolded/gap-filled FASTA) — https://github.com/BirolLab/abyss
- **QUAST** (Evaluates gap quality with --split-scaffold, reporting separate Contig- vs Scaffold-level NG50/NGA50 and # N's per 100 kbp) — https://github.com/ablab/quast
- **btllib** (Required library dependency (1.7.1 or higher) for running ntLink) — https://github.com/BirolLab/btllib

## Examples

```
ntLink scaffold gap_fill target=celegans_flye.fa reads=SRR10028109.fq.gz k=32 w=100 t=5
```

## Evaluation signals

- The gap-filled FASTA exists and parses as valid FASTA
- abyss-fac statistics on the output reproduce reported contiguity and N-content values (e.g. Table 4)
- QUAST --split-scaffold shows Contig-level NG50/NGA50 approaching Scaffold-level statistics after gap-filling
- # N's per 100 kbp decreases relative to the pre-gap-fill assembly while NG50 is maintained
- The scaffold graph is produced with oriented contig nodes and long-read-supported join edges

## Limitations

- Gap-filling requires the overlap=True parameter; omitting it prevents raw read sequence from being inserted into scaffold gaps
- Joins depend on long-read mapping evidence; contigs without sufficient linking reads will not be scaffolded
- Reported statistics (e.g. Table 4) are dataset-specific (C. elegans ONT, SRR10028109) and must be recomputed for other genomes

## Evidence

- [abstract] minimizer mappings rather than read alignments: "Instead of using read alignments to identify candidate joins, ntLink utilizes minimizer-based mappings to infer how input sequences should be ordered and oriented into scaffolds"
- [abstract] new features: "Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations"
- [methods] workflow steps: "1. Compute ordered minimizer sketches of the input target assembly and long reads"
- [methods] scaffold graph: "4. Output a scaffold graph, where the nodes are oriented contigs and the edges are joins suggested by the long read data"
- [full_text] gap-filling command: [evidence span withheld — anchor status: not_found]
- [results] gap quality assessment: [evidence span withheld — anchor status: not_found]
