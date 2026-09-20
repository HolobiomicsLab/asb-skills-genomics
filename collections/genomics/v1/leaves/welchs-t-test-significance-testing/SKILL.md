---
name: welchs-t-test-significance-testing
description: 'Use when a putative DEL/DUP call (e.g., from Sniffles/CuteSV/SVIM/NanoVar
  nanopore pipelines) needs coverage-based validation: supported if coverage shift
  >=40% with Welch t-test p < 0.05, discarded otherwise.'
when_to_use_negative:
- The SVs to check are not deletions or duplications — coverage analysis can only
  be applied to 'some types, namely deletions and duplications' (e.g., inversions
  or translocations are out of scope)
- You need to discover SVs de novo — disCoverage 'is not a variant caller and needs
  putative SV positions', which must come from another technique or multi-sample analysis
- The region has severe local coverage peaks/valleys (e.g., centromeres or >10 Mb
  artifact-like SVs), where coverage-based shifts do not reliably indicate copy-number
  changes
edam_topics:
- http://edamontology.org/topic_0266
- http://edamontology.org/topic_0622
tools:
- name: disCoverage
  role: Runs the coverage-support check, computing the relative coverage shift and
    Welch's t-test (R) to label each DEL/DUP as supported or discarded
  repo: https://github.com/javiercguard/disCoverage
- name: Mosdepth
  role: Provides the per-base coverage over each SV's coordinates extended by 1-Mb
    flanking regions, which is the input to the t-test comparison
  repo: https://github.com/brentp/mosdepth
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:115fe51196ef217a3f38763f1f838ec6ddbed0547f1b557f05731df100a177f9
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
  - disCoverage
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0266
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Welch's t-test significance testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

disCoverage supports or discards putative deletions and duplications by comparing the coverage of the SV interval with its 1-Mb flanking surroundings using a 40% relative coverage-shift threshold and Student's t-test (Welch's approximation) implemented in R, labelling an SV as supported only when both criteria pass.

## When to use

Use when a putative DEL/DUP call (e.g., from Sniffles/CuteSV/SVIM/NanoVar nanopore pipelines) needs coverage-based validation: supported if coverage shift >=40% with Welch t-test p < 0.05, discarded otherwise.

## When NOT to use

- The SVs to check are not deletions or duplications — coverage analysis can only be applied to 'some types, namely deletions and duplications' (e.g., inversions or translocations are out of scope)
- You need to discover SVs de novo — disCoverage 'is not a variant caller and needs putative SV positions', which must come from another technique or multi-sample analysis
- The region has severe local coverage peaks/valleys (e.g., centromeres or >10 Mb artifact-like SVs), where coverage-based shifts do not reliably indicate copy-number changes

## Inputs

- 4-column BED file of putative SVs (DEL and DUP entries retained; other variant types ignored)
- Sorted/indexed nanopore BAM alignment (e.g., NA19240) for per-base coverage extraction

## Outputs

- sv_support_records.tsv — one labelled support/discard row per input SV
- Per-SV Welch t-test p-values and relative coverage-shift values

## How to apply

Parse a 4-column BED of putative SVs, retaining only deletion (DEL) and duplication (DUP) entries. Run mosdepth on the sample BAM (e.g., NA19240) to extract per-base coverage over each SV's coordinates extended by 1-Mb flanks on both sides. Compute the relative coverage shift between the SV interval and its surrounding flanking windows, then evaluate support with Welch's two-tailed t-test on the coverage distributions as implemented in R. Label the SV as supported when the coverage shift is >=40% and the p-value is < 0.05 (the paper's chosen threshold), and as discarded otherwise. Write one labelled support/discard record per input SV to sv_support_records.tsv in the output directory.

## Related tools

- **disCoverage** (Runs the coverage-support check, computing the relative coverage shift and Welch's t-test (R) to label each DEL/DUP as supported or discarded) — https://github.com/javiercguard/disCoverage
- **Mosdepth** (Provides the per-base coverage over each SV's coordinates extended by 1-Mb flanking regions, which is the input to the t-test comparison) — https://github.com/brentp/mosdepth

## Examples

```
disCoverage -h
```

## Evaluation signals

- sv_support_records.tsv exists in the output directory and contains exactly one labelled row per input SV
- Every SV labelled 'supported' satisfies BOTH the >=40% coverage shift and Welch t-test p < 0.05 criteria
- Every SV labelled 'discarded' fails at least one of the two criteria (shift < 40% or p >= 0.05)
- All non-DEL/DUP variant types in the input BED are absent from the output records
- Coverage extraction windows equal the SV coordinates plus 1 Mb on each side

## Limitations

- Coverage is not uniform in a sample; local peaks and valleys are frequently observed without necessarily pointing to an SV
- The p-value threshold of 0.05 was not confirmed as optimal — the authors note 'Further studies are required to confirm its optimality'
- Coverage-based filtering applies only to deletions and duplications, not all SV types
- disCoverage requires putative SV positions as input, which must come from an additional technique or multi-sample analysis

## Evidence

- [full_text] disCoverage compares the mean coverage of each putative SV with its surroundings using a threshold of 40% shift in coverage, and checks statistical significance with Student's t-test (Welch's approximation) implemented in R.: [evidence span withheld — anchor status: not_found]
