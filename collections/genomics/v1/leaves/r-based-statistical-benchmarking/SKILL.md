---
name: r-based-statistical-benchmarking
description: 'Use when you need to reproduce or extend the benchmark of m6A detection
  tools on dRNA-seq data: tool BED calls and a gold-standard peaks BED exist, and
  per-tool-per-dataset TP/FP/FN-derived precision, recall and F1 at ''default'' thresholds
  must be computed.'
when_to_use_negative:
- When the goal is precision-recall curve analysis across stringency levels (e.g.
  AUPRC) rather than performance at the default cut-offs — use the PR-curve analysis
  instead
- When benchmarking on the synthetic oligos dataset where m6A positions are known
  by design rather than derived from orthogonal short-read reference peaks
- When tool runs did not complete (abandoned due to lack of progress or empty output
  files) — no valid call BED exists to benchmark
edam_operation: http://edamontology.org/operation_3480
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline providing the containerized R post-processing and statistical_analysis
    scripts that compute TP/FP/FN and precision/recall/F1 from tool calls vs reference
    peaks
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine executing the NanOlympicsMod benchmarking pipeline with containerized
    reproducibility
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:c6e9945cb4719daf654232b60bd1e19c7ebee3ecfd262bb58ac9613c75d5e023
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  edam_operation: http://edamontology.org/operation_3480
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# r-based-statistical-benchmarking

## Summary

Compute per-tool precision, recall and F1 at default significance cut-offs for Nanopore dRNA-seq m6A detection tools, by overlapping tool BED calls with orthogonal reference m6A peak sets in binned genomic coordinates, using the R post-processing and statistical_analysis scripts of the NanOlympicsMod Nextflow pipeline.

## When to use

Use when you need to reproduce or extend the benchmark of m6A detection tools on dRNA-seq data: tool BED calls and a gold-standard peaks BED exist, and per-tool-per-dataset TP/FP/FN-derived precision, recall and F1 at 'default' thresholds must be computed.

## When NOT to use

- When the goal is precision-recall curve analysis across stringency levels (e.g. AUPRC) rather than performance at the default cut-offs — use the PR-curve analysis instead
- When benchmarking on the synthetic oligos dataset where m6A positions are known by design rather than derived from orthogonal short-read reference peaks
- When tool runs did not complete (abandoned due to lack of progress or empty output files) — no valid call BED exists to benchmark

## Inputs

- Per-tool BED files of m6A calls produced by the NanOlympicsMod pipeline (task_001 outputs)
- Gold-standard m6A peaks BED file supplied via the --peaksfile argument
- pipeline.conf configuration with per-tool default thresholds (nanom6A probability, yanocomp FDR, differr FDR, drummer P-value, epinano error-sum/residual)
- Bin length parameter (--binLength) defining genomic bin size for overlap
- Lift-over mapping for yeast reference peaks to the SK1 reference genome

## Outputs

- Per-tool-per-dataset precision/recall/F1 table written as pr_f1_benchmark.tsv in the results directory
- Binned confidence-score matrix across all tools for downstream restriction analyses (RRACH motif, high-coverage, exclusion-zone bins)

## How to apply

Load the per-tool BED call files produced upstream (task_001) together with the gold-standard m6A peaks BED supplied via the --peaksfile argument, lift-over-ing yeast reference peaks to the SK1 genome where required. Overlap tool calls with reference m6A positions within genomic bins of the size set by --binLength, and restrict calls to the 'default' threshold set, applying each tool's parameter as configured in pipeline.conf (nanom6A probability threshold, yanocomp FDR, differr FDR, drummer P-value, epinano error-sum/residual thresholds). Count TP/FP/FN per tool per dataset and compute precision, recall and F1 using the pipeline's R post-processing and statistical_analysis scripts. The binned confidence-score matrix is also assembled for downstream restriction analyses (RRACH motif, high-coverage, exclusion-zone bins). Results are written as a per-tool-per-dataset precision/recall/F1 table (pr_f1_benchmark.tsv) plus the score matrix in the results directory; note that reference sets for yeast, mouse and human derive from orthogonal short-read methods, not true ground truth.

## Related tools

- **NanOlympicsMod** (Nextflow pipeline providing the containerized R post-processing and statistical_analysis scripts that compute TP/FP/FN and precision/recall/F1 from tool calls vs reference peaks) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine executing the NanOlympicsMod benchmarking pipeline with containerized reproducibility) — https://github.com/mfurla/NanOlympicsMod

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- Exactly one precision/recall/F1 row exists for every completed tool x dataset combination
- Every reference-set bin is represented in the binned confidence-score matrix
- Output file pr_f1_benchmark.tsv is present in the configured --resultsDir
- Only the 'default' threshold set was used: calls restricted per pipeline.conf (nanom6A probability threshold, yanocomp FDR, differr FDR, drummer P-value, epinano error-sum/residual thresholds)
- Yeast reference peaks were successfully lifted over to the SK1 genome before overlap counting

## Limitations

- Yeast, mouse and human datasets lack ground truth; reference m6A sets come from orthogonal short-read methods, and possible differences in bias between those reference sets and nanopore-based techniques confound performance assessment
- Several tools could not complete the benchmark runs — for a few, runs were abandoned due to lack of progress or unexpected generation of empty output files, leaving gaps in the tool x dataset matrix
- At default settings on the biological datasets no tool achieved both high precision and high recall; performance was significantly worse than on the synthetic oligos dataset, especially in recall

## Evidence

- [methods] Overlapping tool calls with reference m6A positions within genomic bins: "--peaksfile Path to bed file with set of m6A gold-standard peaks"
- [methods] Default threshold filtering of m6A calls: "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
- [discussion] Performance worse on biological datasets than oligos: "the tools performed very well on the synthetic oligos dataset, while the yeast, mouse and human datasets represented a more challenging task"
- [discussion] No ground truth in biological datasets: "The yeast, mouse and human datasets lack a ground truth, but fully represent in vivo conditions"
- [readme] Pipeline structure and scripts: "Scripts: folder containing a bash script to run the nextflow pipeline, a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1"
