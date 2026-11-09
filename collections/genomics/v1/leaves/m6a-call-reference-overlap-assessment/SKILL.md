---
name: m6a-call-reference-overlap-assessment
description: Use when per-tool m6A BED call files already exist (upstream tool runs)
  and the goal is to score each tool against a gold-standard peaks BED at default
  thresholds (nanom6A probability, yanocomp/differr FDR, drummer P-value, epinano
  error thresholds) on yeast, mouse or human dRNA-seq data.
when_to_use_negative:
- Working from raw FAST5/POD5 data without completed per-tool m6A call files — the
  tool runs (task_001) must exist first
- Evaluating tuned or non-default cut-offs (e.g. the 'relaxed' threshold set or PR-curve
  sweeps) — this skill is restricted to the 'default' threshold set only
- Benchmarking on the synthetic oligos dataset where m6A positions are known by design
  and the in vivo reference-set biases do not apply
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3569
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline that orchestrates the reference-overlap benchmarking, supplies
    --peaksfile, --binLength and --threshold 'default' filtering, and writes pr_f1_benchmark.tsv
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine executing the containerized NanOlympicsMod steps (overlap,
    counting, R post-processing and statistical analysis)
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:81fe3b005a449d0a8fba10e02b1587b017b13b69c868cfe5f71bda4f1dd2f6cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3569
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# m6a-call-reference-overlap-assessment

## Summary

Benchmark Nanopore dRNA-seq m6A detection tools at their default significance cut-offs by overlapping per-tool BED calls against orthogonal short-read reference m6A peak sets within genomic bins, computing per-tool per-dataset precision, recall and F1. Implemented inside the NanOlympicsMod Nextflow pipeline (task_003).

## When to use

Use when per-tool m6A BED call files already exist (upstream tool runs) and the goal is to score each tool against a gold-standard peaks BED at default thresholds (nanom6A probability, yanocomp/differr FDR, drummer P-value, epinano error thresholds) on yeast, mouse or human dRNA-seq data.

## When NOT to use

- Working from raw FAST5/POD5 data without completed per-tool m6A call files — the tool runs (task_001) must exist first
- Evaluating tuned or non-default cut-offs (e.g. the 'relaxed' threshold set or PR-curve sweeps) — this skill is restricted to the 'default' threshold set only
- Benchmarking on the synthetic oligos dataset where m6A positions are known by design and the in vivo reference-set biases do not apply

## Inputs

- Per-tool m6A call BED files from completed tool x dataset runs (task_001 outputs)
- Gold-standard m6A peaks BED file (--peaksfile argument)
- Yeast SK1 reference genome lift-over script (python) for yeast reference peaks
- pipeline.conf with per-tool default threshold parameters
- Bin length parameter (--binLength) defining genomic bin size

## Outputs

- pr_f1_benchmark.tsv: per-tool-per-dataset precision/recall/F1 table
- Binned confidence-score matrix across all tools for downstream restriction analyses (RRACH, high-coverage, exclusion-zone bins)
- Per-tool per-dataset TP/FP/FN counts

## How to apply

Load the per-tool BED call files produced upstream (task_001) together with the gold-standard peaks BED supplied via the --peaksfile argument, lifting yeast reference peaks to the SK1 genome with the provided lift-over python script. Overlap tool calls with reference m6A positions within genomic bins of the size set by --binLength, then restrict calls to the 'default' threshold set chosen via --threshold ('default' vs 'relaxed'), applying each tool's parameter as configured in pipeline.conf (nanom6A probability threshold, yanocomp FDR, differr FDR, drummer P-value, epinano error-sum/residual thresholds). Count TP/FP/FN per tool per dataset and compute precision, recall and F1 using the post-processing and statistical_analysis R scripts. Assemble the full binned confidence-score matrix across all tools for downstream restriction analyses (RRACH motif bins, high-coverage bins, exclusion-zone bins). Write the per-tool-per-dataset precision/recall/F1 table and score matrix to the results directory as pr_f1_benchmark.tsv.

## Related tools

- **NanOlympicsMod** (Nextflow pipeline that orchestrates the reference-overlap benchmarking, supplies --peaksfile, --binLength and --threshold 'default' filtering, and writes pr_f1_benchmark.tsv) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine executing the containerized NanOlympicsMod steps (overlap, counting, R post-processing and statistical analysis)) — https://github.com/mfurla/NanOlympicsMod

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples=/path/to/samples.txt --resultsDir=/path/to/resultsDir --peaksfile=/path/to/m6A_gold_standard.bed --threshold default
```

## Evaluation signals

- One precision/recall/F1 row exists for every completed tool x dataset combination in pr_f1_benchmark.tsv
- Every reference-set bin is represented in the binned confidence-score matrix
- TP + FP counts equal the number of tool calls retained after the 'default' threshold restriction
- F1 values are consistent with the independently reported precision and recall per tool x dataset row
- Yeast reference peaks are confirmed lifted to the SK1 genome before overlap counting

## Limitations

- No ground truth exists for m6A positions in the yeast, mouse and human datasets; reference sets were derived from orthogonal short-read methods, and possible differences in bias between the orthogonal reference sets and nanopore-based techniques confound performance assessment
- No tool achieved both high precision and high recall at default settings on the biological datasets; performance was significantly worse than on the synthetic oligos dataset, especially in recall
- Several tools could not complete the benchmark runs (lack of progress or empty output files), so not every tool x dataset combination yields a precision/recall/F1 row

## Evidence

- [results] At default settings on the biological datasets, no tool achieved both high precision and high recall; performance was significantly worse than on the synthetic oligos dataset, especially in terms of recall.: [evidence span withheld — anchor status: not_found]
- [methods] Tools were benchmarked against orthogonal reference sets; overlap logic grounded in methods argument.: "--peaksfile Path to bed file with set of m6A gold-standard peaks"
- [methods] Threshold filtering between 'default' and 'relaxed' is defined as a methods-level option.: "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
- [readme] The pipeline provides the SK1 lift-over script for yeast reference peaks.: "a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1 reference genome"
- [intro] Results were benchmarked against orthogonal reference m6A sets, grounding the overlap approach.: "The results were benchmarked against reference m6A sets obtained from established orthogonal techniques."
