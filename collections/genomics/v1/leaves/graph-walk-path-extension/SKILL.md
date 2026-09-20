---
name: graph-walk-path-extension
description: Use when clustered seed hits from a coarse seeding stage must be converted
  into read/reference coordinate anchors for anchor chaining, especially for long,
  error-prone MinION 1d/2d reads where repetitive loci require separate graph paths.
when_to_use_negative:
- Input is raw reads with no stage I candidate regions — stage I gapped spaced seeding
  must run first to reduce the search space.
- 'Very large references when the goal is exhaustive sensitivity: the percentile filter
  (--freq-percentile) slightly reduces sensitivity; use the composite option -x sensitive
  to switch off minimizers and percentile filtering instead.'
- Mapping to a transcriptome or circular reference requiring specialized handling
  — GraphMap routes these through --gtf or circular-genome handling rather than the
  plain anchor-construction path.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: GraphMap
  role: Implements the five-stage read-funneling pipeline; stage II performs graph-based
    vertex-centric seed processing to construct alignment anchors
  repo: https://github.com/isovic/graphmap
- name: GCC/G++
  role: Compiles the GraphMap C++ source (make modules; make) to produce the Linux-x64
    binary used for mapping
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:18ec2de6094941113fe9460bface0c2d16b6b1125df28dd0bfdd903e2b02b402
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - GCC/G++
  - GraphMap
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# graph-walk-path-extension

## Summary

Refines clustered seed hits into alignment anchors by building a k-mer mapping graph (one vertex per k-mer occurrence, at most 9 outbound edges per vertex, identical k-mers kept as distinct vertices) and greedily traversing it with O(1) vertex-update walks, as done in stage II of GraphMap for nanopore reads.

## When to use

Use when clustered seed hits from a coarse seeding stage must be converted into read/reference coordinate anchors for anchor chaining, especially for long, error-prone MinION 1d/2d reads where repetitive loci require separate graph paths.

## When NOT to use

- Input is raw reads with no stage I candidate regions — stage I gapped spaced seeding must run first to reduce the search space.
- Very large references when the goal is exhaustive sensitivity: the percentile filter (--freq-percentile) slightly reduces sensitivity; use the composite option -x sensitive to switch off minimizers and percentile filtering instead.
- Mapping to a transcriptome or circular reference requiring specialized handling — GraphMap routes these through --gtf or circular-genome handling rather than the plain anchor-construction path.

## Inputs

- Reference genome (FASTA)
- Stage I clustered seed hits / candidate regions from upstream seeding
- Read set (FASTA/FASTQ long reads, e.g. Oxford Nanopore MinION 1d and 2d reads)

## Outputs

- Alignment anchors with read and reference coordinates (anchors.tsv)
- Anchor set ready for stage III anchor chaining via kmer LCSk

## How to apply

Load the reference and the stage I candidate regions, applying the seed-hit percentile filter (--freq-percentile 0.99 discards the most repetitive 1% of seed lookups) when resolving seed hits. Seed each read with the default k-mer length T=6, restricting seed hits to the candidate regions to keep the search space small. Build the k-mer mapping graph with one vertex per k-mer occurrence, connecting each vertex to at most l=9 outbound edges, and keep identical k-mers as distinct vertices so repetitive loci retain separate paths. Traverse the graph using the O(1) vertex-update walk, extending candidate paths greedily from seed vertices without recomputing vertex neighbourhoods, which enables seed-level parallelism. Filter completed walks to those spanning at least 12 bases and record their reference/read positions as anchors that feed the stage III kmer-LCSk chaining step.

## Related tools

- **GraphMap** (Implements the five-stage read-funneling pipeline; stage II performs graph-based vertex-centric seed processing to construct alignment anchors) — https://github.com/isovic/graphmap
- **GCC/G++** (Compiles the GraphMap C++ source (make modules; make) to produce the Linux-x64 binary used for mapping)

## Examples

```
./bin/Linux-x64/graphmap align -r reference.fa -d reads.fasta -o output.sam
```

## Evaluation signals

- Completed graph walks span at least 12 bases before being recorded as anchors
- Each vertex in the k-mer mapping graph has at most l=9 outbound edges
- Identical k-mers remain distinct vertices, so repetitive loci yield separate paths rather than collapsed vertices
- Output anchors carry both read and reference coordinates and feed a stage III kmer LCSk chaining step
- Resulting mappings should reproduce GraphMap's reported high precision (>95%) and >90% of reads mapped on MinION data

## Limitations

- GraphMap requires large-memory machines (~100 GB for the human genome); its hash index is about ~14x reference size (~42 GB peak during construction), and the FM-index-based memory reduction is 'currently under implementation'.
- The --freq-percentile 0.99 thresholding slightly reduces sensitivity, though it plays a huge role when mapping to large references.
- Mapping outside of reference bounds was a bug fixed only in version 0.5.2, so older versions may emit invalid anchor coordinates.

## Evidence

- [intro] Stage II graph-based vertex-centric anchor construction: [evidence span withheld — anchor status: external_doc]
- [readme] Percentile seed-hit filtering parameter: "The number of hits for each seed lookup is now thresholded by a percentile value, e.g. --freq-percentile 0.99 means that 1% of the most repetitive seeds will be skipped."
- [readme] Five-stage read-funneling pipeline context: "The GraphMap algorithm** is structured to achieve high-sensitivity and speed using a five-stage
read-funneling approach."
- [abstract] Graph traversal precision rationale: "a fast graph traversal to align long reads with speed and high precision (>95%)"
