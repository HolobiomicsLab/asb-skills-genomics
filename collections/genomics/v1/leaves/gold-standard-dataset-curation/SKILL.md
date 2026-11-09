---
name: gold-standard-dataset-curation
description: Use when benchmarking dRNA-seq m6A callers against orthogonal reference
  m6A positions in yeast (SK1), mouse or human datasets, requiring a curated BED gold-standard
  and per-tool TP/FP/FN accounting.
when_to_use_negative:
- Input is the synthetic oligos dataset, where m6A positions are known by design and
  do not need orthogonal gold-standard peaks
- You require a true ground truth rather than an orthogonal-method proxy — none exists
  for the yeast, mouse and human in vivo datasets
- Your goal is tuning stringency cut-offs or AUPRC/PR-curve analysis rather than fixed
  default-cut-off benchmarking (use the 'relaxed' threshold or PR-curve skill instead)
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline that containerizes the 14 m6A detection tools, applies --peaksfile
    gold-standard BEDs, --binLength binning and --threshold filtering, and runs the
    R post-processing/statistical_analysis scripts that compute precision, recall
    and F1
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine executing the containerized NanOlympicsMod pipeline configured
    via pipeline.conf
  repo: https://github.com/mfurla/NanOlympicsMod
- name: ont_fast5_api
  role: Supporting Oxford Nanopore FAST5 interface used upstream for handling basecalled
    dRNA-seq FAST5 data feeding the benchmark
  repo: https://github.com/nanoporetech/ont_fast5_api
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:ea577ed1a46afecb1b63eae5703d6faf718e172260580438a5c3a6c17941807a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  - ont_fast5_api
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# gold-standard-dataset-curation

## Summary

Assemble and apply gold-standard m6A peak sets (BED files from orthogonal short-read techniques, e.g. the --peaksfile argument in NanOlympicsMod) to benchmark Nanopore dRNA-seq m6A detection tools by TP/FP/FN counting and precision, recall and F1 computation at default significance cut-offs.

## When to use

Use when benchmarking dRNA-seq m6A callers against orthogonal reference m6A positions in yeast (SK1), mouse or human datasets, requiring a curated BED gold-standard and per-tool TP/FP/FN accounting.

## When NOT to use

- Input is the synthetic oligos dataset, where m6A positions are known by design and do not need orthogonal gold-standard peaks
- You require a true ground truth rather than an orthogonal-method proxy — none exists for the yeast, mouse and human in vivo datasets
- Your goal is tuning stringency cut-offs or AUPRC/PR-curve analysis rather than fixed default-cut-off benchmarking (use the 'relaxed' threshold or PR-curve skill instead)

## Inputs

- Per-tool m6A call BED files from upstream tool executions (task_001 outputs)
- Gold-standard m6A peaks BED file (--peaksfile) from orthogonal short-read methods
- Reference genome/transcriptome FASTA files (e.g. SK1 yeast strain) for lift-over and binning
- pipeline.conf with per-tool default threshold parameters (nanom6A probability, yanocomp FDR, differr FDR, drummer P-value, epinano thresholds)
- Binning parameters (--binLength) and threshold set selector (--threshold 'default' or 'relaxed')

## Outputs

- Per-tool-per-dataset precision/recall/F1 table written as pr_f1_benchmark.tsv
- Full binned confidence-score matrix across all tools for downstream restriction analyses (RRACH motif, high-coverage, exclusion-zone bins)
- TP/FP/FN counts per tool per dataset

## How to apply

Load the per-tool BED call files (produced upstream by the 14 m6A detection tools run through the NanOlympicsMod Nextflow pipeline) together with a gold-standard peaks BED supplied via the --peaksfile argument, lifting yeast reference peaks to the SK1 reference genome with the repository's Python lift-over script where required. Overlap tool calls with reference m6A positions within genomic bins of the size set by --binLength, and restrict calls to the 'default' threshold set, applying each tool's parameter as configured in pipeline.conf (nanom6A probability threshold, yanocomp FDR, differr FDR, drummer P-value, epinano error-sum/residual thresholds). Count true positives, false positives and false negatives per tool per dataset, and compute precision, recall and F1 with the post-processing and statistical_analysis R scripts. The rationale is that the yeast, mouse and human datasets lack nanopore ground truth, so orthogonal short-read-derived reference sets are used, accepting possible technique-specific bias. Because no tool achieved both high precision and high recall at default settings on biological data, the binned confidence-score matrix should also be emitted for downstream restriction analyses (RRACH motif, high-coverage, exclusion-zone bins).

## Related tools

- **NanOlympicsMod** (Nextflow pipeline that containerizes the 14 m6A detection tools, applies --peaksfile gold-standard BEDs, --binLength binning and --threshold filtering, and runs the R post-processing/statistical_analysis scripts that compute precision, recall and F1) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine executing the containerized NanOlympicsMod pipeline configured via pipeline.conf) — https://github.com/mfurla/NanOlympicsMod
- **ont_fast5_api** (Supporting Oxford Nanopore FAST5 interface used upstream for handling basecalled dRNA-seq FAST5 data feeding the benchmark) — https://github.com/nanoporetech/ont_fast5_api

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- One precision/recall/F1 row exists for every completed tool x dataset combination (tools that produced empty output or stalled are absent, not zero-filled)
- Every reference-set genomic bin is represented in the binned confidence-score matrix
- Calls are restricted to the 'default' threshold set with the exact per-tool parameters from pipeline.conf (e.g. yanocomp FDR, drummer P-value)
- Yeast gold-standard peaks have been lifted to the SK1 reference genome before overlap counting
- Computed F1 values are consistent with the reported TP/FP/FN counts (F1 = 2TP / (2TP + FP + FN)) and reproduce the finding that no tool achieved both high precision and high recall on biological datasets

## Limitations

- The yeast, mouse and human datasets lack ground truth; reference sets come from orthogonal short-read methods, and differences in bias between these and nanopore techniques may confound the performance assessment
- Several tools could not complete benchmark runs — abandoned due to lack of progress or unexpected empty output files — so the tool x dataset matrix is incomplete
- Low m6A stoichiometry in the biological datasets only partially explains reduced F1 scores; residual causes are unattributed

## Evidence

- [methods] --peaksfile Path to bed file with set of m6A gold-standard peaks: "--peaksfile Path to bed file with set of m6A gold-standard peaks"
- [methods] --threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed'): "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
- [discussion] The yeast, mouse and human datasets lack a ground truth, but fully represent in vivo conditions: "The yeast, mouse and human datasets lack a ground truth, but fully represent in vivo conditions"
- [discussion] the tools performed very well on the synthetic oligos dataset, while the yeast, mouse and human datasets represented a more challenging task: "the tools performed very well on the synthetic oligos dataset, while the yeast, mouse and human datasets represented a more challenging task"
- [readme] a python script to lift-over m6A peaks to SK1 reference genome: "a python script to lift-over m6A peaks to SK1 reference genome"
