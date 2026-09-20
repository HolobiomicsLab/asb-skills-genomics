---
name: precision-recall-curve-evaluation
description: Use when per-tool m6A confidence scores have been assigned to fixed genome
  bins (via --binLength) and you need to quantify detection-tool performance (precision,
  recall, AUPRC, 95% CIs) against a gold-standard peaks bed file.
when_to_use_negative:
- Predictions are per-site rather than per-bin and have not been aggregated into fixed
  genome bins consistent with the gold-standard peaks annotation
- No orthogonal gold-standard m6A peak set exists for the organism/dataset, since
  PR labelling depends entirely on the peaks bed file
- The goal is saturation or coverage analysis rather than fixed-coverage precision-recall
  benchmarking
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline whose genome binning (--binLength), threshold filtering
    (--threshold) and R post-processing/statistical_analysis scripts generate the
    per-bin scores, PR curves and AUPRC CIs
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine that executes the containerized NanOlympicsMod pipeline steps
    producing the binned tool-score matrix
  repo: https://github.com/mfurla/NanOlympicsMod
- name: PRROC
  role: R package used within the statistical_analysis scripts to compute precision-recall
    curves and AUPRC at each swept threshold
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:a9f255810418c76edf1f238abe9277f5172526c6db4764a08322bc81f884d870
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  - PRROC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# precision-recall-curve-evaluation

## Summary

Sweep per-tool m6A confidence-score cut-offs on genome-binned dRNA-seq predictions, compute precision-recall curves and AUPRC with 95% confidence intervals against gold-standard m6A peaks, and compare 14 tools across oligos, yeast, mouse and human datasets using the NanOlympicsMod R post-processing and statistical_analysis scripts with PRROC.

## When to use

Use when per-tool m6A confidence scores have been assigned to fixed genome bins (via --binLength) and you need to quantify detection-tool performance (precision, recall, AUPRC, 95% CIs) against a gold-standard peaks bed file.

## When NOT to use

- Predictions are per-site rather than per-bin and have not been aggregated into fixed genome bins consistent with the gold-standard peaks annotation
- No orthogonal gold-standard m6A peak set exists for the organism/dataset, since PR labelling depends entirely on the peaks bed file
- The goal is saturation or coverage analysis rather than fixed-coverage precision-recall benchmarking

## Inputs

- Binned per-tool m6A confidence-score matrix from genome binning (--binLength fixed window size)
- m6A gold-standard peaks bed file (--peaksfile) for oligos, yeast, mouse and human datasets
- Per-tool significance cut-off settings ('default' and 'relaxed' --threshold sets, plus native tool thresholds)
- R post-processing and statistical_analysis scripts from the NanOlympicsMod Scripts folder

## Outputs

- Precision-recall curve figure file per tool/dataset
- AUPRC point estimates with 95% confidence intervals per tool-dataset pair (auprc_confidence_intervals.tsv)
- Precision and recall values at each swept stringency cut-off computed via PRROC

## How to apply

Load the binned per-tool confidence-score matrix produced by genome binning (window size set by the pipeline's --binLength parameter) and the gold-standard m6A peaks bed file (--peaksfile) to label each bin positive or negative. For each tool, sweep its native significance parameter: nanom6A probability thresholds for PR-curve plotting, Yanocomp and DiffErr FDR thresholds, DRUMMER P-value threshold, and EpiNano error-threshold sum/residuals, in addition to the 'default' and 'relaxed' threshold sets (--threshold). Compute precision and recall at each swept cut-off with the R post-processing and statistical_analysis scripts using PRROC, then derive AUPRC point estimates and 95% confidence intervals per tool per dataset. Account for the number of positive bins when interpreting CI width — 1543 (oligos) and 1453 (yeast) yield wide CIs, while 84060 (mouse) and 18932 (human) yield narrower CIs with more frequent tool overlap. Save one PR-curve figure per tool/dataset plus a single auprc_confidence_intervals.tsv summary table.

## Related tools

- **NanOlympicsMod** (Nextflow pipeline whose genome binning (--binLength), threshold filtering (--threshold) and R post-processing/statistical_analysis scripts generate the per-bin scores, PR curves and AUPRC CIs) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine that executes the containerized NanOlympicsMod pipeline steps producing the binned tool-score matrix) — https://github.com/mfurla/NanOlympicsMod
- **PRROC** (R package used within the statistical_analysis scripts to compute precision-recall curves and AUPRC at each swept threshold)

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- auprc_confidence_intervals.tsv contains AUPRC point estimates with finite 95% CI bounds for every tool-dataset pair
- Confidence intervals are wider for low-positive-count datasets (oligos: 1543 positives; yeast: 1453 positives) than for mouse (84060) and human (18932)
- One PR-curve figure file exists for each tool/dataset combination
- Precision and recall at the 'default' threshold set are reproducible from the swept cut-off series and consistent with tool-specific parameter ranges (e.g. Yanocomp/DiffErr FDR, DRUMMER P-value)
- Tools show dataset-dependent PR preferences: strong performance on the synthetic oligos dataset and reduced performance on yeast, mouse and human biological datasets

## Limitations

- AUPRC confidence intervals are large for datasets with few positive bins (oligos 1543, yeast 1453), reducing power to distinguish tools
- Yeast, mouse and human datasets lack a true ground truth; reference m6A sets come from orthogonal short-read methods, so reference bias confounds performance assessment
- Several of the 14 tools could not complete benchmark runs (abandoned due to lack of progress or empty output files), leaving gaps in the PR evaluation

## Evidence

- [abstract] Changing the stringency cut-offs allowed for tuning the precision-recall trade-off towards user preferences.: "Changing the stringency cut-offs allowed for tuning the precision-reca"
- [abstract] The performance of the tools markedly differed across datasets, and methods adopting different approaches showed different preferences in terms of precision and recall.: "The performance of the tools markedly differed across datasets, and m"
- [methods] --peaksfile Path to bed file with set of m6A gold-standard peaks: "--peaksfile Path to bed file with set of m6A gold-standard peaks"
- [results] Cut-off tuning allowed exploring the precision-recall space, with nanom6A probability thresholds, Yanocomp/DiffErr FDR thresholds, DRUMMER P-value and EpiNano error thresholds swept per tool.: "nanom6A probability thresholds for PR curve plotting; Yanocomp FDR threshold"
- [readme] The Scripts folder contains a set of R scripts for data post-processing and statistical analysis supporting the PR evaluation.: "a set of R scripts for data post-processing and statistical analysis"
