---
name: r-statistical-postprocessing
description: Use when benchmarking dRNA-seq m6A detection tool outputs against gold-standard
  peak BED files and you need PR curves, AUPRC with 95% CIs, or pairwise concordance
  heatmaps across tools and datasets.
when_to_use_negative:
- The task is raw signal processing or m6A calling itself — this skill only handles
  post-processing of already-generated per-tool calls/scores
- The dataset lacks a gold-standard m6A peak set, so positive bins cannot be defined
  (yeast/mouse/human reference sets are orthogonal short-read-derived, not true ground
  truth)
- You need per-tool hit counts, F1 at default conditions, or saturation analysis rather
  than PR-curve/AUPRC or concordance statistics
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: NanOlympicsMod
  role: Supplies the R post-processing and statistical-analysis scripts in its Scripts
    folder for PR-curve, AUPRC-CI, and concordance computation
  repo: https://github.com/mfurla/NanOlympicsMod
- name: R
  role: Language for the post-processing scripts; PRROC computes precision-recall
    curves and AUPRC with 95% confidence intervals
- name: Nextflow
  role: Pipeline engine (pipeline.nf/pipeline.conf) that orchestrates the binned tool
    outputs feeding the R analysis
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:7f072fdc255cec5869b5e6b0ce27a1f759bf5b12544eb5b4c71c3be2630095eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  - R
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# r-statistical-postprocessing

## Summary

R-based post-processing and statistical analysis layer of the NanOlympicsMod pipeline: computes precision-recall curves, AUPRC with 95% confidence intervals (via PRROC), and pairwise tool concordance matrices over binned genomic query spaces for 14 dRNA-seq m6A detection tools.

## When to use

Use when benchmarking dRNA-seq m6A detection tool outputs against gold-standard peak BED files and you need PR curves, AUPRC with 95% CIs, or pairwise concordance heatmaps across tools and datasets.

## When NOT to use

- The task is raw signal processing or m6A calling itself — this skill only handles post-processing of already-generated per-tool calls/scores
- The dataset lacks a gold-standard m6A peak set, so positive bins cannot be defined (yeast/mouse/human reference sets are orthogonal short-read-derived, not true ground truth)
- You need per-tool hit counts, F1 at default conditions, or saturation analysis rather than PR-curve/AUPRC or concordance statistics

## Inputs

- Binned per-tool confidence-score matrices from upstream genome binning (--binLength windows)
- BED file of m6A gold-standard peaks (--peaksfile) for bin labeling
- Per-tool m6A site BED files produced by the NanOlympicsMod pipeline
- Per-tool swept significance cut-offs: nanom6A probability thresholds, Yanocomp/DiffErr FDR thresholds, DRUMMER p-value threshold, EpiNano error thresholds

## Outputs

- One PR curve figure per tool/dataset
- AUPRC-with-95%-CI summary table (auprc_confidence_intervals.tsv)
- Pairwise concordance matrix table and heatmap per dataset (oligos, yeast, mouse, human)

## How to apply

Load the binned per-tool confidence-score matrices produced by genome binning (--binLength; 5 nt bins for the synthetic oligos dataset, 50 nt bins for yeast, mouse and human) and the gold-standard m6A peaks BED file ('--peaksfile') to label each bin positive or negative. For each tool, sweep its native significance cut-off (nanom6A probability thresholds; Yanocomp and DiffErr FDR thresholds; DRUMMER p-value threshold; EpiNano error-threshold sum/residuals) plus the 'default' and 'relaxed' threshold sets (--threshold) to explore the precision-recall space and tune the precision-recall trade-off. Compute precision and recall at each swept threshold with the R post-processing and statistical-analysis scripts in the NanOlympicsMod Scripts folder, using PRROC, and derive AUPRC point estimates with 95% confidence intervals per tool-dataset pair. When comparing tools pairwise, mark a bin m6A-positive when at least one of a tool's calls falls inside it, compute the overlap matrix, and restrict the average-overlap summary to tools with >1000 called sites. Interpret CI width in light of positive-bin counts: 1543 (oligos) and 1453 (yeast) yield wide CIs, while 84060 (mouse) and 18932 (human) give narrower CIs and more frequent tool overlaps.

## Related tools

- **NanOlympicsMod** (Supplies the R post-processing and statistical-analysis scripts in its Scripts folder for PR-curve, AUPRC-CI, and concordance computation) — https://github.com/mfurla/NanOlympicsMod
- **R** (Language for the post-processing scripts; PRROC computes precision-recall curves and AUPRC with 95% confidence intervals)
- **Nextflow** (Pipeline engine (pipeline.nf/pipeline.conf) that orchestrates the binned tool outputs feeding the R analysis) — https://github.com/mfurla/NanOlympicsMod

## Evaluation signals

- auprc_confidence_intervals.tsv contains AUPRC point estimates with finite 95% CI bounds for every tool-dataset pair (14 tools x 4 datasets)
- CIs are wider for the low-positive-count oligos (1543 positives) and yeast (1453 positives) datasets than for mouse (84060) and human (18932)
- Average pairwise overlap among oligos tools with >1000 called sites approximates 0.93
- Exactly one concordance matrix table/heatmap per dataset is present in the results directory
- Restricting average-overlap summaries to tools with >1000 sites excludes sparse callers from the reported statistic

## Limitations

- Yeast, mouse and human datasets lack ground truth; reference m6A sets come from orthogonal short-read methods, and possible differences in bias between those references and nanopore techniques confound performance assessment
- The oligos dataset is artificial and poorly recapitulates the relative location and frequency of in vivo marks, so PR statistics there may not transfer to biological samples
- AUPRC confidence intervals are large when the number of positive bins is small (1543 in oligos, 1453 in yeast), limiting resolution between tools

## Evidence

- [readme] R scripts folder: "a bash script to run the nextflow pipeline, a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1 reference genome"
- [abstract] PR performance differs across datasets: "The performance of the tools markedly differed across datasets, and methods adopting different approaches showed different preferences in terms of precision and recall."
- [abstract] cut-off tuning: "Changing the stringency cut-offs allowed for tuning the precision-recall trade-off towards user preferences."
- [methods] gold-standard peaks input: "--peaksfile Path to bed file with set of m6A gold-standard peaks"
