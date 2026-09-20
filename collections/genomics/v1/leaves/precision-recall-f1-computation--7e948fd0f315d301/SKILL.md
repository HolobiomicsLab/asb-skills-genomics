---
name: precision-recall-f1-computation
description: Use when you need to benchmark m6A detection tool calls against a gold-standard
  peaks BED file, counting TP/FP/FN per bin to derive precision, recall and F1 at
  a chosen threshold set.
when_to_use_negative:
- When evaluating on the synthetic oligos dataset where m6A positions are known by
  design and per-position ground truth (not reference peaks) should be used instead
- When the analysis goal is precision-recall trade-off exploration across thresholds
  (AUPRC), not a single default cut-off comparison
- When the tool run did not complete or produced empty output files — no valid precision/recall/F1
  row can be computed
edam_operation: http://edamontology.org/operation_0417
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3511
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline orchestrating tool calls, bin overlaps and the post-processing/statistical_analysis
    R scripts that compute precision, recall and F1
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine executing the containerized NanOlympicsMod pipeline steps
    (pipeline.nf, pipeline.conf)
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:c4c0bedc76228cd57fe8e3cebbb55440ab12644030b445befa918f97f3a37799
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  edam_operation: http://edamontology.org/operation_0417
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# precision-recall-f1-computation

## Summary

Computes per-tool precision, recall and F1 by overlapping dRNA-seq m6A tool calls (at default significance cut-offs) against orthogonal short-read reference m6A peak sets within genomic bins, enabling quantitative benchmarking across datasets.

## When to use

Use when you need to benchmark m6A detection tool calls against a gold-standard peaks BED file, counting TP/FP/FN per bin to derive precision, recall and F1 at a chosen threshold set.

## When NOT to use

- When evaluating on the synthetic oligos dataset where m6A positions are known by design and per-position ground truth (not reference peaks) should be used instead
- When the analysis goal is precision-recall trade-off exploration across thresholds (AUPRC), not a single default cut-off comparison
- When the tool run did not complete or produced empty output files — no valid precision/recall/F1 row can be computed

## Inputs

- Per-tool m6A call BED files from upstream NanOlympicsMod runs (task_001)
- Gold-standard m6A reference peaks BED file (passed via --peaksfile)
- SK1 lift-over script output for yeast reference peaks
- pipeline.conf with per-tool default threshold parameters (nanom6A probability, yanocomp FDR, differr FDR, drummer P-value, epinano error-sum/residual)

## Outputs

- Per-tool-per-dataset precision/recall/F1 table written as pr_f1_benchmark.tsv
- TP/FP/FN counts per tool per dataset
- Full binned confidence-score matrix across all tools for downstream restriction analyses (RRACH motif, high-coverage, exclusion-zone bins)

## How to apply

Load the per-tool BED call files produced upstream and the gold-standard peaks BED supplied via the --peaksfile argument, lifting yeast reference peaks to the SK1 genome with the provided python script where needed. Overlap tool calls with reference m6A positions within genomic bins of the size set by --binLength, then restrict calls to the 'default' threshold set, applying each tool's configured parameter from pipeline.conf (nanom6A probability threshold, yanocomp FDR, differr FDR, drummer P-value, epinano error-sum/residual thresholds). Count TP/FP/FN per tool per dataset and compute precision, recall and F1 using the post-processing and statistical_analysis R scripts. Also assemble the full binned confidence-score matrix across all tools so downstream restriction analyses (RRACH motif, high-coverage, exclusion-zone bins) can be performed. The rationale is that orthogonal short-read reference sets stand in for missing ground truth in the yeast, mouse and human biological datasets, where default-setting performance was expected to be worse than on the synthetic oligos dataset, especially in recall.

## Related tools

- **NanOlympicsMod** (Nextflow pipeline orchestrating tool calls, bin overlaps and the post-processing/statistical_analysis R scripts that compute precision, recall and F1) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine executing the containerized NanOlympicsMod pipeline steps (pipeline.nf, pipeline.conf)) — https://github.com/mfurla/NanOlympicsMod

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples=/path/to/samples.txt --resultsDir=/path/to/resultsDir
```

## Evaluation signals

- One precision/recall/F1 row exists for every completed tool x dataset combination in pr_f1_benchmark.tsv
- Every reference-set bin is represented in the binned confidence-score matrix (no missing bins)
- TP/FP/FN counts satisfy the invariant TP + FN = number of reference m6A positions within the assessed bins
- Precision, recall and F1 values fall in [0, 1] and F1 is consistent with the harmonic mean of precision and recall
- Only calls belonging to the 'default' threshold set (per pipeline.conf parameters) are included in the counts

## Limitations

- No ground truth exists for the yeast, mouse and human datasets; orthogonal short-read reference sets may carry biases relative to nanopore-based m6A techniques, confounding performance assessment
- Several tools could not complete the benchmark runs (abandoned due to lack of progress or empty output files), so the tool x dataset matrix may be incomplete
- Performance at default settings was markedly worse on biological datasets than on the synthetic oligos dataset, especially recall, partly attributable to low m6A stoichiometry that is only partially explanatory

## Evidence

- [other] Overlap tool calls with reference m6A positions within genomic bins of the size set by --binLength: [evidence span withheld — anchor status: not_found]
- [methods] peaksfile argument: "--peaksfile Path to bed file with set of m6A gold-standard peaks"
- [intro] benchmarking against orthogonal reference sets: "The results were benchmarked against reference m6A sets obtained from established orthogonal techniques."
- [discussion] worse performance on biological datasets: "the tools performed very well on the synthetic oligos dataset, while the yeast, mouse and human datasets represented a more challenging task"
- [readme] lift-over of yeast peaks to SK1 genome: "a python script to lift-over m6A peaks to SK1 reference genome"
- [readme] threshold parameter descriptions: "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
