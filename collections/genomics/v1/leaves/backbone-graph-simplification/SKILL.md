---
name: backbone-graph-simplification
description: 'Use when reconstructing HASLR''s hybrid assembly pipeline: you have
  short-read contigs aligned to error-prone long reads (PacBio/ONT) and need to build
  the initial backbone graph from unique SR contigs and simplify it.'
when_to_use_negative:
- When your SR contigs are heavily fragmented (e.g., high-heterozygosity regions where
  Minia contigs are filtered out), the backbone graph will lack unique contigs and
  fragment into multiple connected components.
- When assembling a diploid genome requiring phasing — HASLR's current version does
  not generate a phased assembly.
- When you need an OLC-style assembly of long reads — unused long reads are not assembled
  in the backbone step (e.g., via miniasm) in the current version.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: HASLR
  role: Hybrid assembler that builds and simplifies the backbone graph from unique
    SR contigs and long-read alignments
  repo: https://github.com/vpc-ccg/haslr
- name: minimap2
  role: Aligns short read contigs onto long reads to derive graph edge connections
  repo: https://github.com/lh3/minimap2
- name: Minia
  role: Assembles the Illumina short reads into contigs that become the unique graph
    anchors
  repo: https://github.com/GATB/minia
- name: fastutils
  role: FASTA/Q manipulation for preparing SR contig and long read subsets
  repo: https://github.com/haghshenas/fastutils
- name: Bandage
  role: Visualizes backbone.06.smallbubble.gfa to verify graph topology
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:6d63fb8fd1d187b6f17dd01fabe65ae76bda426cc5d5196bc1d9fc1654c35dcb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/vpc-ccg/haslr
  repo_url: https://github.com/vpc-ccg/haslr
  tools:
  - Bandage
  - HASLR
  - Minia
  - fastutils
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# backbone-graph-simplification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs and progressively simplifies HASLR's backbone graph, which models connections between unique short-read (SR) contigs via their alignments onto long reads, to yield a final scaffold topology (backbone.06.smallbubble.gfa).

## When to use

Use when reconstructing HASLR's hybrid assembly pipeline: you have short-read contigs aligned to error-prone long reads (PacBio/ONT) and need to build the initial backbone graph from unique SR contigs and simplify it.

## When NOT to use

- When your SR contigs are heavily fragmented (e.g., high-heterozygosity regions where Minia contigs are filtered out), the backbone graph will lack unique contigs and fragment into multiple connected components.
- When assembling a diploid genome requiring phasing — HASLR's current version does not generate a phased assembly.
- When you need an OLC-style assembly of long reads — unused long reads are not assembled in the backbone step (e.g., via miniasm) in the current version.

## Inputs

- sr_k49_a3.contigs.nooverlap.250.fa (short read contigs at least 250 bp, overlap-free, from Minia k=49, min abundance 3)
- lr25x.fasta (longest 25x coverage of long reads)
- minimap2 alignments of SR contigs onto long reads

## Outputs

- backbone.01.init.gfa (initial backbone graph from unique SR contigs)
- backbone.02.weakEdge.gfa (graph after removing weak edges)
- backbone.03.tip.gfa (graph after tip removal)
- backbone.04.simplebubble.gfa (graph after simple bubble removal)
- backbone.05.superbubble.gfa (graph after super bubble removal)
- backbone.06.smallbubble.gfa (final simplified backbone graph)

## How to apply

First align short-read contigs onto the selected long reads (lr25x.fasta) with minimap2, retaining only alignment blocks of at least 500 bp (--aln-block 500) with similarity at least 0.85 (--aln-sim 0.85). Build the initial graph (backbone.01.init.gfa) using only unique SR contigs — those likely present once in the genome — ordered by their long-read alignments, requiring at least 3 long reads to support each edge (--edge-sup 3). Then simplify sequentially: remove weak edges (backbone.02.weakEdge.gfa), tips (backbone.03.tip.gfa), simple bubbles (backbone.04.simplebubble.gfa), super bubbles (backbone.05.superbubble.gfa), and small bubbles (backbone.06.smallbubble.gfa). Rationale: connectivity depends on unique SR contigs, so filtering repeated contigs and weakly supported edges produces a reliable backbone; the sequence of graph-cleaning steps removes assembly artifacts before consensus calling between anchors.

## Related tools

- **HASLR** (Hybrid assembler that builds and simplifies the backbone graph from unique SR contigs and long-read alignments) — https://github.com/vpc-ccg/haslr
- **minimap2** (Aligns short read contigs onto long reads to derive graph edge connections) — https://github.com/lh3/minimap2
- **Minia** (Assembles the Illumina short reads into contigs that become the unique graph anchors) — https://github.com/GATB/minia
- **fastutils** (FASTA/Q manipulation for preparing SR contig and long read subsets) — https://github.com/haghshenas/fastutils
- **Bandage** (Visualizes backbone.06.smallbubble.gfa to verify graph topology) — https://github.com/rrwick/Bandage

## Examples

```
python haslr.py -x pacbio --cov-lr 25 --aln-block 500 --aln-sim 0.85 --edge-sup 3 -l lr.fastq -s sr_1.fastq sr_2.fastq -o out
```

## Evaluation signals

- backbone.06.smallbubble.gfa exists and parses as valid GFA format.
- Exported Bandage image of backbone.06.smallbubble.gfa shows a connected graph whose components are consistent with the S. cerevisiae reference chromosome structure.
- Every edge in the initial graph (backbone.01.init.gfa) is supported by at least 3 long reads (--edge-sup 3).
- All retained SR contig alignments onto long reads have blocks ≥ 500 bp with similarity ≥ 0.85.
- Nodes in the backbone correspond to unique SR contigs (≥ 250 bp, overlap-free); repeated contigs are absent.

## Limitations

- Backbone graph connectivity depends on the existence of unique SR contigs; regions lacking them yield multiple connected components.
- High heterozygosity regions are not resolved because Minia short read contigs are fragmented and filtered out.
- Longer long reads generate a more connected and resolved backbone graph; short read length limits contiguity.

## Evidence

- [methods] Backbone graph uses only unique SR contigs: [evidence span withheld — anchor status: not_found]
