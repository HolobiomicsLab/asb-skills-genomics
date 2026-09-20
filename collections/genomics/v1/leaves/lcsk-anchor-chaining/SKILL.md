---
name: lcsk-anchor-chaining
description: Use when refining coarse seed-hit anchors from noisy long reads (e.g.
  Oxford Nanopore MinION) into a small set of candidate mapping locations, prior to
  final gapped alignment.
when_to_use_negative:
- Reads are short and high-accuracy (e.g. Illumina) where standard short-read mappers
  suffice and the five-stage funnel is unnecessary
- Input lacks pre-computed Stage II anchors — LCSk chaining requires seed-derived
  alignment anchors, not raw reads alone
- 'Reference is very large and memory-constrained: GraphMap''s hash index needs ~14x
  reference size (~100 GB peak for human genome)'
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: GraphMap
  role: Implements the five-stage read-funneling pipeline including LCSk anchor chaining
    (stage III) and L1 regression refinement (stage IV)
  repo: https://github.com/isovic/graphmap
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:db9a449ecd91c6fef01abb89aa5fea30499bccf21afa3a7bba33051bc80812e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - GCC
  - GraphMap
  - make
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lcsk-anchor-chaining

## Summary

GraphMap's Stage III–IV candidate refinement: chain Stage II alignment anchors with a variable-length k-mer longest common subsequence (LCSk) procedure, then reject outlier anchors via L1 (least absolute deviation) regression to a 45-degree line, progressively narrowing candidate mapping locations.

## When to use

Use when refining coarse seed-hit anchors from noisy long reads (e.g. Oxford Nanopore MinION) into a small set of candidate mapping locations, prior to final gapped alignment.

## When NOT to use

- Reads are short and high-accuracy (e.g. Illumina) where standard short-read mappers suffice and the five-stage funnel is unnecessary
- Input lacks pre-computed Stage II anchors — LCSk chaining requires seed-derived alignment anchors, not raw reads alone
- Reference is very large and memory-constrained: GraphMap's hash index needs ~14x reference size (~100 GB peak for human genome)

## Inputs

- Stage II anchor hits (graph-processed seed hits) for each nanopore read
- Reference genome sequence (e.g. FASTA)
- Long error-prone reads (Oxford Nanopore MinION 1d/2d or PacBio, FASTA/FASTQ)

## Outputs

- Chained candidate mapping paths per read
- Ranked candidate regions with region quality scores (Eqs. 4–7)
- Candidate locations for downstream Myers bit-vector refinement and SAM output (graphmap_alignments.sam)

## How to apply

Load the Stage II anchor hits for each nanopore read together with the reference genome sequence. Chain the anchors using a kmer version of longest common subsequence construction (LCSk) to build candidate mapping paths through the read funnel. For each candidate path, fit a 45-degree line by L1 (least absolute deviation) regression and discard anchors whose residuals fall outside the ±3-sigma confidence band, then re-run LCSk chaining on the retained anchors. Score the resulting candidate regions with GraphMap's region quality scoring formulae (Eqs. 4–7) to rank candidate locations, after which the top candidates can be refined with Myers bit-vector alignment. This progressive refinement is the mechanism behind GraphMap's >95% mapping precision on high-error nanopore reads.

## Related tools

- **GraphMap** (Implements the five-stage read-funneling pipeline including LCSk anchor chaining (stage III) and L1 regression refinement (stage IV)) — https://github.com/isovic/graphmap

## Examples

```
./bin/Linux-x64/graphmap align -r reference.fa -d reads.fasta -o output.sam
```

## Evaluation signals

- Emitted SAM file (graphmap_alignments.sam) parses correctly as valid SAM
- Mapping precision exceeds 95% on processed reads, consistent with reported results
- Anchor residuals after L1 regression fall within the ±3-sigma confidence band for retained anchors
- Location precision is typically >98% and GraphMap distinguishes candidate locations >94% identical on the human genome
- Region quality scores (Eqs. 4–7) correctly rank the true location above spurious candidates

## Limitations

- GraphMap requires large-memory machines (~100 GB for the human genome); memory-efficient index structures were not yet implemented at publication
- The percentile seed filtering (--freq-percentile 0.99, skipping 1% most repetitive seeds) slightly reduces sensitivity, though it speeds up mapping to large references
- L1 regression refinement assumes a monotone, near-diagonal anchor path; highly rearranged or split alignments may shed anchors

## Evidence

- [readme] GraphMap then chains anchors using a kmer version of longest common subsequence (LCSk) construction (stage III): [evidence span withheld — anchor status: external_doc]
- [abstract] progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%): "progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%)"
- [discussion] GraphMap's precision is typically >98% and it is able to distinguish between candidate locations that are >94% identical on the human genome: "GraphMap's precision is typically >98% and it is able to distinguish between candidate locations that are >94% identical on the human genome"
- [readme] The number of hits for each seed lookup is now thresholded by a percentile value, e.g. --freq-percentile 0.99 means that 1% of the most repetitive seeds: "The number of hits for each seed lookup is now thresholded by a percentile value, e.g. --freq-percentile 0.99 means that 1% of the most repetitive seeds"
- [discussion] A drawback of the current implementation of GraphMap is the requirement of large-memory machines for mapping to large genomes (∼100 GB for the human genome): "A drawback of the current implementation of GraphMap is the requirement of large-memory machines for mapping to large genomes (∼100 GB for the human genome)"
