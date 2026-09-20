---
name: multiple-testing-benjamini-hochberg-correction
description: Use when many p-values from per-event statistical tests (e.g. beta-binomial
  likelihood-ratio tests of PSI differences between sample groups) must be corrected
  for multiple comparisons before thresholding at an FDR level.
when_to_use_negative:
- When only a single hypothesis is tested (no family-wise multiplicity), where a raw
  p-value suffices
- When raw p-values are not calibrated (e.g. fail uniformity on simulated nulls with
  KS test p <= 0.05), which invalidates BH's control assumptions
- When the goal is genome-wide transcript-level differential expression rather than
  per-event alternative splicing calls from LRTS segment graphs
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: IsoTools
  role: Python LRTS analysis framework that generates per-event p-values via a beta-binomial-based
    differential splicing test, to which BH correction is applied across events
  repo: https://github.com/MatthiasLienhard/isotools
- name: Python
  role: Implementation environment; SciPy (L-BFGS-B likelihood fitting) and standard
    Python statistics libraries produce the p-value vector and perform BH correction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:8dd68e6999f7bafc400cd3768e9052367339b63ca2d38101e24ebfaf8e4bed15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Multiple-testing Benjamini-Hochberg correction

## Summary

Applies Benjamini-Hochberg false-discovery-rate correction across alternative splicing events to control the expected fraction of false positives when testing thousands of events (e.g. IsoTools beta-binomial differential splicing tests) simultaneously.

## When to use

Use when many p-values from per-event statistical tests (e.g. beta-binomial likelihood-ratio tests of PSI differences between sample groups) must be corrected for multiple comparisons before thresholding at an FDR level.

## When NOT to use

- When only a single hypothesis is tested (no family-wise multiplicity), where a raw p-value suffices
- When raw p-values are not calibrated (e.g. fail uniformity on simulated nulls with KS test p <= 0.05), which invalidates BH's control assumptions
- When the goal is genome-wide transcript-level differential expression rather than per-event alternative splicing calls from LRTS segment graphs

## Inputs

- Vector of raw per-event p-values from beta-binomial likelihood-ratio tests of differential splicing between two sample groups
- FDR significance threshold (e.g. 0.05)
- Optionally: table of alternative splicing events defined from segment graphs with associated sample groups

## Outputs

- BH-adjusted q-values (FDR-adjusted p-values) aligned to the input event list
- Set of significant differential splicing events passing the q-value cutoff
- Optionally, a calibration report documenting p-value uniformity of null events (KS test, QQ plot)

## How to apply

After computing raw p-values for each tested event — in IsoTools' case one chi-square likelihood-ratio p-value per alternative splicing event from the beta-binomial group-vs-group comparison — collect the complete vector of p-values across events and apply the Benjamini-Hochberg step-up procedure to convert them to FDR-adjusted q-values. Keep events whose adjusted value passes the chosen significance threshold (e.g. q < 0.05) rather than filtering on raw p-values, since thousands of segment-graph-derived events are tested simultaneously and uncorrected calls would be dominated by false positives. Verify calibration of the underlying raw p-values first (e.g. uniformity via QQ plot or Kolmogorov-Smirnov test on simulated null groups), because BH's guarantees assume valid p-values; the workflow accepts the null when the KS test gives p > 0.05. This correction step is applied by IsoTools' differential splicing testing, which itself is based on the beta-binomial distribution for detecting differential events.

## Related tools

- **IsoTools** (Python LRTS analysis framework that generates per-event p-values via a beta-binomial-based differential splicing test, to which BH correction is applied across events) — https://github.com/MatthiasLienhard/isotools
- **Python** (Implementation environment; SciPy (L-BFGS-B likelihood fitting) and standard Python statistics libraries produce the p-value vector and perform BH correction)

## Examples

```
from statsmodels.stats.multitest import multipletests; rej, qvals, _, _ = multipletests(beta_binomial_lrt_pvals, alpha=0.05, method='fdr_bh')
```

## Evaluation signals

- On simulated null groups (identical PSI in both groups), raw p-values pass a uniformity check (QQ plot / Kolmogorov-Smirnov test p > 0.05) before BH correction is applied
- The adjusted q-value vector is monotonically non-decreasing when p-values are sorted ascending and every q-value >= its raw p-value
- Sanity comparison: with overdispersion forced to zero, beta-binomial LRT p-values match binomial LRT p-values within tolerance (max absolute difference < 1e-3), so corrected calls agree
- Number of significant events after BH correction is far below the raw p < 0.05 count on null data, confirming false-discovery-rate control
- Results are written to a calibration report documenting both raw and adjusted significance calls

## Limitations

- BH control assumes independent or positively correlated tests and calibrated p-values; overdispersed or miscalibrated counts require the beta-binomial model rather than a plain binomial test
- Correction is only as good as the underlying event definitions from segment graphs; fragment-filtered reads or poorly quantified events affect upstream p-values
- Overdispersion-parameter fitting (beta-distributed PSI fit numerically via L-BFGS-B with positivity bounds) is computationally involved and may be unstable at very low coverage

## Evidence

- [abstract] Beta-binomial differential testing basis: "we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events"
- [intro] Differential splicing between groups: "Detection of differential alternative splicing between samples and groups of samples."
- [readme] Tool context: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
- [intro] Per-event alternative splicing events feeding the test family: "Definition of alternative splicing events based on segment graphs."
