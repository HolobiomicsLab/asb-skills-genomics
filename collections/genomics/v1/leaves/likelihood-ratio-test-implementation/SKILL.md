---
name: likelihood-ratio-test-implementation
description: Use when detecting differential splicing between two sample groups from
  long-read transcriptome sequencing, and you must account for within-group PSI variability
  / overdispersion in supporting-read counts rather than assuming a simple binomial.
when_to_use_negative:
- Input is short-read RNA-seq data intended for tools like rMATS or MISO rather than
  LRTS events defined by IsoTools' segment graphs
- Groups have identical PSI by construction and the goal is differential transcript
  quantification rather than event-level differential splicing testing
- Read counts are not overdispersed and a simple binomial LRT is sufficient (the beta-binomial
  reduces to binomial when overdispersion is zero)
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
tools:
- name: IsoTools
  role: Python framework that defines alternative splicing events from segment graphs
    and implements the beta-binomial-based detection of differential alternative splicing
    between sample groups
  repo: https://github.com/MatthiasLienhard/isotools
- name: Python
  role: Implementation language; scipy.optimize (L-BFGS-B with positivity bounds)
    fits the beta-binomial mixture parameters and computes the LRT
- name: rMATS
  role: Alternative short-read differential splicing tool; IsoTools can export alternative
    splicing events for it, but the beta-binomial LRT here is IsoTools' own approach
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:f50b4429cce80016fe6f1151e9eea321ef334a074a9af9585f8c65003fb566eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  - rMATS
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# likelihood-ratio-test-implementation

## Summary

Implements the beta-binomial mixture likelihood ratio test IsoTools uses to detect differential alternative splicing between groups of LRTS samples, modeling within-group PSI variability and overdispersion of supporting-read counts. Overdispersion parameters (alpha/beta) are fit numerically so group-vs-group testing does not falsely inflate significance when PSI varies within groups.

## When to use

Use when detecting differential splicing between two sample groups from long-read transcriptome sequencing, and you must account for within-group PSI variability / overdispersion in supporting-read counts rather than assuming a simple binomial.

## When NOT to use

- Input is short-read RNA-seq data intended for tools like rMATS or MISO rather than LRTS events defined by IsoTools' segment graphs
- Groups have identical PSI by construction and the goal is differential transcript quantification rather than event-level differential splicing testing
- Read counts are not overdispersed and a simple binomial LRT is sufficient (the beta-binomial reduces to binomial when overdispersion is zero)

## Inputs

- Two-group alternative splicing event supporting-read counts from LRTS bam files (e.g. PacBio SequelII) processed by IsoTools
- Alternative splicing events defined from segment graphs by IsoTools
- Synthetic two-group splice counts simulated from a beta-binomial (beta-distributed PSI) generator across grids of coverage and overdispersion settings with a fixed random seed

## Outputs

- Per-event differential splicing p-values (likelihood-ratio statistic vs chi-square) with Benjamini-Hochberg correction
- Fitted group-level alpha/beta overdispersion parameters and shared PSI estimates
- Calibration report with QQ plot / KS-test uniformity results and beta-binomial (zero overdispersion) vs binomial LRT p-value differences

## How to apply

Extract alternative splicing events with IsoTools' segment-graph-based event definition, then model supporting-read counts per event with a beta-binomial where the binomial probability p follows a Beta(a,b) distribution fit numerically (L-BFGS-B with positivity bounds via scipy.optimize). Simulate or take real two-group splice counts, fit group-level alpha/beta overdispersion parameters plus shared PSI, compute the likelihood-ratio statistic against a shared-PSI null fit, evaluate it against the chi-square distribution, and apply Benjamini-Hochberg correction across events. As a sanity check, force the overdispersion parameter to zero: the test reduces to the binomial LRT and p-values should be near-identical (max absolute difference < 1e-3). Calibration is verified on simulated null groups (identical PSI in both groups) by checking p-value uniformity with a QQ plot / Kolmogorov-Smirnov test (accept if KS p > 0.05), writing results to a calibration report.

## Related tools

- **IsoTools** (Python framework that defines alternative splicing events from segment graphs and implements the beta-binomial-based detection of differential alternative splicing between sample groups) — https://github.com/MatthiasLienhard/isotools
- **Python** (Implementation language; scipy.optimize (L-BFGS-B with positivity bounds) fits the beta-binomial mixture parameters and computes the LRT)
- **rMATS** (Alternative short-read differential splicing tool; IsoTools can export alternative splicing events for it, but the beta-binomial LRT here is IsoTools' own approach)

## Examples

```
from isotools import Transcriptome; isoseq.filter_transcripts(min_coverage=10); res = isoseq.alternative_splicing('diff_splice', groups={'A': ['CTL1','CTL2'], 'B': ['VPA1','VPA2']})
```

## Evaluation signals

- Null-group simulated p-values pass uniformity check: Kolmogorov-Smirnov test p > 0.05 (QQ plot approximately uniform)
- With overdispersion parameter forced to zero, max absolute difference between beta-binomial LRT and binomial LRT p-values is < 1e-3
- Fitted alpha/beta overdispersion parameters remain positive under the L-BFGS-B positivity bounds
- Benjamini-Hochberg corrected q-values computed across all simulated/reported events
- Calibration results written to a report for reproducibility with the fixed random seed

## Limitations

- Performance depends on simulated/real coverage and overdispersion regimes; calibration must be re-checked across grids of coverage and overdispersion settings
- The test targets alternative splicing events defined by IsoTools' segment graphs on long-read (LRTS) data; it is not a generic two-group count test for other data modalities

## Evidence

- [other] IsoTools captures within-group PSI variability by modeling supporting-read counts with a beta-binomial mixture: "we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events"
- [other] L-BFGS-B fits of alpha/beta with positivity bounds underpin the mixture: [evidence span withheld — anchor status: not_found]
- [intro] Events come from segment graphs, the inputs to the test: "Definition of alternative splicing events based on segment graphs. Detection of differential alternative splicing between samples and groups of samples."
- [readme] IsoTools is the Python framework the skill is grounded in: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
- [discussion] Export path to established differential splicing tools exists as alternative: "added option to export alternative splicing events for MISO and rMATS"
