---
name: kmer-seed-graph-construction
description: Use when coarse seeded candidate regions from a long-read mapper must
  be converted into precise alignment anchors, especially for noisy Oxford Nanopore
  MinION or PacBio reads with repetitive reference loci.
when_to_use_negative:
- 'The task is full end-to-end read mapping: this skill covers only stage-II anchor
  construction — stage-III chaining and alignment stages must follow to produce a
  SAM alignment.'
- 'The reference is very large and RAM is limited: GraphMap''s hash index is ~14x
  the reference size (~42 GB peak for human), so a BWT/FM-index mapper is preferable
  unless the (not-yet-implemented-at-publication) memory-reduction option is available.'
- 'Sensitivity on highly repetitive seeds matters more than speed: setting --freq-percentile
  1.0 (or -x sensitive) is required, since the default 0.99 filter skips 1% of the
  most repetitive seed lookups at a slight sensitivity cost.'
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: GraphMap
  role: 'Implements the stage-II k-mer mapping graph: builds vertices per k-mer occurrence
    with ≤9 outbound edges and traverses via O(1) vertex-update walks to produce alignment
    anchors'
  repo: https://github.com/isovic/graphmap
- name: GCC/G++
  role: Compile the GraphMap C++ binary from source (make modules; make) before running
    the anchor-construction stage
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:9d27a9e5d2119fce1cb27c12de799fdb20d6d670a9c67916dcb621442a2756e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - GCC/G++
  - GraphMap
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# kmer-seed-graph-construction

## Summary

Stage II of GraphMap's five-stage read funnel: refine coarse stage-I seed hits by building a k-mer mapping graph (one vertex per k-mer occurrence, ≤l outbound edges per vertex) and greedily traversing it with O(1) vertex updates to construct alignment anchors for long, error-prone nanopore reads.

## When to use

Use when coarse seeded candidate regions from a long-read mapper must be converted into precise alignment anchors, especially for noisy Oxford Nanopore MinION or PacBio reads with repetitive reference loci.

## When NOT to use

- The task is full end-to-end read mapping: this skill covers only stage-II anchor construction — stage-III chaining and alignment stages must follow to produce a SAM alignment.
- The reference is very large and RAM is limited: GraphMap's hash index is ~14x the reference size (~42 GB peak for human), so a BWT/FM-index mapper is preferable unless the (not-yet-implemented-at-publication) memory-reduction option is available.
- Sensitivity on highly repetitive seeds matters more than speed: setting --freq-percentile 1.0 (or -x sensitive) is required, since the default 0.99 filter skips 1% of the most repetitive seed lookups at a slight sensitivity cost.

## Inputs

- Reference genome FASTA and GraphMap's hash-table seed index (~14x reference size; ~42 GB peak for a human genome)
- Stage-I clustered seed hits / candidate regions from upstream seeding
- Long error-prone sequencing reads in FASTA/FASTQ (Oxford Nanopore MinION 1d/2d, PacBio)

## Outputs

- Alignment anchors with read and reference coordinates (anchors.tsv), each walk at least 12 bases
- Candidate anchor set feeding the stage-III LCSk longest-common-subsequence chaining

## How to apply

Load the reference and stage-I candidate regions, resolving seed lookups through the hash index while filtering the most repetitive 1% of seed hits via the percentile threshold (--freq-percentile 0.99). Seed each read with a short k-mer (default T=6) and restrict hits to the candidate regions. Build the k-mer mapping graph with one vertex per k-mer occurrence, connecting each vertex to at most l=9 outbound edges, deliberately keeping identical k-mers as distinct vertices so repetitive loci retain separate graph paths. Traverse the graph using the O(1) vertex-update walk, greedily extending candidate paths from seed vertices without recomputing vertex neighbourhoods, and keep completed walks spanning at least 12 bases as anchors. Record each anchor with its read and reference coordinates so the downstream LCSk-based anchor-chaining stage (stage III) can consume them.

## Related tools

- **GraphMap** (Implements the stage-II k-mer mapping graph: builds vertices per k-mer occurrence with ≤9 outbound edges and traverses via O(1) vertex-update walks to produce alignment anchors) — https://github.com/isovic/graphmap
- **GCC/G++** (Compile the GraphMap C++ binary from source (make modules; make) before running the anchor-construction stage)

## Examples

```
git clone https://github.com/isovic/graphmap.git && cd graphmap && make modules && make; ./bin/Linux-x64/graphmap align -r reference.fa -d reads.fasta -o output.sam
```

## Evaluation signals

- All retained walks span at least 12 bases; shorter graph walks are discarded before anchor recording.
- Every vertex in the mapping graph has at most l=9 outbound edges, and identical k-mers remain distinct vertices so repetitive loci yield separate paths.
- Repetitive-seed filtering matches --freq-percentile 0.99: the top 1% most frequent seed lookups are excluded (verify against --freq-percentile 1.0 sensitive-mode output for recall comparison).
- Anchor read/reference coordinates are in bounds and consistent with the stage-I candidate regions (README v0.5.2 specifically fixed 'mapping outside of reference bounds').
- Downstream stage III (LCSk chaining) consumes the anchors.tsv without orphan anchors; final mapping precision should reach the paper's reported >95% and location precision typically >98%.

## Limitations

- GraphMap's hash-table index requires large-memory machines (~100 GB for the human genome); FM-index-based memory reduction was 'currently under implementation' at publication time.
- Percentile filtering (--freq-percentile 0.99) slightly reduces sensitivity, trading recall for tractability on large references.
- Minimum-index versions via larger minimizer windows (--minimizer-window) reduce memory but 'will probably cause a drop in sensitivity'.

## Evidence

- [readme] Stage II anchor construction via graph traversal: [evidence span withheld — anchor status: external_doc]
- [readme] Percentile seed filter semantics: "The number of hits for each seed lookup is now thresholded by a percentile value, e.g. `--freq-percentile 0.99` means that `1%` of the most repetitive seeds will be skipped."
- [abstract] High precision via progressive refinement and graph traversal: "progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%)"
- [readme] Memory footprint of the hash index: [evidence span withheld — anchor status: external_doc]
