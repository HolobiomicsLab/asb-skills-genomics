---
name: region-quality-scoring
description: Use when chaining alignment anchors for long, error-prone (e.g., Oxford
  Nanopore MinION) reads and you must rank or prune candidate mapping regions before
  final gapped alignment.
when_to_use_negative:
- Short-read mapping where a BWT/FM-index mapper (e.g., BWA-MEM) suffices — GraphMap's
  hash index is ~14x the reference size (~42 GB peak for human) and the FM-index memory
  reduction was only 'currently under implementation'
- Already-aligned inputs — the funnel assumes raw seed hits and anchors, not a SAM/BAM
  alignment
- Use cases requiring minimal-memory large-genome indexing; the article notes GraphMap
  requires large-memory machines (~100 GB for the human genome)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: GraphMap
  role: 'Implements the five-stage read funnel: LCSk anchor chaining (stage III),
    L1 regression refinement, region quality scoring (Eqs. 4–7), and Myers bit-vector
    final alignment'
  repo: https://github.com/isovic/graphmap
- name: GCC
  role: Toolchain used to compile GraphMap from source via its Makefile targets
- name: make
  role: Build system invoked as 'make modules' then 'make' to produce the graphmap
    binary
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:5c35ded60f6e66c72b9ef5b14893f9b6db3f177981b72a2b4aeb8e7b2f09302e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - GCC
  - GraphMap
  - make
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# region-quality-scoring

## Summary

GraphMap's candidate-region scoring step (stage IV of its five-stage read funnel), where chained anchor paths from the LCSk chaining stage are refined by L1 linear regression, filtered to a ±3-sigma residual band, and ranked with the paper's region quality scoring formulae (Eqs. 4–7) to select the best mapping locations on nanopore reads.

## When to use

Use when chaining alignment anchors for long, error-prone (e.g., Oxford Nanopore MinION) reads and you must rank or prune candidate mapping regions before final gapped alignment.

## When NOT to use

- Short-read mapping where a BWT/FM-index mapper (e.g., BWA-MEM) suffices — GraphMap's hash index is ~14x the reference size (~42 GB peak for human) and the FM-index memory reduction was only 'currently under implementation'
- Already-aligned inputs — the funnel assumes raw seed hits and anchors, not a SAM/BAM alignment
- Use cases requiring minimal-memory large-genome indexing; the article notes GraphMap requires large-memory machines (~100 GB for the human genome)

## Inputs

- Stage II anchor hits (graph-refined seed anchors) per nanopore read
- Reference genome sequence (FASTA)
- Long, error-prone reads in FASTA/FASTQ (Oxford Nanopore MinION 1d/2d or PacBio)

## Outputs

- Ranked list of candidate mapping regions per read (scored via Eqs. 4–7)
- Final gapped alignment emitted to a SAM file (e.g., graphmap_alignments.sam) with mapping quality and BLAST-style E-value (ZE:f tag)

## How to apply

Load the Stage II anchor hits for each read together with the reference genome. Chain anchors with a variable-length LCSk (longest common subsequence of k-mers) procedure to build candidate mapping paths. For each candidate path, fit a 45-degree line by L1 (least absolute deviation) regression, discard anchors whose residuals fall outside the ±3-sigma confidence band, and re-run the LCSk chaining on the retained anchors. Score the surviving candidate regions with the region quality scoring formulae (Eqs. 4–7) to rank and select the best candidate locations, then refine the top candidates with Myers bit-vector alignment for the final gapped alignment. This progressive funnel is what allows GraphMap to keep mapping precision above 95% despite high nanopore error rates, while remaining mapping-position-agnostic to alignment parameters.

## Related tools

- **GraphMap** (Implements the five-stage read funnel: LCSk anchor chaining (stage III), L1 regression refinement, region quality scoring (Eqs. 4–7), and Myers bit-vector final alignment) — https://github.com/isovic/graphmap
- **GCC** (Toolchain used to compile GraphMap from source via its Makefile targets)
- **make** (Build system invoked as 'make modules' then 'make' to produce the graphmap binary)

## Examples

```
./bin/Linux-x64/graphmap align -r reference.fa -d reads.fasta -o output.sam
```

## Evaluation signals

- Emitted SAM file parses correctly and alignment positions fall within reference bounds (v0.5.2 specifically fixed mapping outside of reference bounds)
- Progressive-refinement pipeline attains the reported high mapping precision (>95%) on processed nanopore reads; location precision typically >98%
- GraphMap can distinguish candidate locations that are >94% identical on the human genome — near-identical paralogous loci should be resolved rather than collapsed
- Mapped bases should exceed ~95% of read bases on MinION data, a 10–80% increase over other mappers
- Unmapped alignments carry a valid float ZE:f E-value tag (v0.5.2 changed Inf to std::numeric_limits<float>::max())

## Limitations

- GraphMap requires large-memory machines for mapping to large genomes (~100 GB for the human genome); the FM-index-based memory reduction was only 'currently under implementation'
- The percentile seed filter (--freq-percentile 0.99, skipping the 1% most repetitive seeds) slightly reduces sensitivity, notably dropping recall (but not precision) on transcriptome mapping prior to fixes
- With --auto-rebuild-index absent, incompatible index files halt execution instead of automatically rebuilding

## Evidence

- [abstract] progressive refinement with high precision: "progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%)"
- [readme] LCSk chaining in read funnel: [evidence span withheld — anchor status: external_doc]
- [discussion] precision and paralog discrimination: "GraphMap's precision is typically >98% and it is able to distinguish between candidate locations that are >94% identical on the human genome"
- [readme] index memory footprint: [evidence span withheld — anchor status: not_found]
