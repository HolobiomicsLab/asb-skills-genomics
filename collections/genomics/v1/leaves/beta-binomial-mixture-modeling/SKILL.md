---
name: beta-binomial-mixture-modeling
description: Use when comparing PSI / splice-supporting read counts between two groups
  of LRTS samples and plain binomial tests show overdispersion, so within-group variability
  must be modeled before calling differential splicing events.
when_to_use_negative:
- Input is a short-read RNA-seq event table meant for rMATS or MISO without read counts
  from IsoTools segment-graph events
- Coverage per sample is so low that alpha/beta overdispersion parameters are not
  identifiable (binomial LRT with zero overdispersion is the degenerate fallback)
- Goal is transcript reconstruction or transcript classification rather than differential
  splicing testing
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: IsoTools
  role: Defines alternative splicing events on segment graphs and supplies per-sample
    supporting read counts that feed the beta-binomial differential test
  repo: https://github.com/MatthiasLienhard/isotools
- name: Python
  role: Implementation language for the mixture log-likelihood, scipy.optimize L-BFGS-B
    fitting, chi-square LRT evaluation, and Benjamini-Hochberg correction
  repo: https://github.com/MatthiasLienhard/isotools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:11ad9b1586327f1b6805ee46e520eb8a29c7d6d0fc4f285ba6b88726b211d186
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# beta-binomial-mixture-modeling

## Summary

Statistical skill for detecting differential alternative splicing in IsoTools by modeling group-level splice-supporting read counts with a beta-binomial mixture: the binomial probability p follows a Beta(a,b) distribution fit numerically, so within-group PSI overdispersion is captured in a group-vs-group likelihood-ratio test.

## When to use

Use when comparing PSI / splice-supporting read counts between two groups of LRTS samples and plain binomial tests show overdispersion, so within-group variability must be modeled before calling differential splicing events.

## When NOT to use

- Input is a short-read RNA-seq event table meant for rMATS or MISO without read counts from IsoTools segment-graph events
- Coverage per sample is so low that alpha/beta overdispersion parameters are not identifiable (binomial LRT with zero overdispersion is the degenerate fallback)
- Goal is transcript reconstruction or transcript classification rather than differential splicing testing

## Inputs

- Per-sample alternative splicing event read counts (splice-supporting vs total) from IsoTools LRTS bam-derived transcriptome objects
- Synthetic two-group splice count datasets drawn from a beta-binomial (beta-distributed PSI) generator across coverage/overdispersion grids

## Outputs

- Beta-binomial LRT statistics with chi-square p-values per event
- Benjamini-Hochberg adjusted differential splicing results
- Calibration report with null p-value uniformity check (QQ plot / KS test) and binomial-LRT equivalence check

## How to apply

For each alternative splicing event defined on IsoTools segment graphs, extract per-sample supporting read counts and fit a beta-binomial mixture log-likelihood per group, estimating alpha/beta overdispersion parameters plus shared PSI via scipy.optimize with L-BFGS-B under positivity bounds. Compute the likelihood-ratio statistic comparing the two-group fit against a shared-PSI null fit and evaluate it against the chi-square distribution, then apply Benjamini-Hochberg correction across all simulated or measured events. For calibration, run the test on simulated null groups with identical PSI and check p-value uniformity with a QQ plot or Kolmogorov-Smirnov test (accept KS p > 0.05); forcing the overdispersion parameter to zero must reduce the model to the binomial LRT, verified by max absolute p-value difference < 1e-3 against the binomial test. A fixed random seed is used when simulating synthetic two-group beta-binomial counts across grids of coverage and overdispersion settings.

## Related tools

- **IsoTools** (Defines alternative splicing events on segment graphs and supplies per-sample supporting read counts that feed the beta-binomial differential test) — https://github.com/MatthiasLienhard/isotools
- **Python** (Implementation language for the mixture log-likelihood, scipy.optimize L-BFGS-B fitting, chi-square LRT evaluation, and Benjamini-Hochberg correction) — https://github.com/MatthiasLienhard/isotools

## Examples

```
from scipy.optimize import minimize; res = minimize(neg_loglik, x0, bounds=[(1e-8, 5), (1e-8, 5)], method='L-BFGS-B')
```

## Evaluation signals

- Null-group p-values pass uniformity via KS test with p > 0.05 (or visually flat QQ plot)
- Max absolute p-value difference between beta-binomial LRT with overdispersion forced to zero and binomial LRT < 1e-3
- Optimization converges with alpha/beta parameters strictly positive under L-BFGS-B positivity bounds
- Benjamini-Hochberg corrected values across events are monotone with raw p-values
- Results are reproducible under the fixed random seed used for synthetic count simulation

## Limitations

- The beta-binomial fit is numerical (LM-BFGS/L-BFGS-B), so convergence and positivity bounds can affect p-values in low-coverage or extreme-overdispersion settings
- With zero overdispersion the model degenerates to the binomial LRT, so it offers no advantage when within-group PSI variance is negligible
- Missing metadata provenance in repository changelog sections complicates exact version-to-method traceability

## Evidence

- [abstract] a statistical approach based on the beta-binomial distribution for detecting differential events: "we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events"
- [intro] Detection of differential alternative splicing between samples and groups of samples.: "Detection of differential alternative splicing between samples and groups of samples."
- [readme] Definition of alternative splicing events based on segment graphs.: "Definition of alternative splicing events based on segment graphs."
