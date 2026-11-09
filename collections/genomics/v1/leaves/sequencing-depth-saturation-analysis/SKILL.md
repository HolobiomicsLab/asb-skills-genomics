---
name: sequencing-depth-saturation-analysis
description: Use when you have a high-coverage dRNA-seq dataset (e.g. PromethION human
  reads, SRA BioProject PRJNA995902) and need to quantify how sequencing depth affects
  m6A tool performance (hits, F1, AUPRC).
when_to_use_negative:
- When the dataset has low coverage and cannot be meaningfully subsampled to multiple
  depths (e.g. the oligos, yeast or mouse datasets used for the main benchmark rather
  than the high-throughput PromethION human data).
- When the goal is to compare tools at a single fixed depth rather than characterize
  performance as a function of sequencing depth.
- When assessing the 'relaxed' threshold regime specifically — this skill fixes filtering
  at the 'default' threshold set so F1 is comparable across depths.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3170
tools:
- name: NanOlympicsMod
  role: Containerized Nextflow workflow that executes and benchmarks the 14 m6A detection
    tools on each subsampled coverage level
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine used to run pipeline.nf per depth via 'nextflow -c pipeline.conf
    run pipeline.nf --samples=... --resultsDir=...'
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:b14ecb1fba9973fe2b5ff09a8f75e31c5b1eab178b384721962b5a48e1159f37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sequencing-depth-saturation-analysis

## Summary

Reproduces the NanOlympicsMod coverage-saturation analysis, in which a high-depth human PromethION dRNA-seq dataset is subsampled to 25%, 50%, 75% and 100% of reads to quantify how sequencing depth affects the number of m6A hits called, default-condition F1 scores, and AUPRC for each of the benchmarked m6A detection tools.

## When to use

Use when you have a high-coverage dRNA-seq dataset (e.g. PromethION human reads, SRA BioProject PRJNA995902) and need to quantify how sequencing depth affects m6A tool performance (hits, F1, AUPRC).

## When NOT to use

- When the dataset has low coverage and cannot be meaningfully subsampled to multiple depths (e.g. the oligos, yeast or mouse datasets used for the main benchmark rather than the high-throughput PromethION human data).
- When the goal is to compare tools at a single fixed depth rather than characterize performance as a function of sequencing depth.
- When assessing the 'relaxed' threshold regime specifically — this skill fixes filtering at the 'default' threshold set so F1 is comparable across depths.

## Inputs

- High-depth human dRNA-seq reads from SRA BioProject PRJNA995902 (PromethION platform), basecalled FAST5/FASTQ
- samples.tsv file with sample name, condition and path to basecalled fast5 folder
- m6A gold-standard peaks bed file (--peaksfile)
- NanOlympicsMod pipeline.nf and pipeline.conf configuration

## Outputs

- Per-tool, per-depth table of number of m6A hits, default-condition F1 score, and AUPRC (saturation table)
- Saturation curves plotting hit counts, F1, and AUPRC versus coverage (25/50/75/100%)
- Per-depth results directories produced by the Nextflow pipeline

## How to apply

Retrieve the high-depth human PromethION dRNA-seq reads and randomly subsample them to 25%, 50% and 75% of the full read set, keeping the 100% run as the reference depth. For each subsampled dataset, run the NanOlympicsMod Nextflow pipeline (e.g. 'nextflow -c pipeline.conf run pipeline.nf --samples=<samples.tsv> --resultsDir=<results_dir>') to execute the m6A detection tools per depth. Filter called m6A sites with the 'default' threshold set (--threshold, choosing between 'default' and 'relaxed') so F1 is reported at default conditions across depths. Run the post-processing and statistical_analysis R scripts to compare each tool's hits against the m6A gold-standard peaks bed file, computing per-tool number of hits, default-condition F1, and area under the precision-recall curve at each depth. Assemble per-depth metrics into a saturation table and plot hit counts, F1 and AUPRC versus coverage as saturation curves; the rationale is that precision and recall of tools are markedly influenced by sequencing depth, and cut-off settings should be tuned considering the available coverage depth.

## Related tools

- **NanOlympicsMod** (Containerized Nextflow workflow that executes and benchmarks the 14 m6A detection tools on each subsampled coverage level) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine used to run pipeline.nf per depth via 'nextflow -c pipeline.conf run pipeline.nf --samples=... --resultsDir=...') — https://github.com/mfurla/NanOlympicsMod

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples=samples.tsv --resultsDir=results_50pct --threshold=default
```

## Evaluation signals

- Per-tool number of m6A hits increases monotonically with higher coverage for all tools except DiffErr, which shows the opposite trend.
- EpiNano-SVM AUPRC behaves as the expected exception to monotonic AUPRC increase with depth.
- F1 is reported at default conditions consistently across all depths, using the 'default' --threshold filter set for every run.
- Subsampling fractions (25%, 50%, 75%) plus the full 100% run reproduce four coverage points per tool.
- Precision and recall show marked sensitivity to sequencing depth, consistent with the paper's reported finding.

## Limitations

- The human dataset lacks a true ground truth; performance is assessed against reference m6A sets derived from orthogonal short-read methods, so bias differences between reference sets and nanopore-based techniques confound assessment.
- High computational burden: the full human PromethION run comprises >22 M files totalling 4 TB of raw data.
- Several tools could not complete the benchmark runs; a few were abandoned due to lack of progress or unexpected generation of empty output files.
- Additional sequencing would likely reveal additional m6A sites, so the 100% depth does not represent true saturation.

## Evidence

- [discussion] Saturation analysis of hits, F1 at default conditions, and AUPRC across coverage depth values: [evidence span withheld — anchor status: not_found]
- [discussion] Subsampling fractions: "precision and recall of tools are markedly influenced by sequencing depth, and that additional sequencing would likely reveal additional m6A sites."
- [methods] Threshold parameter: "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
- [readme] Pipeline invocation: "nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir""
- [discussion] Human dataset source: "we sequenced a human dataset taking advantage of the higher throughput PromethION platform"
