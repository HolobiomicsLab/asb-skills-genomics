---
name: m6a-detection-tool-benchmarking
description: Use when comparing m6A detection tools on Nanopore direct RNA-seq data,
  tuning 'default' vs 'relaxed' thresholds, or assessing whether sequencing depth
  (e.g. 25/50/75% subsampling) saturates m6A hit discovery.
when_to_use_negative:
- Data is short-read m6A-seq (e.g. MeRIP-seq/miCLIP) rather than Nanopore direct RNA-seq
  signal-based data — NanOlympicsMod tools operate on FAST5/dRNA inputs
- A single pre-validated tool with a fixed threshold is all that is needed, with no
  cross-tool comparison or precision-recall tuning required
- Only synthetic oligo results matter and in vivo confounders are irrelevant — the
  article shows tools perform far better on synthetic oligos than on yeast/mouse/human
  data, so oligo-only benchmarking overestimates in vivo performance
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3563
tools:
- name: NanOlympicsMod
  role: Containerized Nextflow pipeline that executes and compares 14 m6A detection
    tools on dRNA-seq data and produces per-tool benchmark metrics
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine running pipeline.nf with pipeline.conf to orchestrate tool
    execution per sample/depth
  repo: https://github.com/mfurla/NanOlympicsMod
- name: ont_fast5_api
  role: Python interface and console scripts (single_to_multi_fast5, multi_to_single_fast5,
    fast5_subset) to handle multi-read/single-read FAST5 conversion and read extraction
    for subsampling
  repo: https://github.com/nanoporetech/ont_fast5_api
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:59582a078cb9e3d88cfc741bb6e922ba8cb0852d9371b3861b51f80fb29efefd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  - ont_fast5_api
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3563
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# m6A-detection-tool-benchmarking

## Summary

Benchmark m6A detection tools on Oxford Nanopore dRNA-seq data with the NanOlympicsMod Nextflow pipeline, scoring each tool's called sites against gold-standard m6A peak sets by hits, default-condition F1, and AUPRC, and probing how coverage depth affects precision/recall.

## When to use

Use when comparing m6A detection tools on Nanopore direct RNA-seq data, tuning 'default' vs 'relaxed' thresholds, or assessing whether sequencing depth (e.g. 25/50/75% subsampling) saturates m6A hit discovery.

## When NOT to use

- Data is short-read m6A-seq (e.g. MeRIP-seq/miCLIP) rather than Nanopore direct RNA-seq signal-based data — NanOlympicsMod tools operate on FAST5/dRNA inputs
- A single pre-validated tool with a fixed threshold is all that is needed, with no cross-tool comparison or precision-recall tuning required
- Only synthetic oligo results matter and in vivo confounders are irrelevant — the article shows tools perform far better on synthetic oligos than on yeast/mouse/human data, so oligo-only benchmarking overestimates in vivo performance

## Inputs

- Basecalled FAST5 folders from ONT dRNA-seq (samples.tsv with sample name, condition, path to FAST5)
- Read sets subsampled to 25%, 50%, 75% and 100% of full PromethION depth
- m6A gold-standard peaks bed file (--peaksfile) from orthogonal methods
- pipeline.conf Nextflow configuration (resultsDir, fast5_slot, tombo_slot, threshold)

## Outputs

- Per-tool m6A hit counts, default-condition F1 scores and AUPRC at each coverage depth
- Saturation table and plots of hits/F1/AUPRC versus coverage depth
- Precision-recall curves per tool with 'default' and 'relaxed' threshold settings

## How to apply

Retrieve dRNA-seq reads (e.g. human PromethION data from SRA BioProject PRJNA995902, yeast SK1 or mouse datasets) and clone NanOlympicsMod from github.com/mfurla/NanOlympicsMod. Run the containerized pipeline with `nextflow -c pipeline.conf run pipeline.nf --samples=<samples.tsv> --resultsDir=<results_dir>`, where samples.tsv lists sample name, condition (e.g. 'WT') and the basecalled FAST5 folder, setting the FAST5 and Tombo slots in pipeline.conf. For the saturation analysis, randomly subsample reads to 25%, 50% and 75% of the full read set (100% as reference depth) and execute the pipeline per depth. Filter called m6A sites with the '--threshold' option choosing between 'default' and 'relaxed' cut-offs so F1 is reported at default conditions across depths. Run the post-processing and statistical_analysis R scripts to compare each tool's hits against the m6A gold-standard peaks bed file ('--peaksfile'), computing per-tool hit counts, default-condition F1, and area under the precision-recall curve at each coverage depth; assemble results into a saturation table and plot hits, F1 and AUPRC versus coverage.

## Related tools

- **NanOlympicsMod** (Containerized Nextflow pipeline that executes and compares 14 m6A detection tools on dRNA-seq data and produces per-tool benchmark metrics) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine running pipeline.nf with pipeline.conf to orchestrate tool execution per sample/depth) — https://github.com/mfurla/NanOlympicsMod
- **ont_fast5_api** (Python interface and console scripts (single_to_multi_fast5, multi_to_single_fast5, fast5_subset) to handle multi-read/single-read FAST5 conversion and read extraction for subsampling) — https://github.com/nanoporetech/ont_fast5_api

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples=/path/to/samples.txt --resultsDir=/path/to/resultsDir --threshold=default
```

## Evaluation signals

- Per-tool hit counts and AUPRC increase monotonically with sequencing depth for all tools except DiffErr (opposite hit trend) and EpiNano-SVM AUPRC as expected exceptions
- Precision and recall are markedly influenced by sequencing depth, consistent with the finding that additional sequencing would likely reveal additional m6A sites
- Default-condition F1 is reproducible when --threshold is fixed to 'default' across all depths, enabling direct cross-depth comparison
- Multi-sample tools (ELIGOS, Yanocomp, DiffErr, Nanocompore) should rank higher on the high-coverage yeast dataset, mirroring the paper's reported pattern

## Limitations

- Yeast, mouse and human datasets lack a true ground truth; reference m6A sets derive from orthogonal short-read methods, and bias differences between orthogonal references and nanopore-based tools confound performance assessment
- Several tools could not complete benchmark runs — abandoned due to lack of progress or unexpected empty output files
- High computational burden: >22 M files totalling 4 TB of raw data for the human PromethION run
- m6Anet uses a default neural network trained on human data and was not re-trained for yeast/mouse; EpiNano-SVM and Nanom6A were trained on synthetic oligos lacking real transcriptome complexity

## Evidence

- [abstract] NanOlympicsMod, a Nextflow pipeline exploiting containerized technology for comparing 14 tools for m6A detection on dRNA-seq data: "We developed NanOlympicsMod, a Nextflow pipeline exploiting containerized technology for comparing 14 tools for m6A detection on dRNA-seq data."
- [abstract] precision and recall of tools are markedly influenced by sequencing depth: "precision and recall of tools are markedly influenced by sequencing depth, and that additional sequencing would likely reveal additional m6A sites."
- [discussion] evaluated the tools' performances ... at different coverage depth values: [evidence span withheld — anchor status: not_found]
- [methods] --threshold Set of thresholds to use for the filtering of m6A sites: "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
- [readme] usage line for the Nextflow pipeline: "nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir""
