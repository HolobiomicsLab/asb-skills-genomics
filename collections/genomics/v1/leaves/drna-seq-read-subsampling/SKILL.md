---
name: drna-seq-read-subsampling
description: Use when you have a high-depth dRNA-seq dataset (e.g. PromethION human
  RNA) and need to quantify how coverage depth influences m6A tool sensitivity, precision/recall,
  or call counts.
when_to_use_negative:
- The dataset is already low-coverage — subsampling further will not reveal the depth-performance
  relationship and will starve multi-sample tools like ELIGOS, Yanocomp, DiffErr and
  Nanocompore that benefit from higher coverage.
- The goal is to evaluate m6A stoichiometry effects on in vitro synthetic oligos rather
  than coverage effects on a biological transcriptome.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0080
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline that executes the 14 m6A detection tools on each subsampled
    depth level and produces per-tool calls
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine used to run pipeline.nf with pipeline.conf for each coverage-depth
    condition
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:4b1991ba99645d4fdba71cd5d6332a23f0558d1983aeea5f7bb68d83de0ffc8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# dRNA-seq read subsampling

## Summary

Randomly subsample a high-coverage Nanopore dRNA-seq dataset to defined depth fractions (25%, 50%, 75%, with 100% as reference) to characterize how sequencing depth affects m6A detection tool performance (hit counts, default-condition F1, AUPRC) via saturation curves.

## When to use

Use when you have a high-depth dRNA-seq dataset (e.g. PromethION human RNA) and need to quantify how coverage depth influences m6A tool sensitivity, precision/recall, or call counts.

## When NOT to use

- The dataset is already low-coverage — subsampling further will not reveal the depth-performance relationship and will starve multi-sample tools like ELIGOS, Yanocomp, DiffErr and Nanocompore that benefit from higher coverage.
- The goal is to evaluate m6A stoichiometry effects on in vitro synthetic oligos rather than coverage effects on a biological transcriptome.

## Inputs

- High-coverage human PromethION dRNA-seq reads (SRA BioProject PRJNA995902), basecalled FAST5/FASTQ
- samples.tsv file listing sample name, condition, and path to base-called fast5 folder
- m6A gold-standard peaks bed file (--peaksfile)
- pipeline.conf Nextflow configuration with desired options

## Outputs

- Per-depth (25/50/75/100%) subsampled read sets
- Saturation table of per-tool number of m6A hits, default-condition F1, and AUPRC at each coverage depth
- Saturation curves plotting hit counts, F1, and AUPRC versus coverage per tool

## How to apply

Retrieve the high-depth human dRNA-seq reads (SRA BioProject PRJNA995902, PromethION platform) and randomly subsample reads to 25%, 50% and 75% of the full set, keeping the 100% run as the reference depth. For each subsampled dataset, execute the NanOlympicsMod Nextflow pipeline ('nextflow -c pipeline.conf run pipeline.nf --samples=<samples.tsv> --resultsDir=<results_dir>') so all 14 m6A detection tools run per depth. Filter called m6A sites with the 'default' threshold set (--threshold) so F1 is comparable at default conditions across depths. Then run the post-processing and statistical_analysis R scripts to compare each tool's hits against the m6A gold-standard peaks bed file, computing per-tool hit counts, default-condition F1, and AUPRC at each depth. Assemble per-depth metrics into a saturation table and plot hits, F1, and AUPRC versus coverage. Rationale: precision and recall of m6A tools are markedly influenced by sequencing depth, so depth-tuned cut-offs matter when interpreting benchmark results.

## Related tools

- **NanOlympicsMod** (Nextflow pipeline that executes the 14 m6A detection tools on each subsampled depth level and produces per-tool calls) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine used to run pipeline.nf with pipeline.conf for each coverage-depth condition) — https://github.com/mfurla/NanOlympicsMod

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples=samples.tsv --resultsDir=results_depth50 --threshold default
```

## Evaluation signals

- Per-tool number of m6A hits and AUPRC should increase monotonically with sequencing depth.
- DiffErr hits are the expected exception to the monotonic-increase trend (opposite trend), and EpiNano-SVM AUPRC may deviate as well.
- F1 must be computed at the 'default' --threshold setting across all depths so the comparison is consistent.
- Precision and recall should be markedly influenced by sequencing depth, matching the paper's reported behavior.
- Each subsampled dataset should be validated as a true random subset of the original read set (read IDs disjoint across fractions, 25/50/75% of full read counts).

## Limitations

- The high-depth human dataset consists of >22 M files totalling 4 TB of raw data, so the saturation analysis carries a substantial computational burden.
- Several m6A tools could not complete benchmark runs (lack of progress or empty output files), so not all tools contribute per-depth metrics.
- Yeast, mouse and human datasets lack a true m6A ground truth; reference sets come from orthogonal short-read methods, and possible bias differences confound performance assessment.

## Evidence

- [discussion] Saturation analysis of hits, F1 at default conditions, and AUPRC across coverage depth values: [evidence span withheld — anchor status: not_found]
- [discussion] Subsampling human PromethION dataset: "we sequenced a human dataset taking advantage of the higher throughput PromethION platform"
- [abstract] Precision/recall influenced by depth: "precision and recall of tools are markedly influenced by sequencing depth, and that additional sequencing would likely reveal additional m6A sites."
- [readme] Pipeline invocation: "nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir""
- [methods] Threshold filtering option: "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
