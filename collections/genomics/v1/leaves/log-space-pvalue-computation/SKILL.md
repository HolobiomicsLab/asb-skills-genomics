---
name: log-space-pvalue-computation
description: Use when a coverage-based significance test (e.g., the Welch t-test comparing
  variant-region vs. expected coverage in disCoverage) yields p-values that underflow
  below 2.225074e-308 and must be recovered at full precision.
when_to_use_negative:
- p-values are within normal double-precision range (above 2.225074e-308), where plain
  R cdf evaluation suffices
- The SV calls are not deletions or duplications — disCoverage coverage analysis can
  only check DEL/DUP agreement
- No t statistic/degrees of freedom are available because the input is raw coverage
  rather than R test output
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: disCoverage
  role: Runs the coverage comparison whose significance filter emits t and df and
    requires the log-space p-value computation
  repo: https://github.com/javiercguard/disCoverage
- name: Mosdepth
  role: Computes the per-region coverage from the BAM that feeds the R coverage comparison
    underlying the t statistic
  repo: https://github.com/brentp/mosdepth
- name: R Rmpfr
  role: Exponentiates ln(p) at 100 bits of precision to recover extreme p-values without
    rounding to zero
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:907ade0d951551f3a44e5b2b1e59a21174a19df008f592f1b5709005d3efd1d8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/disCoverage
  repo_url: https://github.com/javiercguard/disCoverage
  tools:
  - Mosdepth
  - R Rmpfr
  - disCoverage
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# log-space-pvalue-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes two-tailed p-values too small for R's double precision (below 2.225074e-308) by evaluating the Student-t tail probability ln(p) = 2*cdf(-|t|, df) in log space and exponentiating with the R Rmpfr package at 100 bits. This is how disCoverage quantifies coverage support for called SVs without underflow to zero.

## When to use

Use when a coverage-based significance test (e.g., the Welch t-test comparing variant-region vs. expected coverage in disCoverage) yields p-values that underflow below 2.225074e-308 and must be recovered at full precision.

## When NOT to use

- p-values are within normal double-precision range (above 2.225074e-308), where plain R cdf evaluation suffices
- The SV calls are not deletions or duplications — disCoverage coverage analysis can only check DEL/DUP agreement
- No t statistic/degrees of freedom are available because the input is raw coverage rather than R test output

## Inputs

- t statistic and degrees of freedom from the R coverage comparison (two-tailed Welch t-test) for each candidate SV
- Per-region coverage from Mosdepth run on the sample BAM against the SV BED/VCF input (disCoverage run with -bam, -bed, -o, -i and -t options)

## Outputs

- Numeric table of high-precision per-SV two-tailed p-values (values down to ~1e-13482, non-zero), used with the p < 0.05 coverage-support criterion

## How to apply

For each candidate SV, obtain the t statistic and degrees of freedom from the R coverage comparison between the variant region and the expected coverage, computed from Mosdepth-derived per-region coverage. Compute the natural logarithm of the two-tailed p-value as ln(p) = 2*cdf(-|t|, df) in log space, avoiding direct evaluation of the vanishingly small tail probability. Recover the p-value by exponentiating ln(p) with the R Rmpfr package at 100 bits of precision, yielding values far below 2.225074e-308 (down to ~1e-13482) without rounding to zero. Assemble the per-SV p-values into a numeric table retaining full precision for extreme magnitudes, then apply the disCoverage coverage-support criterion of p < 0.05.

## Related tools

- **disCoverage** (Runs the coverage comparison whose significance filter emits t and df and requires the log-space p-value computation) — https://github.com/javiercguard/disCoverage
- **Mosdepth** (Computes the per-region coverage from the BAM that feeds the R coverage comparison underlying the t statistic) — https://github.com/brentp/mosdepth
- **R Rmpfr** (Exponentiates ln(p) at 100 bits of precision to recover extreme p-values without rounding to zero)

## Examples

```
library(Rmpfr); p <- exp(mpfr(ln_p, precBits=100))  # recover p-value from ln(p) = 2*cdf(-|t|, df) computed in log space
```

## Evaluation signals

- Recomputed p-values match ln(p) = 2*cdf(-|t|, df) for the R-reported t and df
- Most extreme p-values are non-zero and reach magnitudes of at least ~1e-13482 (below the 2.225074e-308 double-precision floor)
- Output is a per-SV numeric table retaining full precision (no values rounded to zero)
- Coverage-support classification agrees with the disCoverage criterion p < 0.05
- The disCoverage run was invoked with the documented -bam, -bed, -o, -i and -t options and Mosdepth coverage was produced per region

## Limitations

- Standard double precision in R underflows at 2.225074e-308, motivating the log-space/Rmpfr workaround
- The p-value threshold of 0.05 was not confirmed as optimal; the article states further studies are required
- Coverage is not uniform (local peaks and valleys occur without necessarily pointing to an SV), so extreme p-values reflect coverage support only for deletions/duplications

## Evidence

- [other] disCoverage's significance filter computes the natural logarithm of the p-value for a two-tailed test from the t parameter and the degrees of freedom returned by R: [evidence span withheld — anchor status: not_found]
- [other] Recover the p-value from ln(p) by exponentiation with the R Rmpfr package at 100 bits of precision, yielding values far below 2.225074e-308: [evidence span withheld — anchor status: not_found]
