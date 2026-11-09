---
name: nanopore-drna-seq-m6a-detection
description: Use when base-called Nanopore fast5 dRNA-seq data (e.g. SK1 yeast, mouse,
  or human samples) must be profiled for m6A sites and detection tools need to be
  compared or applied under controlled thresholds.
when_to_use_negative:
- Input is already a short-read methylation dataset (this workflow is specific to
  ONT direct RNA fast5 data)
- The goal is calling modifications other than m6A at stoichiometry level with no
  comparison to reference peaks, since this skill centers on m6A detection and benchmarking
- Data are unbasecalled raw signal only and no basecalling step (--fast5_slot for
  basecalled bases) can be performed
edam_operation: http://edamontology.org/operation_0278
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3524
tools:
- name: NanOlympicsMod
  role: Containerized Nextflow pipeline that orchestrates execution and comparison
    of 14 m6A detection tools on dRNA-seq data
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine used to run pipeline.nf with pipeline.conf configuration and
    containerized reproducibility
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Yanocomp
  role: Multi-sample m6A detection tool executed within the pipeline, thresholded
    by FDR; performed well on the yeast dataset
  repo: https://github.com/mfurla/NanOlympicsMod
- name: ELIGOS
  role: Multi-sample m6A detection tool executed within the pipeline, benefiting from
    higher coverage in the yeast dataset
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nanocompore
  role: Multi-sample m6A detection tool executed within the pipeline; performed well
    on the yeast dataset
  repo: https://github.com/mfurla/NanOlympicsMod
- name: ont_fast5_api
  role: Interface for handling ONT fast5 files (single/multi-read conversion) feeding
    the pipeline's fast5 inputs
  repo: https://github.com/nanoporetech/ont_fast5_api
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:02c8022657de4eedaf562df7b66033706baa7f31eab22284140820c75a46f2eb
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
  - ont_fast5_api
  edam_operation: http://edamontology.org/operation_0278
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# nanopore-drna-seq-m6a-detection

## Summary

Run the NanOlympicsMod containerized Nextflow pipeline to detect m6A sites on Nanopore direct RNA-seq (dRNA-seq) data with multiple detection tools (Yanocomp, ELIGOS, Nanocompore, DiffErr, etc.), and benchmark hits against orthogonal reference m6A peak sets.

## When to use

Use when base-called Nanopore fast5 dRNA-seq data (e.g. SK1 yeast, mouse, or human samples) must be profiled for m6A sites and detection tools need to be compared or applied under controlled thresholds.

## When NOT to use

- Input is already a short-read methylation dataset (this workflow is specific to ONT direct RNA fast5 data)
- The goal is calling modifications other than m6A at stoichiometry level with no comparison to reference peaks, since this skill centers on m6A detection and benchmarking
- Data are unbasecalled raw signal only and no basecalling step (--fast5_slot for basecalled bases) can be performed

## Inputs

- Base-called ONT fast5 dRNA-seq read folders per sample
- Tab-separated samples file (sample name, condition, path to fast5 folder)
- Reference genome/transcriptome fasta and index (e.g. SK1 yeast strain)
- Genes-to-transcripts mapping and bed annotation files
- m6A gold-standard peaks bed file for benchmarking (--peaksfile)

## Outputs

- Per-tool m6A hit bins/sites per condition with significance statistics
- Benchmarking outputs (precision, recall, F1, AUPRC) against orthogonal reference m6A sets
- Post-processed R analysis summaries of hits across tools and threshold sets

## How to apply

Clone the NanOlympicsMod repository, build a tab-separated samples file listing sample name, condition and path to the base-called fast5 folder, and configure pipeline.conf with --samples, --test_condition (e.g. 'WT'), the reference genome/transcriptome fasta (e.g. SK1 yeast), annotation files (genes-to-transcripts, bed peaks), and per-tool significance thresholds (yanocomp FDR, drummer p-value, differr FDR) plus the 'default' or 'relaxed' m6A-site filtering threshold set via --threshold. Execute with 'nextflow -c pipeline.conf run pipeline.nf --samples=/path/to/samples.txt --resultsDir=/path/to/resultsDir'; multi-sample tools such as ELIGOS, Yanocomp, DiffErr and Nanocompore can receive replicates either as separate samples or merged, with merged-vs-separate overlap typically exceeding 59%. Post-process runs with the repository's R scripts, and benchmark per-tool hits against gold-standard m6A peaks (bed file) lifted over to the reference genome, tuning precision-recall via stringency cut-offs and coverage-aware filtering (e.g. restricting to RRACH motifs, high-coverage or highly expressed bins).

## Related tools

- **NanOlympicsMod** (Containerized Nextflow pipeline that orchestrates execution and comparison of 14 m6A detection tools on dRNA-seq data) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine used to run pipeline.nf with pipeline.conf configuration and containerized reproducibility) — https://github.com/mfurla/NanOlympicsMod
- **Yanocomp** (Multi-sample m6A detection tool executed within the pipeline, thresholded by FDR; performed well on the yeast dataset) — https://github.com/mfurla/NanOlympicsMod
- **ELIGOS** (Multi-sample m6A detection tool executed within the pipeline, benefiting from higher coverage in the yeast dataset) — https://github.com/mfurla/NanOlympicsMod
- **Nanocompore** (Multi-sample m6A detection tool executed within the pipeline; performed well on the yeast dataset) — https://github.com/mfurla/NanOlympicsMod
- **ont_fast5_api** (Interface for handling ONT fast5 files (single/multi-read conversion) feeding the pipeline's fast5 inputs) — https://github.com/nanoporetech/ont_fast5_api

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- Hits from merged-replicate vs separate-replicate configurations overlap by more than 59% per tool, and over 99% for four out of six replicate-capable tools
- Per-tool hit counts are extracted consistently from both runs via the repository's R post-processing scripts
- Precision/recall (F1, AUPRC) computed against the m6A gold-standard peaks bed file change predictably when stringency cut-offs are tuned
- The pipeline completes for the included tools without empty output files, which flagged failed runs in the original benchmark
- Restricting to highly expressed bins boosts precision and recall, whereas RRACH-motif filtering alone should not be expected to

## Limitations

- Several tools could not complete the benchmark runs due to lack of progress or unexpected generation of empty output files
- The full dataset is computationally heavy: more than 22 million fast5 files totalling 4 TB of raw data
- Precision and recall are markedly influenced by sequencing depth; additional sequencing would likely reveal additional m6A sites
- For biological datasets (yeast, mouse, human) there is no true ground truth, so reference m6A sets come from orthogonal short-read methods and may carry bias relative to nanopore-based techniques
- Tools trained on synthetic oligos or other species (e.g. m6Anet on human, EpiNano-SVM, Nanom6A) show reduced performance on yeast due to training-set mismatch

## Evidence

- [abstract] We developed NanOlympicsMod, a Nextflow pipeline exploiting containerized technology for comparing 14 tools for m6A detection on dRNA-seq data.: "We developed NanOlympicsMod, a Nextflow pipeline exploiting containerized technology for comparing 14 tools for m6A detection on dRNA-seq data."
- [readme] Path to the tab-separated sample file including sample name, condition and path to base-called fast5 folder: "--samples                                                Path to the tab-separated sample file including sample name, condition and path to base-called fast5 folder"
- [methods] Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed'): "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
- [discussion] The multi-sample tools, such as ELIGOS, Yanocomp, DiffErr and Nanocompore performed better in the yeast dataset, probably benefiting from the higher coverage.: "The multi-sample tools, such as ELIGOS, Yanocomp, DiffErr and Nanocompore performed better in the yeast dataset, probably benefiting from the higher coverage."
- [discussion] Only by filtering for highly expressed bins significatively boosted precision and recall.: "Only by filtering for highly expressed bins significatively boosted precision and recall."
