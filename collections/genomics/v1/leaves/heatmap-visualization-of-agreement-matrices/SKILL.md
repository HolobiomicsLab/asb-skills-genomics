---
name: heatmap-visualization-of-agreement-matrices
description: Use when multiple m6A detection tools have produced per-tool site call
  files on the same dRNA-seq data and you need to measure and visualize pairwise agreement,
  e.g. to cluster concordant callers or flag sparse outliers.
when_to_use_negative:
- You need precision/recall or F1 scores against gold-standard m6A peaks — that is
  the benchmarking-against-reference skill, not concordance visualization
- No per-tool m6A call files exist yet — the NanOlympicsMod pipeline (task_001) must
  run first to generate the BED files this skill consumes
- Comparing tool calls against orthogonal reference m6A sets rather than tool-to-tool
  agreement; concordance here does not establish correctness
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: NanOlympicsMod
  role: Provides the R post-processing and statistical-analysis scripts that bin m6A
    calls, compute pairwise overlap matrices, and render the concordance heatmaps
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Executes the NanOlympicsMod pipeline that upstream generates the per-tool
    m6A BED files and results directory consumed by the concordance analysis
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:38688de4a6d19568d7eff89f66c8bd7844dfe77fa43c46223ca9027ea74dc700
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# heatmap-visualization-of-agreement-matrices

## Summary

Computes and renders pairwise concordance (overlap) matrices between m6A detection tools' site calls, binned over a query space, as one heatmap per dRNA-seq dataset using the R post-processing scripts shipped with NanOlympicsMod. It quantifies how consistently the 14 benchmarked tools call m6A on oligos, yeast, mouse and human datasets.

## When to use

Use when multiple m6A detection tools have produced per-tool site call files on the same dRNA-seq data and you need to measure and visualize pairwise agreement, e.g. to cluster concordant callers or flag sparse outliers.

## When NOT to use

- You need precision/recall or F1 scores against gold-standard m6A peaks — that is the benchmarking-against-reference skill, not concordance visualization
- No per-tool m6A call files exist yet — the NanOlympicsMod pipeline (task_001) must run first to generate the BED files this skill consumes
- Comparing tool calls against orthogonal reference m6A sets rather than tool-to-tool agreement; concordance here does not establish correctness

## Inputs

- Per-tool m6A site call BED files produced by the NanOlympicsMod pipeline (task_001)
- Bin length setting from pipeline.conf: 5 nt (oligos) or 50 nt (yeast, mouse, human)
- Per-tool site counts, used to exclude callers with ≤1000 sites from the average-overlap summary

## Outputs

- One pairwise concordance (overlap) matrix table per dataset, written to the results directory
- One concordance heatmap image per dataset (oligos, yeast, mouse, human)
- Average pairwise overlap value restricted to tools with >1000 called sites

## How to apply

Parse each tool's m6A call coordinates (BED files produced upstream by the NanOlympicsMod Nextflow pipeline) for the oligos, yeast, mouse or human dataset. Partition the query space into fixed-size genomic bins — 5 nt bins for the synthetic oligos dataset and 50 nt bins for yeast, mouse and human — and label a bin m6A-positive for a tool when at least one of its calls falls inside it. Compute the pairwise overlap matrix across all tool pairs using the R post-processing and statistical-analysis scripts in the NanOlympicsMod Scripts folder. When summarizing average agreement, restrict the calculation to tools yielding more than 1000 called sites so sparse callers do not deflate the mean (oligos average pairwise overlap ≈ 0.93 for this subset). Render one concordance heatmap per dataset and write the underlying matrix table to the results directory.

## Related tools

- **NanOlympicsMod** (Provides the R post-processing and statistical-analysis scripts that bin m6A calls, compute pairwise overlap matrices, and render the concordance heatmaps) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Executes the NanOlympicsMod pipeline that upstream generates the per-tool m6A BED files and results directory consumed by the concordance analysis) — https://github.com/mfurla/NanOlympicsMod

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- Exactly one concordance matrix table and one heatmap per dataset are present in the results directory
- Average pairwise overlap among tools with >1000 sites on the synthetic oligos dataset approximates 0.93
- Matrix is square with axes equal to the set of tools that produced non-empty m6A calls; diagonal is 1
- Overlap values fall in [0, 1] for every tool pair in every dataset matrix
- Bin assignment reproducible from the declared binLength (5 nt oligos, 50 nt biological datasets)

## Limitations

- Agreement between tools does not imply accuracy — orthogonal reference m6A sets from short-read methods may carry different biases than nanopore-based techniques
- Concordance reflects each tool's detection volume; restricting the average to tools with >1000 sites excludes sparse callers and changes the reported number
- Several tools could not complete benchmark runs (abandoned for lack of progress or empty output), so the tool set per matrix may be smaller than the full 14

## Evidence

- [readme] R scripts for post-processing and statistical analysis: "Scripts: folder containing a bash script to run the nextflow pipeline, a set of R scripts for data post-processing and statistical analysis"
- [discussion] Benchmark scope of 14 tools on the dRNA-seq data: "We benchmarked 14 tools—all those that were available in November 2022—for the detection of m6A on these data"
- [discussion] Dataset difficulty differential motivating per-dataset matrices: "the tools performed very well on the synthetic oligos dataset, while the yeast, mouse and human datasets represented a more challenging task"
