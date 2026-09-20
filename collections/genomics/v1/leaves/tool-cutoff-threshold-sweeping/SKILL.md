---
name: tool-cutoff-threshold-sweeping
description: Use when benchmarking m6A callers on binned Nanopore dRNA-seq predictions
  against gold-standard peak BED labels and you need full PR curves/AUPRC rather than
  a single fixed-threshold score.
when_to_use_negative:
- You only need a single operating point (e.g. F1 score at default conditions) rather
  than the full precision-recall trade-off curve
- No gold-standard m6A peak BED file exists to label bins as positive/negative (e.g.
  yeast/mouse/human reference sets from orthogonal methods are unavailable)
- The input is raw FAST5/FASTQ signal rather than already-binned per-tool confidence
  scores from the NanOlympicsMod genome-binning step
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline that executes the 14 m6A tools and supplies the R post-processing
    scripts for threshold sweeping and PR analysis
  repo: https://github.com/mfurla/NanOlympicsMod
- name: PRROC
  role: R package used to compute precision and recall at each swept threshold and
    AUPRC with 95% confidence intervals
- name: Yanocomp
  role: Tool whose FDR threshold is swept across its precision-recall space
- name: DRUMMER
  role: Tool whose p-value threshold is swept across its precision-recall space
- name: EpiNano
  role: Tool whose error-threshold-sum and residual thresholds are swept for PR curve
    plotting
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:36b829c1c2fab6d18b369347a111f2f93e1b9181149fa653244d2f3e50e2121c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - DRUMMER
  - EpiNano
  - NanOlympicsMod
  - PRROC
  - Yanocomp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# tool-cutoff-threshold-sweeping

## Summary

Sweeps each m6A detection tool's native significance cut-off (e.g. nanom6A probability thresholds, Yanocomp FDR, DRUMMER p-value, EpiNano error thresholds, plus the pipeline's 'default' and 'relaxed' sets) across binned dRNA-seq predictions to trace full precision-recall curves and AUPRC with 95% CIs, enabling tuning of the precision-recall trade-off.

## When to use

Use when benchmarking m6A callers on binned Nanopore dRNA-seq predictions against gold-standard peak BED labels and you need full PR curves/AUPRC rather than a single fixed-threshold score.

## When NOT to use

- You only need a single operating point (e.g. F1 score at default conditions) rather than the full precision-recall trade-off curve
- No gold-standard m6A peak BED file exists to label bins as positive/negative (e.g. yeast/mouse/human reference sets from orthogonal methods are unavailable)
- The input is raw FAST5/FASTQ signal rather than already-binned per-tool confidence scores from the NanOlympicsMod genome-binning step

## Inputs

- Binned per-tool confidence-score matrix from genome binning (window length set by --binLength)
- BED file of m6A gold-standard peaks (--peaksfile) for oligos, yeast (SK1 liftover), mouse and human datasets
- Tool-specific scoring outputs: nanom6A probabilities, Yanocomp/DiffErr FDR, DRUMMER p-values, EpiNano error-sum/residual values

## Outputs

- One precision-recall curve figure file per tool/dataset
- AUPRC point estimates with 95% confidence intervals per tool per dataset (auprc_confidence_intervals.tsv)

## How to apply

Load the per-tool binned confidence-score matrix produced by genome binning with the pipeline's --binLength window parameter, then label each bin positive/negative using the m6A gold-standard peaks BED file (--peaksfile). For each of the 14 tools, sweep its native cut-off parameter: nanom6A probability thresholds, Yanocomp and DiffErr FDR thresholds, DRUMMER p-value, EpiNano error-sum and residual thresholds, and the pipeline's 'default' and 'relaxed' threshold sets (--threshold). Compute precision and recall at every swept threshold with the PRROC R package via the NanOlympicsMod post-processing R scripts, then derive AUPRC point estimates and 95% confidence intervals per tool per dataset. Interpret CI width relative to positive-bin count: oligos (1543) and yeast (1453) yield wide CIs, while mouse (84,060) and human (18,932) yield narrower CIs with more tool overlap; also consider that cut-off settings should be tuned as a function of available coverage depth.

## Related tools

- **NanOlympicsMod** (Nextflow pipeline that executes the 14 m6A tools and supplies the R post-processing scripts for threshold sweeping and PR analysis) — https://github.com/mfurla/NanOlympicsMod
- **PRROC** (R package used to compute precision and recall at each swept threshold and AUPRC with 95% confidence intervals)
- **Yanocomp** (Tool whose FDR threshold is swept across its precision-recall space)
- **DRUMMER** (Tool whose p-value threshold is swept across its precision-recall space)
- **EpiNano** (Tool whose error-threshold-sum and residual thresholds are swept for PR curve plotting)

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples=/path/to/samples.txt --resultsDir=/path/to/resultsDir
```

## Evaluation signals

- auprc_confidence_intervals.tsv contains AUPRC point estimates with finite 95% CI bounds for every tool-dataset pair
- CIs are wider for low-positive-count oligos (1543) and yeast (1453) datasets than for mouse (84,060) and human (18,932)
- Tool overlap along the PR curve is more frequent for mouse/human (many positive bins) than for oligos/yeast
- Changing the stringency cut-off monotonically moves each tool along its PR curve, demonstrating the trade-off tuning

## Limitations

- AUPRC confidence intervals are large when the number of positive bins is small (oligos: 1543; yeast: 1453), limiting discriminative power between tools
- Yeast, mouse and human lack true ground truth; reference m6A sets come from orthogonal short-read methods, so bias differences confound performance assessment
- Several tools could not complete benchmark runs (lack of progress or empty output files), so the sweep may not cover all 14 tools on every dataset

## Evidence

- [abstract] Changing the stringency cut-offs allowed for tuning the precision-recall trade-off towards user preferences.: "Changing the stringency cut-offs allowed for tuning the precision-recall trade-off towards user preferences."
- [methods] --peaksfile Path to bed file with set of m6A gold-standard peaks: "--peaksfile Path to bed file with set of m6A gold-standard peaks"
- [methods] --threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed'): "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
- [readme] a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1 reference genome: "a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1 reference genome"
- [discussion] cut-off settings should be tuned also considering the available coverage depth: "cut-off settings should be tuned also considering the available coverage depth"
