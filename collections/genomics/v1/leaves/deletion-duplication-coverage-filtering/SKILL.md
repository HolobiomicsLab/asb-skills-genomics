---
name: deletion-duplication-coverage-filtering
description: Use when polishing variant-caller output (Sniffles, CuteSV, SVIM, NanoVar)
  on nanopore WGS data where DEL/DUP calls must be kept or discarded based on coverage
  evidence, especially large (>50 kb) CNVs prone to false positives.
when_to_use_negative:
- The input SVs are not deletions or duplications (e.g. insertions, translocations,
  inversions), since coverage analysis can only check DEL/DUP types
- 'You need a standalone variant caller: disCoverage is not a variant caller and requires
  putative SV positions from another technique or multi-sample analysis'
- Coverage in the target region is confounded by local peaks/valleys or unresolved
  regions, making the coverage-difference test unreliable
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: disCoverage
  role: Runs the coverage-based significance filter on candidate SVs, comparing region
    coverage against expected coverage via R and applying the p < 0.05 threshold
  repo: https://github.com/javiercguard/disCoverage
- name: Mosdepth
  role: Computes per-region read coverage from the sample BAM against the SV BED/VCF
    input regions
  repo: https://github.com/brentp/mosdepth
- name: R / Rmpfr
  role: Performs the t-statistic coverage comparison and exponentiates log p-values
    at 100 bits of precision to avoid underflow below 2.225074e-308
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:7727b3a72fe1f20d0b648285db51a352e49a5687366116f9250232be7077b8bd
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
  - R / Rmpfr
  - disCoverage
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# deletion-duplication-coverage-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Refines nanopore WGS structural variant calls for deletions and duplications by testing whether per-region read coverage (computed with Mosdepth) statistically supports the called SV type, using disCoverage with a two-tailed Welch t-test p-value threshold of < 0.05.

## When to use

Use when polishing variant-caller output (Sniffles, CuteSV, SVIM, NanoVar) on nanopore WGS data where DEL/DUP calls must be kept or discarded based on coverage evidence, especially large (>50 kb) CNVs prone to false positives.

## When NOT to use

- The input SVs are not deletions or duplications (e.g. insertions, translocations, inversions), since coverage analysis can only check DEL/DUP types
- You need a standalone variant caller: disCoverage is not a variant caller and requires putative SV positions from another technique or multi-sample analysis
- Coverage in the target region is confounded by local peaks/valleys or unresolved regions, making the coverage-difference test unreliable

## Inputs

- Sample BAM file from nanopore (PromethION) whole-genome sequencing, aligned with minimap2, NGMLR, or lra against hg38 or hg19
- Candidate SV list as 4-column BED or structural-variant VCF from variant callers (Sniffles, CuteSV, SVIM, NanoVar)
- Reference/control variant set for expected coverage comparison

## Outputs

- Per-SV table of two-tailed Welch t-test p-values with full precision for extreme magnitudes (down to ~1e-13482)
- Filtered SV set retaining deletions and duplications with coverage support (p < 0.05) and discarding unsupported calls

## How to apply

Run disCoverage with the -bam, -bed, -o, -i and -t options so that Mosdepth computes per-region coverage from the sample BAM against the candidate SV BED/VCF input, and coverage within each variant region is compared with expected coverage via R. For each candidate SV, take the t statistic and degrees of freedom from this comparison; when p-values would underflow R's double precision (below 2.225074e-308), compute the natural log tail probability ln(p) = 2*cdf(-|t|, df) in log space, then recover the p-value by exponentiation with the R Rmpfr package at 100 bits of precision (down to ~1e-13482 without rounding to zero). Retain an SV as coverage-supported when p < 0.05, filtering out calls that variant callers reported but that show no change in DNA material. Note this only works for deletions and duplications, since coverage shifts are only interpretable for those SV types.

## Related tools

- **disCoverage** (Runs the coverage-based significance filter on candidate SVs, comparing region coverage against expected coverage via R and applying the p < 0.05 threshold) — https://github.com/javiercguard/disCoverage
- **Mosdepth** (Computes per-region read coverage from the sample BAM against the SV BED/VCF input regions) — https://github.com/brentp/mosdepth
- **R / Rmpfr** (Performs the t-statistic coverage comparison and exponentiates log p-values at 100 bits of precision to avoid underflow below 2.225074e-308)

## Examples

```
disCoverage -bam sample.bam -bed sv_calls.bed -o out_prefix -i 0 -t 4
```

## Evaluation signals

- Output p-value table is well-formed and the most extreme reported p-values reach magnitudes of at least 1e-13482 and are non-zero
- SVs previously reported with coverage support are retained when p < 0.05, while unsupported caller calls (no implied change in DNA material) are filtered out
- Filtering improves specificity relative to raw caller output — the article reports up to 62% specificity improvement (and 15% sensitivity with additional data)
- Recovering p = exp(ln(p)) with Rmpfr at 100 bits reproduces values that standard R doubles would round to zero below 2.225074e-308

## Limitations

- Coverage analysis applies only to deletions and duplications, not all SV types
- Coverage is not uniform in a sample; local peaks and valleys frequently occur without pointing to an SV, and shift boundaries may be unclear
- The p-value threshold of 0.05 was not confirmed as optimal; further studies are required
- disCoverage is not a variant caller — it needs putative SV positions from an additional technique or multi-sample analysis
- Large SVs (>10 Mb) encompassing other SVs or centromeres are likely artifacts and were not considered true

## Evidence
