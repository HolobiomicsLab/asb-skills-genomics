---
name: coverage-based-sv-significance-testing
description: Use when filtering nanopore SV calls (DEL/DUP) by coverage support and
  the required Welch t-test p-values fall below R's smallest representable double
  (~2.225074e-308), risking underflow to zero.
when_to_use_negative:
- The SV types are not deletions or duplications — disCoverage can only check DEL/DUP
  because it relies on coverage changes in DNA material
- You need a de novo variant caller — disCoverage is not a variant caller and requires
  putative SV positions from another technique or multi-sample analysis
- Coverage is expected to be non-informative for the region (e.g. local peaks/valleys,
  centromeres, unresolved regions) where significance testing may reflect coverage
  artifacts rather than a true SV
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: disCoverage
  role: 'Runs the coverage significance filter: takes -bam, -bed, -o, -i and -t options,
    computes log-space two-tailed Welch p-values and applies the p < 0.05 coverage-support
    criterion'
  repo: https://github.com/javiercguard/disCoverage
- name: Mosdepth
  role: Computes per-region BAM/CRAM depth that disCoverage's t-test compares between
    the variant region and expected coverage
  repo: https://github.com/brentp/mosdepth
- name: Rmpfr (R)
  role: Exponentiates log p-values at 100 bits of precision so values far below 2.225074e-308
    are recovered without rounding to zero
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:a30c2be40bdef2c60d9f83e6e67e57591e98c8e893ac041b00a8fd5b83fcaba8
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
  - Rmpfr (R)
  - disCoverage
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# coverage-based-sv-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes Welch two-tailed t-test p-values comparing per-region read-depth (from Mosdepth) between a candidate SV region and expected coverage, recovering extreme p-values in log-CDF space and exponentiating them with R Rmpfr at 100 bits of precision so disCoverage can test coverage support at p < 0.05.

## When to use

Use when filtering nanopore SV calls (DEL/DUP) by coverage support and the required Welch t-test p-values fall below R's smallest representable double (~2.225074e-308), risking underflow to zero.

## When NOT to use

- The SV types are not deletions or duplications — disCoverage can only check DEL/DUP because it relies on coverage changes in DNA material
- You need a de novo variant caller — disCoverage is not a variant caller and requires putative SV positions from another technique or multi-sample analysis
- Coverage is expected to be non-informative for the region (e.g. local peaks/valleys, centromeres, unresolved regions) where significance testing may reflect coverage artifacts rather than a true SV

## Inputs

- Sample BAM file of nanopore (PromethION) whole-genome reads
- Candidate SV list as 4-column BED or structural-variant VCF
- Per-region coverage values computed by Mosdepth
- t statistic and degrees of freedom from R's coverage comparison per SV

## Outputs

- Per-SV p-value numeric table retaining full precision for extreme magnitudes (down to ~1e-13482)
- Filtered SV set retaining deletions/duplications with coverage support at p < 0.05

## How to apply

Run disCoverage with the -bam, -bed, -o, -i and -t options so Mosdepth yields per-region coverage from the sample BAM against the SV BED/VCF input. For each candidate SV, R compares variant-region coverage with expected coverage and returns a t statistic and degrees of freedom. Rather than evaluating the vanishing p-value directly, compute the natural logarithm of the tail probability from ln(p) = 2*cdf(-|t|, df) in log space to avoid CDF underflow. Recover the actual p-value by exponentiating ln(p) with the R Rmpfr package at 100 bits of precision, yielding non-zero values as extreme as ~1e-13482. Assemble per-SV p-values into a numeric table at full precision and apply disCoverage's coverage-support threshold of p < 0.05 to keep or discard each deletion or duplication call.

## Related tools

- **disCoverage** (Runs the coverage significance filter: takes -bam, -bed, -o, -i and -t options, computes log-space two-tailed Welch p-values and applies the p < 0.05 coverage-support criterion) — https://github.com/javiercguard/disCoverage
- **Mosdepth** (Computes per-region BAM/CRAM depth that disCoverage's t-test compares between the variant region and expected coverage) — https://github.com/brentp/mosdepth
- **Rmpfr (R)** (Exponentiates log p-values at 100 bits of precision so values far below 2.225074e-308 are recovered without rounding to zero)

## Examples

```
disCoverage -bam sample.wgs.bam -bed candidate_sv.bed -o out_prefix -i cov_index -t 0.05
```

## Evaluation signals

- The most extreme reported p-values reach magnitudes of at least 1e-13482 and are non-zero (no underflow to 0)
- Every retained SV passes the coverage-support criterion p < 0.05 used by disCoverage
- Output p-value table is numeric, full-precision, and has one p-value per candidate SV in the input BED/VCF
- Mosdepth per-region coverage output (regions.bed.gz) has one depth entry per input SV region
- Read-depth-only calls that variant callers reported but coverage does not support (p >= 0.05) are excluded from the refined set

## Limitations

- Coverage-based filtering applies only to deletions and duplications, not all SV types
- Coverage is not uniform in a sample; local peaks and valleys, unresolved genome regions, and unclear shift boundaries can confound the significance test
- The p-value threshold 0.05 was not confirmed as optimal; further studies are required
- disCoverage is not a variant caller and needs putative SV positions supplied as input

## Evidence

- [readme] Mosdepth region output: [evidence span withheld — anchor status: external_doc]
