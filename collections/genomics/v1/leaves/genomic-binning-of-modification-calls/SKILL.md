---
name: genomic-binning-of-modification-calls
description: Use when per-tool m6A calls in BED format must be scored against a gold-standard
  peaks BED to compute TP/FP/FN and precision/recall/F1 in genomic bins, e.g. at default
  significance cut-offs or restricted bin sets (RRACH, high-coverage, exclusion-zone
  bins).
when_to_use_negative:
- When the dataset has a true ground truth rather than an orthogonal short-read reference
  set (e.g. the synthetic oligos dataset, where m6A positions are known by design)
  — a direct position comparison is more appropriate
- When assessing coverage-depth effects or tuning stringency cut-offs across a precision-recall
  curve (AUPRC analysis) rather than scoring at a single default cut-off
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: NanOlympicsMod
  role: Containerized Nextflow workflow whose binning/benchmark step overlaps tool
    m6A calls with reference peaks per bin and computes precision/recall/F1
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Execution engine running pipeline.nf/pipeline.conf steps that perform the
    bin overlap and statistic computation
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:d23d61a9bf014ab1cc20b50407fa229386c2d8ba9a8922fdfd3d33f7e522cf41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# genomic-binning-of-modification-calls

## Summary

A benchmarking procedure from NanOlympicsMod that overlaps per-tool m6A calls from Nanopore dRNA-seq against orthogonal reference m6A peak sets within fixed genomic bins, yielding per-tool per-dataset precision, recall and F1. It converts raw BED modification calls into quantitative, comparable performance statistics.

## When to use

Use when per-tool m6A calls in BED format must be scored against a gold-standard peaks BED to compute TP/FP/FN and precision/recall/F1 in genomic bins, e.g. at default significance cut-offs or restricted bin sets (RRACH, high-coverage, exclusion-zone bins).

## When NOT to use

- When the dataset has a true ground truth rather than an orthogonal short-read reference set (e.g. the synthetic oligos dataset, where m6A positions are known by design) — a direct position comparison is more appropriate
- When assessing coverage-depth effects or tuning stringency cut-offs across a precision-recall curve (AUPRC analysis) rather than scoring at a single default cut-off

## Inputs

- Per-tool m6A call files in BED format produced by the 14 detection tools on dRNA-seq data (oligos, yeast SK1, mouse, human)
- Gold-standard reference m6A peaks BED file supplied via --peaksfile
- Bin length (--binLength) and threshold-set choice ('default' or 'relaxed') from pipeline.conf
- Per-tool threshold parameters (nanom6A probability, yanocomp FDR, diffErr FDR, drummer P-value, epinano error-sum/residual)

## Outputs

- Per-tool-per-dataset precision/recall/F1 table written as pr_f1_benchmark.tsv
- Binned confidence-score matrix across all tools for downstream restriction analyses
- TP/FP/FN counts per tool per dataset per bin

## How to apply

Load each tool's BED call file (produced upstream by the NanOlympicsMod Nextflow pipeline) and the gold-standard peaks BED supplied via the --peaksfile argument, first lifting yeast reference peaks to the SK1 genome with the repository's lift-over python script. Overlap tool calls with reference m6A positions within genomic bins of the size set by --binLength, restricting calls to the chosen threshold set via --threshold ('default' or 'relaxed'), which instantiates each tool's own parameter (nanom6A probability, yanocomp FDR, differr FDR, drummer P-value, epinano error-sum/residual thresholds). Count TP, FP and FN per tool per dataset and compute precision, recall and F1 using the post-processing and statistical_analysis R scripts. The same binned confidence-score matrix supports downstream restriction analyses (RRACH motif, high-coverage, exclusion-zone bins); the article found that only filtering for highly expressed bins significantly boosted precision and recall.

## Related tools

- **NanOlympicsMod** (Containerized Nextflow workflow whose binning/benchmark step overlaps tool m6A calls with reference peaks per bin and computes precision/recall/F1) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Execution engine running pipeline.nf/pipeline.conf steps that perform the bin overlap and statistic computation) — https://github.com/mfurla/NanOlympicsMod

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- Exactly one precision/recall/F1 row exists for every completed tool x dataset combination
- Every reference-set bin is represented in the binned confidence-score matrix
- Yeast reference peaks are confirmed lifted to the SK1 genome before overlap
- TP/FP/FN counts are internally consistent: precision = TP/(TP+FP), recall = TP/(TP+FN), F1 = 2*P*R/(P+R)
- pr_f1_benchmark.tsv is produced in the results directory with the full binned score matrix

## Limitations

- The yeast, mouse and human datasets lack a true ground truth; reference sets come from orthogonal short-read methods, and possible bias differences between those sets and nanopore-based techniques confound performance assessment
- No tool achieved both high precision and high recall at default settings on the biological datasets; performance was markedly worse than on the synthetic oligos dataset
- Several tools could not complete benchmark runs (abandoned for lack of progress or empty output files), so some tool x dataset combinations will be missing
- Only filtering for highly expressed bins significantly boosted precision and recall; RRACH-motif and exclusion-zone restrictions did not

## Evidence

- [methods] Gold-standard peaks BED file: "--peaksfile Path to bed file with set of m6A gold-standard peaks"
- [methods] Threshold-set choice: "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
- [methods] SK1 lift-over: "a python script to lift-over m6A peaks to SK1 reference genome"
- [discussion] High-coverage restriction: "Only by filtering for highly expressed bins significatively boosted precision and recall."
- [discussion] F1 at default conditions: [evidence span withheld — anchor status: not_found]
