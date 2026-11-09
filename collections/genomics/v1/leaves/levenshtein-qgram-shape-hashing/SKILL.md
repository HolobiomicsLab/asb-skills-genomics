---
name: levenshtein-qgram-shape-hashing
description: Use when mapping high-error long reads (e.g. Oxford Nanopore MinION FASTA/FASTQ)
  to a large reference genome and you need to reproduce or evaluate how a coarse-alignment
  stage reduces the candidate search space before fine alignment.
when_to_use_negative:
- Reads are short and low-error (e.g. Illumina) — GraphMap's default parameters target
  long, error-prone third-generation reads, and dense spaced-seed hashing is unnecessary
  for high-accuracy short reads
- The goal is fine-grained base-level alignment rather than coarse candidate-region
  selection — Stage I binning only narrows the search space and must be followed by
  graph-based anchor refinement and LCSk chaining (Stages II–III)
- Only very small memory is available and the reference is large — the hash index
  is ~14x reference size (peak ~42 GB for human), so memory-efficient BWT/FM-index
  mappers are preferable
edam_operation: http://edamontology.org/operation_0292
tools:
- name: GraphMap
  role: Implements the gapped spaced-seed hash index, --freq-percentile seed filtering,
    and Hough-style binning of Stage I; used to map long error-prone reads and produce
    candidate locations
  repo: https://github.com/isovic/graphmap
- name: graphmap-reproduce
  role: Repository of scripts to reproduce the GraphMap paper's benchmark results
    on simulated and real data (aligneval-based simulation and evaluation wrappers)
  repo: https://github.com/isovic/graphmap-reproduce
- name: GCC
  role: Toolchain used to build GraphMap from source (make modules; make) before running
    the mapping pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:bb8da34b6f14c95a3df76f1870c5e4be20f5a7bfd6daaa742b5e83430bcc6ab0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - GCC
  - GraphMap
  - graphmap-reproduce
  edam_operation: http://edamontology.org/operation_0292
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# levenshtein-qgram-shape-hashing

## Summary

Reconstructs GraphMap's Stage I 'read-funneling' candidate selection: a reference hash index over Levenshtein gapped q-gram shapes (6-1-6 and 4-1-4-1-4), percentile-thresholded seed lookups, and Hough-transform vote-grid binning of seed hits to nominate candidate genomic regions for high-error nanopore reads.

## When to use

Use when mapping high-error long reads (e.g. Oxford Nanopore MinION FASTA/FASTQ) to a large reference genome and you need to reproduce or evaluate how a coarse-alignment stage reduces the candidate search space before fine alignment.

## When NOT to use

- Reads are short and low-error (e.g. Illumina) — GraphMap's default parameters target long, error-prone third-generation reads, and dense spaced-seed hashing is unnecessary for high-accuracy short reads
- The goal is fine-grained base-level alignment rather than coarse candidate-region selection — Stage I binning only narrows the search space and must be followed by graph-based anchor refinement and LCSk chaining (Stages II–III)
- Only very small memory is available and the reference is large — the hash index is ~14x reference size (peak ~42 GB for human), so memory-efficient BWT/FM-index mappers are preferable

## Inputs

- Reference genome FASTA file (target for hash index construction over gapped seed shapes)
- Long, error-prone reads (e.g. simulated or real Oxford Nanopore MinION reads in FASTA/FASTQ)
- Known true genomic origins of simulated reads (for region-recall evaluation)

## Outputs

- Per-read candidate genomic regions (Hough-transform bins extended by one read length on each side)
- Seed hit sets after percentile-based filtering (--freq-percentile 0.99)
- Per-read and aggregate region recall metrics (e.g. stage1_recall_report.)

## How to apply

Read the reference genome FASTA and build a hash index over two Levenshtein gapped q-gram shapes, 6-1-6 and 4-1-4-1-4, keying each lookup by the base at the seed's first do-care (DC) position; optionally use GraphMap's built-in minimizer index (--minimizer-window, default 5). Extract seed hits per read, discarding the most repetitive seeds via a frequency percentile threshold (--freq-percentile 0.99, i.e. skip the top 1% most repetitive seed lookups) to keep the search tractable on large references. Rasterize each read's seed hits onto a Hough-transform vote grid of genome position vs read offset, accumulating votes per bin. Select bins whose vote count exceeds ~75% of the maximum bin count and extend each selected bin by one read length on both sides to form candidate regions. Validate by comparing predicted candidate regions against known true origins of simulated ONT reads (region recall), e.g. written to stage1_recall_report.; overall mapping quality can be cross-checked with GraphMap's reported precision (>98%) and sensitivity.

## Related tools

- **GraphMap** (Implements the gapped spaced-seed hash index, --freq-percentile seed filtering, and Hough-style binning of Stage I; used to map long error-prone reads and produce candidate locations) — https://github.com/isovic/graphmap
- **graphmap-reproduce** (Repository of scripts to reproduce the GraphMap paper's benchmark results on simulated and real data (aligneval-based simulation and evaluation wrappers)) — https://github.com/isovic/graphmap-reproduce
- **GCC** (Toolchain used to build GraphMap from source (make modules; make) before running the mapping pipeline)

## Examples

```
./bin/Linux-x64/graphmap align -r reference.fa -d reads.fasta -o output.sam --freq-percentile 0.99 --minimizer-window 5
```

## Evaluation signals

- Region recall: predicted Stage I candidate regions overlap the known true origins of simulated ONT reads (compare per-read and aggregate recall in stage1_recall_report.)
- Seed-filter sanity: with --freq-percentile 0.99, roughly 1% of the most repetitive seed lookups are skipped; setting --freq-percentile 1.0 (or -x sensitive) should change hit counts and slightly increase sensitivity
- Index invariants: every stored lookup key corresponds to the base at the first care position of the 6-1-6 or 4-1-4-1-4 seed shapes; index size scales at ~14x reference size
- Bin selection invariant: selected bins have vote counts above ~75% of the maximum bin count, and each selected bin is extended by one read length on both sides
- Downstream check: final GraphMap mappings show location precision typically >98% and mapped bases >95% on MinION data, consistent with a correctly reducing Stage I

## Limitations

- GraphMap's hash index is memory-hungry: about ~14x reference size for the final index (peak ~42 GB for human genome; ~100 GB machine requirement cited in the paper), with the FM-index-based memory reduction only 'currently under implementation'
- Percentile seed filtering (--freq-percentile 0.99) slightly reduces sensitivity in exchange for tractability on large references; use -x sensitive / --freq-percentile 1.0 to recover previous behaviour
- Stage I provides coarse candidate locations only — recall of the funnel depends on seed-hit density, and highly repetitive or large minimizer windows (--minimizer-window > 1) can drop sensitivity

## Evidence

- [readme] Stage I uses a novel adaptation of gapped spaced seeds to reduce candidate search space: [evidence span withheld — anchor status: external_doc]
- [readme] Seed-lookup percentile thresholding parameter: "The number of hits for each seed lookup is now thresholded by a percentile value, e.g. --freq-percentile 0.99 means that 1% of the most repetitive seeds will be skipped."
- [discussion] Gapped spaced seeds are a core GraphMap algorithmic design choice: "The design choices in GraphMap, including the use of new algorithmic ideas such as gapped spaced seeds, graph mapping and longest common subsequence"
- [readme] Hash index size scaling and memory footprint: [evidence span withheld — anchor status: not_found]
- [abstract] High precision from progressive refinement of candidates: "progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%)"
