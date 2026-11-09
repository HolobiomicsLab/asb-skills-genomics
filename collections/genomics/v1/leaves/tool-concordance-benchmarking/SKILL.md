---
name: tool-concordance-benchmarking
description: Use when multiple m6A detection tools have been run on the same dRNA-seq
  data and you need to quantify pairwise agreement of called sites over a binned genomic
  coordinate space.
when_to_use_negative:
- Only a single m6A detection tool was run — no pairwise overlap matrix can be computed
- The goal is accuracy assessment against a gold-standard peak set (use the --peaksfile
  benchmarking path instead of concordance)
- Tool calls have not yet been generated — the pipeline must first produce per-tool
  m6A BED outputs (task_001)
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3524
tools:
- name: NanOlympicsMod
  role: Produces the per-tool m6A site BED files and ships the R scripts for post-processing
    and statistical analysis used to compute the concordance matrices
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Execution engine for the containerized NanOlympicsMod pipeline that generates
    the upstream tool-call inputs
  repo: https://github.com/mfurla/NanOlympicsMod
- name: R
  role: Post-processing and statistical-analysis scripts that bin calls, compute pairwise
    overlap matrices and render heatmaps
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:6e3113477c812bb2a39dcb43bd03a39cfd6ee08cdcb18c3798c750cc70d825fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  - R
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# tool-concordance-benchmarking

## Summary

Reproduce the pairwise concordance matrices of m6A site calls across dRNA-seq detection tools on the oligos, yeast, mouse and human datasets, quantifying how much different tools overlap in binned genomic query space using the NanOlympicsMod R post-processing scripts.

## When to use

Use when multiple m6A detection tools have been run on the same dRNA-seq data and you need to quantify pairwise agreement of called sites over a binned genomic coordinate space.

## When NOT to use

- Only a single m6A detection tool was run — no pairwise overlap matrix can be computed
- The goal is accuracy assessment against a gold-standard peak set (use the --peaksfile benchmarking path instead of concordance)
- Tool calls have not yet been generated — the pipeline must first produce per-tool m6A BED outputs (task_001)

## Inputs

- Per-tool m6A site BED files from the NanOlympicsMod pipeline (task_001 outputs), one per tool per dataset
- Pipeline configuration with --binLength setting (5 nt for oligos; 50 nt for yeast, mouse, human)
- List of m6A detection tools with their per-dataset total site counts (for the >1000-site filter)

## Outputs

- One pairwise concordance heatmap per dataset (oligos, yeast, mouse, human)
- One pairwise overlap matrix table per dataset written to the results directory
- Average pairwise overlap summary restricted to tools with >1000 called sites (expected ~0.93 on oligos)

## How to apply

Load the per-tool m6A site BED files produced upstream by the NanOlympicsMod Nextflow pipeline and parse call coordinates for each of the 14 tools per dataset (oligos, yeast SK1, mouse, human PRJNA995902). Partition the query space into fixed-size genomic bins using the pipeline --binLength setting: 5 nt bins for the synthetic oligos dataset and 50 nt bins for the yeast, mouse and human biological datasets. Label each bin m6A-positive for a given tool when at least one of that tool's calls falls within the bin, then compute the pairwise overlap matrix across all tool pairs with the R post-processing and statistical-analysis scripts shipped in the NanOlympicsMod Scripts folder. When summarizing average overlap, restrict to tools yielding more than 1000 called sites to exclude sparse callers. Render one concordance heatmap per dataset and write the matrix table to the results directory.

## Related tools

- **NanOlympicsMod** (Produces the per-tool m6A site BED files and ships the R scripts for post-processing and statistical analysis used to compute the concordance matrices) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Execution engine for the containerized NanOlympicsMod pipeline that generates the upstream tool-call inputs) — https://github.com/mfurla/NanOlympicsMod
- **R** (Post-processing and statistical-analysis scripts that bin calls, compute pairwise overlap matrices and render heatmaps) — https://github.com/mfurla/NanOlympicsMod

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir" && Rscript Scripts/concordance_matrix.R --resultsDir results/ --binLength 50
```

## Evaluation signals

- Average pairwise overlap among tools with >1000 called sites on the synthetic oligos dataset approximates 0.93
- Exactly one concordance matrix table and one heatmap are present per dataset in the results folder
- Bins are labeled m6A-positive using the correct --binLength per dataset: 5 nt for oligos, 50 nt for yeast/mouse/human
- Matrix is square, symmetric, with diagonal values of 1, and covers all tool pairs
- Sparse callers (<1000 sites) are excluded from the average-overlap summary

## Limitations

- The oligos dataset is an artificial condition that poorly recapitulates the relative location and frequency of in vivo marks, so concordance there may not transfer to biological data
- Several tools could not complete the pipeline runs (abandoned due to lack of progress or empty output files), so the matrix may omit some of the 14 benchmarked tools
- Concordance measures agreement between tools but not correctness — tools may be concordant yet biased relative to the orthogonal reference m6A sets

## Evidence

- [results] In the oligos dataset, tools that identified the most m6A sites (>1000) were in very good agreement with each other, with an average pairwise overlap of 0.93.: [evidence span withheld — anchor status: not_found]
- [methods] Partition query space into fixed-size genomic bins using the pipeline --binLength setting — 5 nt bins for the synthetic oligos dataset and 50 nt bins for the yeast, mouse and human biological datasets.: [evidence span withheld — anchor status: not_found]
- [readme] Scripts: folder containing a bash script to run the nextflow pipeline, a set of R scripts for data post-processing and statistical analysis: "Scripts: folder containing a bash script to run the nextflow pipeline, a set of R scripts for data post-processing and statistical analysis"
- [discussion] the oligos dataset represents an artificial condition that poorly recapitulates the relative location and frequency of in vivo marks: [evidence span withheld — anchor status: not_found]
