---
name: post-hoc-statistical-analysis-of-m6a-calls
description: Use when m6A detection tools have already been run on Nanopore dRNA-seq
  data (e.g. via the NanOlympicsMod Nextflow pipeline) and you need per-tool hit counts,
  Jaccard-style overlap of called bins between configurations, or summary tables of
  detected m6A sites.
when_to_use_negative:
- No m6A detection runs have been completed yet — this skill post-processes existing
  tool outputs and cannot generate calls itself
- Data are not dRNA-seq m6A calls (e.g. short-read m6A peak BED files without nanopore
  per-read signal), since the overlap analysis is defined on NanOlympicsMod-style
  called bins
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline whose R post-processing and statistical analysis scripts
    (Scripts folder) are used to extract and compare per-tool m6A hits
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Yanocomp
  role: Replicate-capable multi-sample m6A tool whose hits are post-processed and
    compared across separate vs merged replicate configurations (FDR threshold set
    in pipeline.conf)
- name: ELIGOS
  role: Replicate-capable multi-sample m6A tool whose detected bins are counted and
    overlap-tested between configurations
- name: Nanocompore
  role: Replicate-capable multi-sample m6A tool whose detected bins are counted and
    overlap-tested between configurations
- name: Nextflow
  role: Workflow engine executing pipeline.nf with pipeline.conf, generating the per-tool
    outputs consumed by the post-processing scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:79a3db70a042d02930c0def1efd79ccc15c1ca72b1df730e28fe96da774d2e35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - ELIGOS
  - NanOlympicsMod
  - Nanocompore
  - Nextflow
  - Yanocomp
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# post-hoc-statistical-analysis-of-m6a-calls

## Summary

Post-process and statistically compare m6A calls produced by dRNA-seq detection tools (e.g. Yanocomp, ELIGOS, Nanocompore, Drummer, DiffErr) using the NanOlympicsMod repository's R post-processing scripts, to extract per-tool hit counts and quantify overlap between analysis configurations.

## When to use

Use when m6A detection tools have already been run on Nanopore dRNA-seq data (e.g. via the NanOlympicsMod Nextflow pipeline) and you need per-tool hit counts, Jaccard-style overlap of called bins between configurations, or summary tables of detected m6A sites.

## When NOT to use

- No m6A detection runs have been completed yet — this skill post-processes existing tool outputs and cannot generate calls itself
- Data are not dRNA-seq m6A calls (e.g. short-read m6A peak BED files without nanopore per-read signal), since the overlap analysis is defined on NanOlympicsMod-style called bins

## Inputs

- NanOlympicsMod pipeline outputs from two configurations (separate-replicates and merged samples file) for replicate-capable tools (ELIGOS, Yanocomp, DiffErr, Nanocompore, Drummer, etc.)
- Per-tool m6A called bins/positions on the yeast SK1 dRNA-seq dataset

## Outputs

- Per-tool number of m6A hit bins for each configuration
- Pairwise overlap percentages (Jaccard-style) of called bins between configurations
- Summary table with per-tool hit counts and overlap percentages

## How to apply

After running the NanOlympicsMod pipeline configurations (e.g. yeast SK1 dRNA-seq replicates as separate samples versus merged into one entry in the tab-separated samples file), apply the repository's R post-processing and statistical analysis scripts from the Scripts folder. For each replicate-capable tool, extract the number of m6A hit bins per configuration, then compute the pairwise overlap of called bins between the separate-replicates and merged configurations (Jaccard-style overlap). Assemble per-tool hit counts and overlap percentages into a summary table. Validate against the expected concordance pattern: overlap must exceed 59% for every tool and be greater than 99% for four of the six replicate-capable tools, reproducing Supplementary Figure S5.

## Related tools

- **NanOlympicsMod** (Nextflow pipeline whose R post-processing and statistical analysis scripts (Scripts folder) are used to extract and compare per-tool m6A hits) — https://github.com/mfurla/NanOlympicsMod
- **Yanocomp** (Replicate-capable multi-sample m6A tool whose hits are post-processed and compared across separate vs merged replicate configurations (FDR threshold set in pipeline.conf))
- **ELIGOS** (Replicate-capable multi-sample m6A tool whose detected bins are counted and overlap-tested between configurations)
- **Nanocompore** (Replicate-capable multi-sample m6A tool whose detected bins are counted and overlap-tested between configurations)
- **Nextflow** (Workflow engine executing pipeline.nf with pipeline.conf, generating the per-tool outputs consumed by the post-processing scripts)

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- For every replicate-capable tool, the Jaccard-style overlap between the separate-replicates and merged configurations exceeds 59%
- Four out of six replicate-capable tools show overlaps greater than 99%
- The results reproduce the concordance pattern reported in Supplementary Figure S5
- The configuration with fewer methylated sites typically yields hits that are a subset of the other configuration's hits
- The summary table contains one row per tool with both hit counts and an overlap percentage, computed from both pipeline runs with all other pipeline.conf settings kept identical

## Limitations

- Several tools could not complete the benchmark runs; for a few, runs were abandoned due to lack of progress or unexpected generation of empty output files, so overlap statistics may be unavailable for those tools
- Multi-sample tools benefit from higher coverage; merging replicates alters effective depth and may affect which m6A sites are detected
- Comparison is only meaningful when all pipeline.conf settings other than the samples file are kept identical between configurations

## Evidence

- [readme] post-processing with R scripts: "Scripts: folder containing a bash script to run the nextflow pipeline, a set of R scripts for data post-processing and statistical analysis"
- [discussion] multi-sample tools in yeast dataset: "The multi-sample tools, such as ELIGOS, Yanocomp, DiffErr and Nanocompore performed better in the yeast dataset, probably benefiting from the higher coverage."
- [results] merging affected replicate-capable tools: [evidence span withheld — anchor status: not_found]
- [readme] pipeline invocation: "nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir""
- [methods] threshold filtering: "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
