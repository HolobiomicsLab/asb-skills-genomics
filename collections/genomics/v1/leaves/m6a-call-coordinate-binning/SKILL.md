---
name: m6a-call-coordinate-binning
description: Use when you have per-tool m6A site BED files from multiple dRNA-seq
  detection tools and need a coordinate-level pairwise concordance matrix over a binned
  query space (e.g. oligos, yeast, mouse, human NanOlympicsMod outputs).
when_to_use_negative:
- When the goal is precision/recall benchmarking of tools against gold-standard or
  orthogonal reference m6A peak sets rather than inter-tool concordance
- When input is per-read modification probability data (e.g. FAST5 slot values) rather
  than aggregated per-tool site BED coordinates
- When only a single tool's calls are available — pairwise overlap matrices require
  at least two callers
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3308
tools:
- name: NanOlympicsMod
  role: Produces the per-tool m6A site BED files and ships the R post-processing and
    statistical-analysis scripts used to bin coordinates and compute pairwise overlap
    matrices
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Runs the containerized NanOlympicsMod pipeline that generates the upstream
    m6A call BED files and enforces the --binLength bin setting
  repo: https://github.com/mfurla/NanOlympicsMod
- name: R
  role: Executes the post-processing and statistical-analysis scripts that bin call
    coordinates, label m6A-positive bins, and render the concordance heatmaps
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:cd0ff21c425d3071e4f4f5d53d1bef9ed6c7afd75dd369155fa2ea84509f3d53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# m6a-call-coordinate-binning

## Summary

Quantify concordance between m6A site calls from different dRNA-seq detection tools by partitioning the query space into fixed-size genomic bins, labeling each bin m6A-positive per tool, and computing pairwise overlap matrices with NanOlympicsMod's R post-processing scripts.

## When to use

Use when you have per-tool m6A site BED files from multiple dRNA-seq detection tools and need a coordinate-level pairwise concordance matrix over a binned query space (e.g. oligos, yeast, mouse, human NanOlympicsMod outputs).

## When NOT to use

- When the goal is precision/recall benchmarking of tools against gold-standard or orthogonal reference m6A peak sets rather than inter-tool concordance
- When input is per-read modification probability data (e.g. FAST5 slot values) rather than aggregated per-tool site BED coordinates
- When only a single tool's calls are available — pairwise overlap matrices require at least two callers

## Inputs

- Per-tool m6A site BED files (call coordinates) produced by the NanOlympicsMod pipeline
- Bin length setting: 5 nt for the synthetic oligos dataset, 50 nt for yeast, mouse and human datasets
- Per-tool called-site counts (to apply the >1000-site caller filter)

## Outputs

- One pairwise concordance/overlap matrix table per dataset written to the results directory
- One concordance heatmap per dataset
- Average pairwise overlap summary restricted to tools with >1000 called sites

## How to apply

Load the per-tool m6A site BED files produced upstream by the NanOlympicsMod Nextflow pipeline and parse each tool's call coordinates per dataset. Partition the query space into fixed-size bins using the pipeline --binLength setting — 5 nt bins for the synthetic oligos dataset and 50 nt bins for the yeast (SK1), mouse and human biological datasets. Label a bin m6A-positive for a tool when at least one of that tool's calls falls within the bin, then compute the pairwise overlap matrix of m6A-positive bins across all tool pairs using the R post-processing and statistical-analysis scripts shipped in the NanOlympicsMod Scripts folder. Restrict the average-overlap summary to tools yielding more than 1000 called sites, excluding sparse callers whose overlaps are uninformative. Render one concordance heatmap per dataset and write the matrix table to the results directory.

## Related tools

- **NanOlympicsMod** (Produces the per-tool m6A site BED files and ships the R post-processing and statistical-analysis scripts used to bin coordinates and compute pairwise overlap matrices) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Runs the containerized NanOlympicsMod pipeline that generates the upstream m6A call BED files and enforces the --binLength bin setting) — https://github.com/mfurla/NanOlympicsMod
- **R** (Executes the post-processing and statistical-analysis scripts that bin call coordinates, label m6A-positive bins, and render the concordance heatmaps)

## Evaluation signals

- Average pairwise overlap among tools with >1000 called sites on the synthetic oligos dataset should approximate 0.93
- Exactly one concordance matrix table and one heatmap per dataset must be present in the results folder
- Every bin labeled m6A-positive must contain at least one call from the corresponding tool's BED coordinates
- Bin sizes must match the pipeline --binLength per dataset: 5 nt (oligos) and 50 nt (yeast, mouse, human)
- Sparse callers (<1000 sites) must be excluded from the average-overlap summary

## Limitations

- Bin granularity (5 nt vs 50 nt) is dataset-dependent; concordance values are not directly comparable across bin lengths
- The synthetic oligos dataset is an artificial condition that poorly recapitulates the relative location and frequency of in vivo marks, so oligos-based concordance may overestimate real-world agreement
- Several m6A detection tools could not complete their benchmark runs or produced empty output files, so the tool set contributing to each matrix may be incomplete

## Evidence

- [readme] R scripts role in post-processing and statistical analysis: "a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1 reference genome"
- [intro] Four datasets compared with NanOlympicsMod: "We used NanOlympicsMod to execute and compare 14 tools on four different dRNA-seq datasets differing in terms of synthetic/biological origin, transcriptome size and coverage depth."
- [discussion] Oligos dataset is artificial: [evidence span withheld — anchor status: not_found]
- [methods] BED file input convention: "--peaksfile Path to bed file with set of m6A gold-standard peaks"
