---
name: hough-transform-vote-binning
description: 'Use when mapping high-error nanopore (ONT) reads and you must reduce
  the candidate search space before alignment: seed hits are already collected (e.'
when_to_use_negative:
- Input reads are short, low-error Illumina reads — the paper targets long, error-prone
  third-generation reads; GraphMap's own scope is MinION 1d/2d and PacBio data
- You need the final high-precision alignment rather than coarse candidate localization
  — Hough binning is only Stage I of the five-stage funnel and must be followed by
  graph-based seed processing (Stage II) and LCSk chaining (Stage III)
- Reference is very large and memory is constrained — GraphMap's hash index is ~14x
  reference size (~42 GB peak for human genome); use a BWT/FM-index mapper instead
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: GraphMap
  role: Implements gapped spaced seeds and the read-funneling Stage I whose seed hits
    are clustered as coarse alignments into candidate locations
  repo: https://github.com/isovic/graphmap
- name: GCC
  role: Compiles GraphMap from source (git clone; make modules; make)
- name: graphmap-reproduce
  role: Provides the simulation and evaluation harness (aligneval) used to test GraphMap's
    stages on data with known true origins
  repo: https://github.com/isovic/graphmap-reproduce
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:e5ad22cb21db2fd7b1d8f20eefae3dae1cee769b49db057488799c9bf0f47a14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - GCC
  - GraphMap
  - graphmap-reproduce
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# hough-transform-vote-binning

## Summary

Rasterize gapped-spaced-seed lookup hits from long error-prone reads onto a Hough-transform vote grid (genome position vs read offset), select high-vote bins as candidate mapping regions, and evaluate region recall against known true read origins — the coarse-alignment core of GraphMap's Stage I read funneling.

## When to use

Use when mapping high-error nanopore (ONT) reads and you must reduce the candidate search space before alignment: seed hits are already collected (e.g. GraphMap's 6-1-6 / 4-1-4-1-4 shapes) and need clustering into candidate regions, or you are measuring Stage I region recall on simulated reads with known origins.

## When NOT to use

- Input reads are short, low-error Illumina reads — the paper targets long, error-prone third-generation reads; GraphMap's own scope is MinION 1d/2d and PacBio data
- You need the final high-precision alignment rather than coarse candidate localization — Hough binning is only Stage I of the five-stage funnel and must be followed by graph-based seed processing (Stage II) and LCSk chaining (Stage III)
- Reference is very large and memory is constrained — GraphMap's hash index is ~14x reference size (~42 GB peak for human genome); use a BWT/FM-index mapper instead

## Inputs

- Reference genome FASTA file
- Simulated or real Oxford Nanopore (MinION) long reads with high error rates, ideally with known true genomic origins for recall evaluation
- Pre-built GraphMap hash index over gapped seed shapes 6-1-6 and 4-1-4-1-4

## Outputs

- Candidate genomic regions per read (high-vote Hough bins extended by one read length on each side)
- Per-read and aggregate Stage I region recall metrics (e.g. stage1_recall_report.json)

## How to apply

Build a reference hash index over the Levenshtein gapped q-gram shapes 6-1-6 and 4-1-4-1-4, keying lookups by the do-care (DC) base at the first care position. When extracting seed hits, discard the most repetitive seeds via a frequency percentile threshold (--freq-percentile 0.99 drops the top 1%; the README notes this slightly reduces sensitivity but is critical when mapping to large references). For each read, rasterize seed hits onto a Hough-transform vote grid with genome position on one axis and read offset on the other, accumulating votes per bin. Select bins whose vote count exceeds 75% of the maximum bin count, then extend each selected bin by one read length on both sides to form candidate regions. Validate by comparing predicted regions to the known true origins of simulated ONT reads and computing per-read and aggregate region recall.

## Related tools

- **GraphMap** (Implements gapped spaced seeds and the read-funneling Stage I whose seed hits are clustered as coarse alignments into candidate locations) — https://github.com/isovic/graphmap
- **GCC** (Compiles GraphMap from source (git clone; make modules; make))
- **graphmap-reproduce** (Provides the simulation and evaluation harness (aligneval) used to test GraphMap's stages on data with known true origins) — https://github.com/isovic/graphmap-reproduce

## Examples

```
./bin/Linux-x64/graphmap align -r reference.fa -d reads.fastq -o output.sam --freq-percentile 0.99
```

## Evaluation signals

- Selected bins each exceed 75% of the maximum vote count on the Hough grid before read-length extension
- Region recall against known true origins of simulated ONT reads is high (candidate regions should contain the true origin after one-read-length extension)
- Skipping --freq-percentile (setting 1.0) should raise sensitivity slightly but drastically increase runtime/hits when mapping to large references — a useful ablation check
- Per-read recall metrics written to stage1_recall_report.json should be non-empty and show most reads with at least one candidate region containing the true origin

## Limitations

- GraphMap's hash index requires large-memory machines (~100 GB for the human genome; ~42 GB peak during construction at ~14x reference size); the FM-index memory reduction was only 'currently under implementation'
- Percentile seed filtering (--freq-percentile 0.99) slightly reduces sensitivity in exchange for speed on large references
- This stage is a coarse funnel, not an aligner: candidates still require graph-based anchor construction and LCSk chaining before final high-precision alignment (>95% precision)

## Evidence

- [readme] In stage I, GraphMap uses a novel adaptation of gapped spaced seeds to efficiently reduce the search space and get seed hits as a form of coarse alignment: [evidence span withheld — anchor status: external_doc]
- [intro] The number of hits for each seed lookup is now thresholded by a percentile value, e.g. --freq-percentile 0.99 means that 1% of the most repetitive seeds will be skipped: "The number of hits for each seed lookup is now thresholded by a percentile value, e.g. --freq-percentile 0.99 means that 1% of the most repetitive seeds will be skipped"
- [abstract] progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%): "progressively refines candidate alignments to robustly handle potentially high-error rates and a fast graph traversal to align long reads with speed and high precision (>95%)"
- [discussion] A drawback of the current implementation of GraphMap is the requirement of large-memory machines for mapping to large genomes (~100 GB for the human genome): "A drawback of the current implementation of GraphMap is the requirement of large-memory machines for mapping to large genomes (∼100 GB for the human genome)"
