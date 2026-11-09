---
name: genomic-interval-overlap-matrix
description: Use when multiple m6A detection tools have returned genomic BED call
  sets on the same dRNA-seq dataset and you need to quantify how concordant their
  calls are, independent of any ground-truth reference.
when_to_use_negative:
- When the goal is accuracy assessment against a gold-standard m6A peak set (use the
  --peaksfile benchmarking path instead) — this skill measures tool-vs-tool concordance
  only
- When only a single m6A detection tool was run, since a pairwise overlap matrix requires
  at least two callers per dataset
- When input is per-read/per-transcript raw signal (e.g. FAST5 or resquiggled data)
  rather than discrete BED call coordinates
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: NanOlympicsMod
  role: Supplies the per-tool m6A BED outputs, the --binLength binning parameter,
    and the R scripts used to compute and plot the pairwise overlap matrices
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Executes the containerized NanOlympicsMod pipeline that generates the tool
    call sets feeding the concordance analysis
  repo: https://github.com/mfurla/NanOlympicsMod
- name: R
  role: Post-processing and statistical-analysis scripts that compute the pairwise
    overlap matrices and render concordance heatmaps
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:625f0b4773ce9191297c340d2980dad3fae034115091f53e47dfb5ef130193ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# genomic-interval-overlap-matrix

## Summary

Computes pairwise concordance matrices of m6A site calls across dRNA-seq detection tools by partitioning the genome into fixed-size bins, labeling each bin as m6A-positive per tool, and measuring bin-level overlap between all tool pairs.

## When to use

Use when multiple m6A detection tools have returned genomic BED call sets on the same dRNA-seq dataset and you need to quantify how concordant their calls are, independent of any ground-truth reference.

## When NOT to use

- When the goal is accuracy assessment against a gold-standard m6A peak set (use the --peaksfile benchmarking path instead) — this skill measures tool-vs-tool concordance only
- When only a single m6A detection tool was run, since a pairwise overlap matrix requires at least two callers per dataset
- When input is per-read/per-transcript raw signal (e.g. FAST5 or resquiggled data) rather than discrete BED call coordinates

## Inputs

- Per-tool m6A site BED files (call coordinates) produced by the NanOlympicsMod pipeline for each dataset
- Bin length setting per dataset (5 nt for oligos; 50 nt for yeast, mouse, human)
- R post-processing and statistical-analysis scripts from the NanOlympicsMod repository

## Outputs

- Pairwise overlap (concordance) matrix of m6A-positive bins across tools, per dataset
- Concordance heatmap image per dataset
- Matrix table written to the results directory

## How to apply

Load the per-tool m6A site BED files produced by the NanOlympicsMod pipeline and parse each tool's call coordinates per dataset. Partition the query space into fixed-size genomic bins using the pipeline --binLength setting: 5 nt bins for the synthetic oligos dataset and 50 nt bins for the yeast, mouse and human biological datasets. Label a bin m6A-positive for a given tool when at least one of that tool's calls falls within the bin, then compute the pairwise overlap matrix of m6A-positive bins across all tool pairs using the R post-processing and statistical-analysis scripts shipped with NanOlympicsMod. Restrict average-overlap summaries to tools yielding more than 1000 called sites, excluding sparse callers, and render one concordance heatmap per dataset.

## Related tools

- **NanOlympicsMod** (Supplies the per-tool m6A BED outputs, the --binLength binning parameter, and the R scripts used to compute and plot the pairwise overlap matrices) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Executes the containerized NanOlympicsMod pipeline that generates the tool call sets feeding the concordance analysis) — https://github.com/mfurla/NanOlympicsMod
- **R** (Post-processing and statistical-analysis scripts that compute the pairwise overlap matrices and render concordance heatmaps) — https://github.com/mfurla/NanOlympicsMod

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- On the synthetic oligos dataset, the average pairwise overlap among tools with >1000 m6A calls should approximate 0.93
- Exactly one concordance matrix table/heatmap per dataset (oligos, yeast, mouse, human) must be present in the results folder
- Every bin labeled m6A-positive for a tool must contain at least one of that tool's call coordinates within the bin
- All tool pairs (N x N symmetric matrix) are represented, with diagonal values of full self-overlap (1.0)
- Summary statistics exclude sparse callers (<1000 called sites) as stipulated by the workflow

## Limitations

- Oligos dataset is an artificial condition that poorly recapitulates the relative location and frequency of in vivo m6A marks, so oligos concordance may overstate real-data agreement
- Sparse callers with fewer than 1000 sites are excluded from the average-overlap summary, so the matrix does not characterize very low-output tools
- Bin size differs by dataset (5 nt vs 50 nt), so overlap values are not directly comparable across datasets

## Evidence

- [abstract] We developed NanOlympicsMod, a Nextflow pipeline exploiting containerized technology for comparing 14 tools for m6A detection on dRNA-seq data.: "We developed NanOlympicsMod, a Nextflow pipeline exploiting containerized technology for comparing 14 tools for m6A detection on dRNA-seq data."
- [readme] Scripts folder with R post-processing and statistical analysis: "Scripts: folder containing a bash script to run the nextflow pipeline, a set of R scripts for data post-processing and statistical analysis"
- [discussion] Tool performance differing across datasets motivates concordance assessment: "The multi-sample tools, such as ELIGOS, Yanocomp, DiffErr and Nanocompore performed better in the yeast dataset"
