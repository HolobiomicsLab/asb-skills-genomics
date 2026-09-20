---
name: nanopore-read-anchor-detection
description: Use when a pipeline must convert coarse seed-hit clusters from a seeding
  stage into precise read/reference coordinate anchors prior to anchor chaining (LCSk),
  specifically for MinION nanopore or other long error-prone reads.
when_to_use_negative:
- Reads are short Illumina reads — GraphMap's five-stage funnel and graph traversal
  target long, error-prone third-generation reads even though defaults can handle
  Illumina
- The task is full end-to-end alignment/variant calling rather than producing intermediate
  anchors — downstream stages (LCSk chaining, alignment) are needed
- Reference-scale memory is unavailable — index construction peaks around ~42 GB for
  a human genome (~14 x reference size final index), so very constrained-memory environments
  are unsuitable
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: GraphMap
  role: Implements stage II graph-based vertex-centric seed refinement and anchor
    construction; built with GCC/G++ via make modules && make
  repo: https://github.com/isovic/graphmap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:11f2c23262004b07c0570a2e4927dedebb6d7bca3b100a802919f8c6e5d5266b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - GCC/G++
  - GraphMap
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# nanopore-read-anchor-detection

## Summary

Reconstructs GraphMap's stage II, which refines stage I clustered seed hits via graph-based vertex-centric seed processing (k-mer mapping graph with l=9 outbound edges per vertex) to construct alignment anchors for long, error-prone nanopore reads.

## When to use

Use when a pipeline must convert coarse seed-hit clusters from a seeding stage into precise read/reference coordinate anchors prior to anchor chaining (LCSk), specifically for MinION nanopore or other long error-prone reads.

## When NOT to use

- Reads are short Illumina reads — GraphMap's five-stage funnel and graph traversal target long, error-prone third-generation reads even though defaults can handle Illumina
- The task is full end-to-end alignment/variant calling rather than producing intermediate anchors — downstream stages (LCSk chaining, alignment) are needed
- Reference-scale memory is unavailable — index construction peaks around ~42 GB for a human genome (~14 x reference size final index), so very constrained-memory environments are unsuitable

## Inputs

- Reference genome FASTA (already indexed by GraphMap's hash/minimizer index)
- Stage I candidate regions / clustered seed hits from upstream seeding task
- Read sequences in FASTA/FASTQ (Oxford Nanopore MinION 1d/2d or PacBio)

## Outputs

- anchors.tsv: anchor list with read coordinates and reference coordinates
- Set of filtered graph walks (≥12 bases) usable as anchors for LCSk chaining

## How to apply

Load the reference genome and the stage I candidate regions, applying the seed-hit percentile filter (--freq-percentile 0.99, which skips the 1% most repetitive seed lookups to keep runtime tractable on large references). Seed each read with the default k-mer length T=6, restricting seed hits to the candidate regions. Build a k-mer mapping graph with one vertex per k-mer occurrence, at most l=9 outbound edges per vertex, keeping identical k-mers as distinct vertices so repetitive loci retain separate paths; rationale is that distinct vertices preserve repetitive-copy resolution (GraphMap distinguishes candidates >94% identical). Traverse the graph with the O(1) vertex-update walk, greedily extending candidate paths from seed vertices without recomputing vertex neighbourhoods, then filter completed walks to those spanning at least 12 bases. Record surviving walk positions with read and reference coordinates as anchors (e.g. anchors.tsv) for the stage III anchor-chaining (LCSk) step.

## Related tools

- **GraphMap** (Implements stage II graph-based vertex-centric seed refinement and anchor construction; built with GCC/G++ via make modules && make) — https://github.com/isovic/graphmap

## Examples

```
git clone https://github.com/isovic/graphmap.git; cd graphmap; make modules; make; ./bin/Linux-x64/graphmap align -r reference.fa -d reads.fastq -o out.sam
```

## Evaluation signals

- All recorded anchors span at least 12 bases of walk length
- Anchor coordinates resolve to valid positions within reference bounds (v0.5.2 fixed a bug mapping outside reference bounds)
- Repetitive loci remain distinguishable: identical k-mers kept as distinct graph vertices so anchor precision stays >98% and candidates >94% identical are separated
- Feeding anchors into stage III LCSk chaining yields alignments with >95% precision as reported for GraphMap
- Turning off percentile filtering with --freq-percentile 1.0 (or -x sensitive) should recover sensitivity lost to the default 1% repetitive-seed skip

## Limitations

- GraphMap requires large-memory machines (~100 GB for the human genome); FM-index-based memory reduction was only 'currently under implementation'
- The --freq-percentile 0.99 default slightly reduces sensitivity, trading it for tractability on large references
- Index is a hash table (~14 x reference size) plus minimizer window (default 5); a larger minimizer window shrinks memory but probably drops sensitivity

## Evidence

- [readme] stage II graph-based vertex-centric anchor construction: [evidence span withheld — anchor status: external_doc]
- [readme] seed-hit percentile filtering: "The number of hits for each seed lookup is now thresholded by a percentile value, e.g. --freq-percentile 0.99 means that 1% of the most repetitive seeds will be skipped."
- [abstract] graph traversal rationale for precision: "progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%)"
- [discussion] memory requirement caveat: "A drawback of the current implementation of GraphMap is the requirement of large-memory machines for mapping to large genomes (∼100 GB for the human genome)"
