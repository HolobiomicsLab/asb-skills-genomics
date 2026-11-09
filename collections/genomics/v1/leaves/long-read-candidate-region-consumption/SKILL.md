---
name: long-read-candidate-region-consumption
description: Use when a long-read mapper must convert coarse stage-I seed-hit clusters
  (candidate regions) into precise alignment anchors for downstream anchor chaining
  (LCSk), particularly for error-prone MinION reads.
when_to_use_negative:
- Input is raw reads with no stage I candidate regions — stage II consumes clustered
  seed hits, not raw read files
- The goal is de novo read overlapping rather than reference-guided anchoring — use
  GraphMap's owler overlapping mode instead
- Maximal sensitivity is required on a small reference and the --freq-percentile 0.99
  filter would discard genuine repetitive hits — use the composite -x sensitive option
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: GraphMap
  role: Implements the five-stage read-funneling algorithm; stage II refines stage
    I clustered seed hits into alignment anchors via graph-based vertex-centric seed
    processing before stage III LCSk chaining
  repo: https://github.com/isovic/graphmap
- name: GCC/G++
  role: Builds the GraphMap C++ binaries from source (make modules; make)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:ad283509ae045ecad829f5f1f0544d5591a9ffbe2a51e8077ba2b020f66703c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - GCC/G++
  - GraphMap
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# long-read-candidate-region-consumption

## Summary

Reconstructs GraphMap's stage II, in which clustered seed hits from stage I are refined into alignment anchors via graph-based vertex-centric seed processing, enabling seed-level parallelism on long, error-prone nanopore reads.

## When to use

Use when a long-read mapper must convert coarse stage-I seed-hit clusters (candidate regions) into precise alignment anchors for downstream anchor chaining (LCSk), particularly for error-prone MinION reads.

## When NOT to use

- Input is raw reads with no stage I candidate regions — stage II consumes clustered seed hits, not raw read files
- The goal is de novo read overlapping rather than reference-guided anchoring — use GraphMap's owler overlapping mode instead
- Maximal sensitivity is required on a small reference and the --freq-percentile 0.99 filter would discard genuine repetitive hits — use the composite -x sensitive option

## Inputs

- Reference genome FASTA (with a prebuilt hash/minimizer index, ~14x reference size)
- Stage I clustered seed hits / candidate regions per read
- Long, error-prone sequencing reads (FASTA/FASTQ, e.g. Oxford Nanopore MinION 1d/2d or PacBio)

## Outputs

- Alignment anchors with read and reference coordinates (e.g. anchors.tsv)
- Anchor set feeding the stage III LCSk anchor-chaining stage

## How to apply

Load the reference genome and the stage I candidate regions, resolving seed lookups with the repetitive-seed percentile filter (--freq-percentile 0.99, which skips the 1% most repetitive seeds — disable with --freq-percentile 1.0 or the composite -x sensitive option when maximal sensitivity matters). Seed each read with the default k-mer length T=6, restricting seed hits to the candidate regions passed from stage I. Build a k-mer mapping graph with one vertex per k-mer occurrence, at most l=9 outbound edges per vertex, keeping identical k-mers as distinct vertices so repetitive loci retain separate paths. Traverse the graph with the O(1) vertex-update walk, greedily extending candidate paths from seed vertices without recomputing vertex neighbourhoods, then filter completed walks to those spanning at least 12 bases and record their read/reference coordinates as anchors. The rationale is that graph-based vertex-centric processing allows seed-level parallelism and progressively refines coarse alignments to handle high nanopore error rates, feeding the stage III kmer-LCS anchor-chaining stage.

## Related tools

- **GraphMap** (Implements the five-stage read-funneling algorithm; stage II refines stage I clustered seed hits into alignment anchors via graph-based vertex-centric seed processing before stage III LCSk chaining) — https://github.com/isovic/graphmap
- **GCC/G++** (Builds the GraphMap C++ binaries from source (make modules; make))

## Examples

```
git clone https://github.com/isovic/graphmap.git && cd graphmap && make modules && make && ./bin/Linux-x64/graphmap align -r reference.fa -d reads.fasta -o output.sam
```

## Evaluation signals

- Completed graph walks span at least 12 bases before being recorded as anchors
- Each graph vertex has at most l=9 outbound edges, and identical k-mers remain distinct vertices so repetitive loci keep separate paths
- Seed lookups respect --freq-percentile 0.99 (the 1% most repetitive seeds are skipped) unless overridden
- Downstream stage III (LCSk chaining) receives anchors with consistent read/reference coordinate pairs; mapping precision should exceed the paper's reported >95% / >98% typical precision
- Anchors file contains non-empty read and reference coordinate columns for reads mapped within stage I candidate regions

## Limitations

- GraphMap's hash-table index is ~14x reference size for the final index (peak ~42 GB for a human genome); large-memory machines are required and FM-index-based memory reduction was only 'currently under implementation' at publication
- The --freq-percentile filter slightly reduces sensitivity, trading recall for speed on large references
- Mapping outside reference bounds caused segfaults in earlier versions (<0.5.2); use a patched release

## Evidence

- [intro] Stage II refines stage I clustered seed hits into anchors via vertex-centric graph processing: [evidence span withheld — anchor status: external_doc]
- [abstract] GraphMap progressively refines candidate alignments to handle high error rates: "progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%)"
- [readme] The percentile filter skips the most repetitive seeds: "The number of hits for each seed lookup is now thresholded by a percentile value, e.g. --freq-percentile 0.99 means that 1% of the most repetitive seeds will be skipped."
- [readme] Index size estimate for the hash-table index: [evidence span withheld — anchor status: not_found]
- [readme] Default parameters handle a wide range of error profiles including nanopore: [evidence span withheld — anchor status: external_doc]
