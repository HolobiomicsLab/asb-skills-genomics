---
name: t-distribution-tail-probability-computation
description: Use when coverage-based significance p-values for candidate structural
  variants fall below R's smallest representable positive double (2.225074e-308) and
  would round to zero with normal precision.
when_to_use_negative:
- p-values lie within the normal double range (> 2.225074e-308), so direct R computation
  without Rmpfr suffices
- SVs other than deletions and duplications — disCoverage's coverage analysis 'can
  only check deletions and duplications'
- No putative SV positions are available — disCoverage 'is not a variant caller and
  needs putative SV positions, which could be obtained from an additional technique'
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: disCoverage
  role: Produces the t statistic and degrees of freedom from the R coverage comparison
    used for the tail-probability computation
  repo: https://github.com/javiercguard/disCoverage
- name: Mosdepth
  role: Computes the per-region coverage of the sample BAM that feeds disCoverage's
    statistical analysis
  repo: https://github.com/brentp/mosdepth
- name: Rmpfr
  role: R package used to exponentiate ln(p) at 100 bits of precision, recovering
    p-values below 2.225074e-308
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:2e017440e4916cd7557d0d0208443971a4fb6142311e2a3fdf1fed25110ed79f
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
  - Rmpfr
  - disCoverage
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# t-distribution-tail-probability-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recover two-tailed Welch t-test p-values from disCoverage's coverage significance analysis that underflow R's smallest positive double (2.225074e-308), by evaluating ln(p) = 2*cdf(-|t|, df) in log space and exponentiating with Rmpfr at 100 bits of precision (down to ~1e-13482).

## When to use

Use when coverage-based significance p-values for candidate structural variants fall below R's smallest representable positive double (2.225074e-308) and would round to zero with normal precision.

## When NOT to use

- p-values lie within the normal double range (> 2.225074e-308), so direct R computation without Rmpfr suffices
- SVs other than deletions and duplications — disCoverage's coverage analysis 'can only check deletions and duplications'
- No putative SV positions are available — disCoverage 'is not a variant caller and needs putative SV positions, which could be obtained from an additional technique'

## Inputs

- BAM alignment file (nanopore WGS) for the sample
- Candidate SV list as 4-column BED or structural-variant VCF (positions from a variant caller or other technique)
- Per-SV t statistic and degrees of freedom from the R coverage comparison between variant region and expected coverage

## Outputs

- Numeric table of per-SV two-tailed p-values at full arbitrary precision (down to ~1e-13482, non-zero)
- Coverage-support verdict per SV (p < 0.05) usable to filter variant-caller output

## How to apply

Run disCoverage with the -bam, -bed, -o, -i and -t options on the sample BAM against the SV BED/VCF input so Mosdepth computes per-region coverage; for each candidate deletion or duplication, take the t statistic and degrees of freedom from the R coverage comparison between the variant region and the expected coverage. Instead of working with the vanishingly small p-value directly, compute the natural logarithm of the two-tailed tail probability ln(p) = 2*cdf(-|t|, df) in log space to avoid underflow of the CDF evaluation. Recover the p-value from ln(p) by exponentiation with the R Rmpfr package at 100 bits of precision, yielding non-zero values far below 2.225074e-308. Assemble the per-SV p-values into a numeric table retaining full precision, and apply disCoverage's coverage-support criterion of p < 0.05 to filter called SVs.

## Related tools

- **disCoverage** (Produces the t statistic and degrees of freedom from the R coverage comparison used for the tail-probability computation) — https://github.com/javiercguard/disCoverage
- **Mosdepth** (Computes the per-region coverage of the sample BAM that feeds disCoverage's statistical analysis) — https://github.com/brentp/mosdepth
- **Rmpfr** (R package used to exponentiate ln(p) at 100 bits of precision, recovering p-values below 2.225074e-308)

## Examples

```
library(Rmpfr); p <- exp(mpfr(ln_p, precBits = 100))  # ln_p = log of 2*cdf(-|t|, df) from disCoverage's R comparison
```

## Evaluation signals

- Most extreme reported p-values are non-zero and reach magnitudes of at least ~1e-13482
- All p-values below 2.225074e-308 survive without rounding to zero (arbitrary-precision check)
- Two-tailed symmetry holds: ln(p) equals 2*cdf(-|t|, df) for the t and df returned by R
- Output is a numeric table retaining full precision for extreme magnitudes
- Per-SV coverage-support classification is consistent with disCoverage's criterion p < 0.05

## Limitations

- R's normal precision underflows below 2.225074e-308, so naive CDF evaluation rounds extreme p-values to zero
- The p-value threshold 0.05 was not confirmed as optimal; the article notes 'Further studies are required to confirm its optimality'
- Coverage is non-uniform, with local peaks and valleys that do not necessarily indicate an SV, and unresolved genome regions limit coverage analysis

## Evidence

- [other] disCoverage's significance filter computes the natural logarithm of the p-value for a two-tailed test from the t parameter and the degrees of freedom returned by R: [evidence span withheld — anchor status: not_found]
