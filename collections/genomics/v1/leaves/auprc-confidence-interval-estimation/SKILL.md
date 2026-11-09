---
name: auprc-confidence-interval-estimation
description: Use when benchmarking m6A-detection tools on binned dRNA-seq confidence
  scores against gold-standard peak BED labels, and you need PR curves plus AUPRC
  CIs per tool per dataset, or you must relate CI width to positive-bin counts.
when_to_use_negative:
- When the analysis goal is F1 or hit counts at a single fixed cut-off rather than
  full PR curves and AUPRC uncertainty
- When bins are not labeled against a gold-standard m6A peak set (no positives/negatives
  to compute precision and recall)
- When a dataset has very few positive bins and the target output is fine-grained
  tool ranking — CI width will be too large for reliable discrimination
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0091
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline producing the binned per-tool confidence-score matrices
    and hosting the R post-processing/statistical scripts that compute PR curves,
    AUPRC and 95% CIs via PRROC
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Yanocomp
  role: One of the 14 benchmarked m6A detection tools; its FDR threshold is swept
    to generate precision-recall points
- name: Nanocompore
  role: Multi-sample m6A detection tool included in the 14-tool benchmark whose calls
    feed the PR curve computation
- name: ELIGOS
  role: Multi-sample m6A detection tool benchmarked across datasets for AUPRC estimation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:804efa9a0454d8ff2ad02b7b4567d31d2e7632e8ff9c619ca17609932352d6ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - ELIGOS
  - NanOlympicsMod
  - Nanocompore
  - Yanocomp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# auprc-confidence-interval-estimation

## Summary

Estimate AUPRC point estimates and 95% confidence intervals for 14 m6A detection tools across Nanopore dRNA-seq datasets (oligos, yeast, mouse, human), and assess how CI width depends on the number of positive genome bins. Uses PRROC on threshold-swept per-tool confidence scores binned by --binLength and labeled against m6A gold-standard peak BED files.

## When to use

Use when benchmarking m6A-detection tools on binned dRNA-seq confidence scores against gold-standard peak BED labels, and you need PR curves plus AUPRC CIs per tool per dataset, or you must relate CI width to positive-bin counts.

## When NOT to use

- When the analysis goal is F1 or hit counts at a single fixed cut-off rather than full PR curves and AUPRC uncertainty
- When bins are not labeled against a gold-standard m6A peak set (no positives/negatives to compute precision and recall)
- When a dataset has very few positive bins and the target output is fine-grained tool ranking — CI width will be too large for reliable discrimination

## Inputs

- Binned per-tool confidence-score matrix from genome binning (window length set by --binLength)
- m6A gold-standard peaks BED file (--peaksfile) for oligos, yeast, mouse, human datasets
- Per-tool significance cut-offs: nanom6A probability thresholds, Yanocomp FDR, DiffErr FDR, DRUMMER P-value, EpiNano error thresholds (sum of errors, residuals)
- 'default' and 'relaxed' threshold sets (--threshold)

## Outputs

- One PR curve figure file per tool/dataset
- AUPRC-with-95%-CI summary table (auprc_confidence_intervals.tsv) for all 14 tool-dataset pairs

## How to apply

Load the binned per-tool confidence-score matrix produced by upstream genome binning (window length set by --binLength) and the m6A gold-standard peaks BED file to label each bin positive or negative for oligos (1543 positives), yeast (1453), mouse (84060) and human (18932). For each of the 14 tools, sweep the native scoring parameter — nanom6A probability thresholds, Yanocomp FDR threshold, DiffErr FDR threshold, DRUMMER P-value threshold, EpiNano error-threshold (sum of errors and residuals) — together with the 'default' and 'relaxed' threshold sets, so cut-off tuning explores the precision-recall space. Compute precision and recall at each swept threshold using PRROC in the R post-processing and statistical analysis scripts, then derive AUPRC point estimates and 95% CIs per tool per dataset. Expect wide CIs where positive-bin counts are low (oligos, yeast) and narrow CIs with frequent tool overlaps where they are high (mouse, human); save one PR curve figure per tool/dataset and the auprc_confidence_intervals.tsv summary table.

## Related tools

- **NanOlympicsMod** (Nextflow pipeline producing the binned per-tool confidence-score matrices and hosting the R post-processing/statistical scripts that compute PR curves, AUPRC and 95% CIs via PRROC) — https://github.com/mfurla/NanOlympicsMod
- **Yanocomp** (One of the 14 benchmarked m6A detection tools; its FDR threshold is swept to generate precision-recall points)
- **Nanocompore** (Multi-sample m6A detection tool included in the 14-tool benchmark whose calls feed the PR curve computation)
- **ELIGOS** (Multi-sample m6A detection tool benchmarked across datasets for AUPRC estimation)

## Examples

```
Rscript scripts/compute_auprc_ci.R --scores binned_scores.tsv --peaksfile gold_standard_peaks.bed --threshold default
```

## Evaluation signals

- auprc_confidence_intervals.tsv contains AUPRC point estimates with finite 95% CI bounds for every one of the 14 tool-dataset pairs
- CI widths are wider for low-positive-count datasets (oligos: 1543 positives; yeast: 1453) than for mouse (84060) and human (18932)
- Tool overlap frequency (overlapping CIs) is higher for mouse and human than for oligos and yeast
- One PR curve figure file exists per tool/dataset combination (14 x 4)
- Swept cut-offs include both the 'default' and 'relaxed' threshold sets, showing movement through the precision-recall space

## Limitations

- Yeast, mouse and human datasets lack a ground truth for m6A positions; reference sets come from orthogonal short-read methods, so bias differences between orthogonal references and nanopore-based techniques confound AUPRC interpretation
- Low positive-bin counts in the oligos (1543) and yeast (1453) datasets yield large AUPRC CIs, limiting tool discrimination
- EpiNano-SVM and Nanom6A were trained on synthetic oligos lacking real transcriptome complexity, which can depress their AUPRC on biological datasets

## Evidence

- [results] AUPRC CIs vs positive-bin count: [evidence span withheld — anchor status: not_found]
- [methods] gold-standard peaks file argument: "--peaksfile Path to bed file with set of m6A gold-standard peaks"
- [methods] threshold set option: "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
- [abstract] cutoff tuning finding: "Changing the stringency cut-offs allowed for tuning the precision-recall trade-off towards user preferences."
- [readme] R scripts available in repo: "Scripts: folder containing a bash script to run the nextflow pipeline, a set of R scripts for data post-processing and statistical analysis"
