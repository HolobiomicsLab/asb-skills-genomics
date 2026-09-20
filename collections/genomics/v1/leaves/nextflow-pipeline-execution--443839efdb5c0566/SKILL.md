---
name: nextflow-pipeline-execution
description: Use when a benchmarking or comparative analysis requires running multiple
  containerized tools over the same dRNA-seq datasets through a single reproducible
  Nextflow workflow, e.g. m6A detection with NanOlympicsMod.
when_to_use_negative:
- The analysis targets a single tool or a bespoke non-containerized method that cannot
  be added to the Nextflow workflow
- Input reads are not base-called Nanopore FAST5 files with a known FAST5 slot for
  basecalled bases (pipeline expects --fast5_slot)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3570
- http://edamontology.org/topic_0622
tools:
- name: NanOlympicsMod
  role: Containerized Nextflow workflow that orchestrates the 14 m6A detection tools
    on dRNA-seq FAST5 data
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine executing pipeline.nf with pipeline.conf configuration
- name: Yanocomp
  role: Multi-sample m6A detection tool executed by the pipeline; FDR threshold set
    in pipeline.conf
- name: ELIGOS
  role: Multi-sample m6A detection tool executed by the pipeline on condition pairs
- name: Nanocompore
  role: Multi-sample m6A detection tool executed by the pipeline on replicate conditions
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:38bccad119858b35a9f1f9f2fa61eab2a032024a7e8da599b06bdfaeb8b0abe8
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3570
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# nextflow-pipeline-execution

## Summary

Execute containerized Nextflow workflows (here NanOlympicsMod for benchmarking 14 m6A detection tools on Nanopore dRNA-seq data) by configuring pipeline.conf and launching pipeline.nf with --samples and --resultsDir, ensuring reproducible, portable multi-tool benchmarking runs.

## When to use

Use when a benchmarking or comparative analysis requires running multiple containerized tools over the same dRNA-seq datasets through a single reproducible Nextflow workflow, e.g. m6A detection with NanOlympicsMod.

## When NOT to use

- The analysis targets a single tool or a bespoke non-containerized method that cannot be added to the Nextflow workflow
- Input reads are not base-called Nanopore FAST5 files with a known FAST5 slot for basecalled bases (pipeline expects --fast5_slot)

## Inputs

- Tab-separated samples file with sample name, condition, and path to base-called fast5 folder
- Base-called FAST5 dRNA-seq read folders (e.g. yeast SK1 Nanopore dRNA-seq or human PromethION PRJNA995902 data)
- pipeline.conf Nextflow configuration file with per-tool thresholds (yanocomp FDR, drummer p-value, differr FDR)
- Reference genome/transcriptome fasta and index files plus genes-to-transcripts and m6A gold-standard peaks bed annotation files

## Outputs

- Per-tool m6A hit bins per sample/condition in the --resultsDir folder
- Filtered m6A site calls at the chosen threshold set ('default' or 'relaxed')
- R post-processing and statistical_analysis outputs (per-tool hit counts, F1, AUPRC) ready for comparison tables and plots

## How to apply

Clone the Nextflow repository (e.g. git clone https://github.com/mfurla/NanOlympicsMod.git), install Nextflow and Singularity, and chmod 755 the pipeline files. Open pipeline.conf and set the mandatory options: the tab-separated --samples file listing sample name, condition and path to base-called fast5 folder; --test_condition (e.g. 'WT'); the reference genome/transcriptome fasta and index; annotation files; per-tool significance thresholds (e.g. yanocomp FDR, drummer p-value, differr FDR); and the 'default' or 'relaxed' --threshold set for m6A-site filtering. Launch with 'nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"'. For multi-condition designs (e.g. separate versus merged yeast SK1 replicates, or 25/50/75/100% subsampled PromethION human reads), change only the samples file between runs while keeping pipeline.conf otherwise identical so results are directly comparable. Rationale: NanOlympicsMod exploits containerized technology so all 14 m6A detection tools run under the same environment, maximizing reproducibility and portability of the benchmark.

## Related tools

- **NanOlympicsMod** (Containerized Nextflow workflow that orchestrates the 14 m6A detection tools on dRNA-seq FAST5 data) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine executing pipeline.nf with pipeline.conf configuration)
- **Yanocomp** (Multi-sample m6A detection tool executed by the pipeline; FDR threshold set in pipeline.conf)
- **ELIGOS** (Multi-sample m6A detection tool executed by the pipeline on condition pairs)
- **Nanocompore** (Multi-sample m6A detection tool executed by the pipeline on replicate conditions)

## Examples

```
git clone https://github.com/mfurla/NanOlympicsMod.git; cd NanOlympicsMod; nextflow -c pipeline.conf run pipeline.nf --samples=/path/to/samples.txt --resultsDir=/path/to/resultsDir
```

## Evaluation signals

- Nextflow completes pipeline.nf without error and populates --resultsDir with per-tool outputs for every sample in the samples file
- Per-tool m6A hit bins are produced for each configured tool; empty output files or stalled runs indicate a tool failed (as occurred for several tools in the paper)
- For replicate-merging comparisons, pairwise overlap of called bins between configurations exceeds 59% for all replicate-capable tools and >99% for 4 of 6
- For saturation analyses, per-tool hit counts and AUPRC increase monotonically with sequencing depth (25%→100%), with DiffErr hits and EpiNano-SVM AUPRC as expected exceptions
- Filtered calls respect the chosen --threshold set ('default' vs 'relaxed') so F1 is comparable across runs

## Limitations

- Several tools could not complete the benchmark runs; some were abandoned due to lack of progress or unexpected generation of empty output files
- High computational burden: the study accumulated >22 M files totalling 4 TB of raw data, and running the full pipeline requires comparable storage/compute
- Pipeline prerequisites include Nextflow and Singularity; Dockerfiles are provided but container engine availability is assumed

## Evidence

- [abstract] We developed NanOlympicsMod, a Nextflow pipeline exploiting containerized technology for comparing 14 tools for m6A detection on dRNA-seq data.: "We developed NanOlympicsMod, a Nextflow pipeline exploiting containerized technology for comparing 14 tools for m6A detection on dRNA-seq data."
- [readme] NanOlympicsMod is a Nextflow pipeline for running multiple m6A detection tools based on Nanopore direct RNA sequencing data.: "NanOlympicsMod is a Nextflow pipeline for running multiple m6A detection tools based on Nanopore direct RNA sequencing data."
- [readme] nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir": "nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir""
- [readme] --samples                                                Path to the tab-separated sample file including sample name, condition and path to base-called fast5 folder: "--samples                                                Path to the tab-separated sample file including sample name, condition and path to base-called fast5 folder"
- [discussion] The source code for the NanOlympicsMod workflow, and for reproducing all the results included in this study, are available at the following GitHub repository: https://github.com/mfurla/NanOlympicsMod: "The source code for the NanOlympicsMod workflow, and for reproducing all the results included in this study, are available at the following GitHub repository: https://github.com/mfurla/NanOlympicsMod"
