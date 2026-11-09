---
name: fishers-exact-testing-of-splicing-order-ratios
description: Use when you have categorical upfi/dofi (or Fupfi) counts from two sequencing
  modalities or conditions and need to formally test agreement of splicing-order proportions,
  e.g. validating a long-read bias correction against a short-read reference.
when_to_use_negative:
- When the goal is to compare full Fupfi distributions across many exons rather than
  aggregate upfi/dofi proportions — the ratio-level test masks per-exon differences
- When counts are not independent (e.g. replicate-level paired designs) — Fisher's
  Exact on pooled counts ignores sample structure
- When the long-read data are already the trusted reference (e.g. long reads span
  beyond the flanking introns, where short reads provide almost no splicing-order
  information)
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Insplico
  role: Produces the upfi/dofi/bus/bos counts and Fupfi measurements from short- and
    long-read alignments; its --biascorr module yields the corrected counts that feed
    the contingency table
  repo: gitlab.com/aghr/insplico
- name: R
  role: Statistical environment used for the analysis; fisher.test with two-sided
    alternative tests the count-table equivalence
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:5e2674816ac077f3a5af29b266952b2e0ed5e3486015bfdbb5537e6155ef3422
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Insplico
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Fisher's exact testing of splicing-order ratios

## Summary

Two-sided Fisher's Exact test on 2x2 contingency tables of upfi/dofi counts to test whether bias-corrected long-read AISO (adjacent intron splicing order) ratios agree with the short-read reference. In the Insplico paper this is the acceptance test for the --biascorr correction on Drexler ONT K562 data (corrected ratio 1.25 vs 1.41 for short reads, P = 0.48).

## When to use

Use when you have categorical upfi/dofi (or Fupfi) counts from two sequencing modalities or conditions and need to formally test agreement of splicing-order proportions, e.g. validating a long-read bias correction against a short-read reference.

## When NOT to use

- When the goal is to compare full Fupfi distributions across many exons rather than aggregate upfi/dofi proportions — the ratio-level test masks per-exon differences
- When counts are not independent (e.g. replicate-level paired designs) — Fisher's Exact on pooled counts ignores sample structure
- When the long-read data are already the trusted reference (e.g. long reads span beyond the flanking introns, where short reads provide almost no splicing-order information)

## Inputs

- Insplico count output per exon of interest: upfi, dofi, bus, bos counts and Fupfi measurements (from short-read and long-read/BAM runs, uncorrected and --biascorr corrected)
- Drexler et al. ONT direct-RNA K562 BAM alignment (or any long-read BAM) and matching short-read count table
- Exon-of-interest input table used consistently across short- and long-read runs

## Outputs

- 2x2 contingency table of upfi/dofi counts per condition
- Per-condition upfi/dofi ratios (e.g. corrected long-read 1.25 vs short-read 1.41)
- Two-sided Fisher's Exact test P-value (expected 0.48 for the reference case)
- Interpretation: corrected long-read AISO pattern statistically indistinguishable from short reads

## How to apply

Aggregate Insplico's output counts into per-condition totals of upstream-first (upfi) and downstream-first (dofi) reads around the exons of interest, then arrange them into a 2x2 contingency table (rows = condition, columns = upfi/dofi). Compute the upfi/dofi ratio for each condition and run a two-sided Fisher's Exact test (in R via fisher.test) on the table. In the Drexler et al. ONT K562 example, the --biascorr-corrected long-read ratio of 1.25 is compared against the short-read ratio of 1.41; agreement is declared when the two-sided P-value shows no significant difference (P = 0.48 here). The test should follow the three bias-correction runs (5' correction alone, 3' correction alone, and both jointly) so the uncorrected long-read, corrected long-read, and short-read distributions can be compared side by side.

## Related tools

- **Insplico** (Produces the upfi/dofi/bus/bos counts and Fupfi measurements from short- and long-read alignments; its --biascorr module yields the corrected counts that feed the contingency table) — gitlab.com/aghr/insplico
- **R** (Statistical environment used for the analysis; fisher.test with two-sided alternative tests the count-table equivalence)

## Examples

```
fisher.test(matrix(c(upfi_lr_corrected, dofi_lr_corrected, upfi_sr, dofi_sr), nrow = 2))  # expected: p = 0.48 for ratio 1.25 vs 1.41
```

## Evaluation signals

- Corrected upfi/dofi ratio equals 1.25 (vs 1.41 for short reads) on the Drexler ONT K562 reference data
- Two-sided Fisher's Exact P-value of 0.48, i.e. no significant difference between corrected long-read and short-read ratios
- The uncorrected long-read Fupfi distribution should deviate from short reads (excess of dofi), while the --biascorr-corrected distribution should harmonize with short reads
- Contingency-table margins must equal the totals of Insplico's raw upfi/dofi count outputs for the same exon-of-interest table
- All three correction runs (5' only, 3' only, both) should be reported to show only the joint correction matches the short-read pattern

## Limitations

- The ratio-level test does not demonstrate equivalence of the full Fupfi distribution; only aggregate upfi/dofi proportions are compared
- Short reads barely provide splicing-order information beyond the introns directly flanking the input exon, so the short-read reference is itself limited
- Read length (and paired-end fragment size) limits the maximum exon length quantifiable with short reads, restricting the set of exons usable in the comparison

## Evidence

- [results] with an upfi/dofi ratio of 1.25 versus 1.41 for short reads (not significantly different, P = 0.48, two-sided Fisher's Exact test): [evidence span withheld — anchor status: not_found]
- [other] Compute the corrected upfi/dofi ratio and test its agreement with the short-read ratio using a two-sided Fisher's Exact test: [evidence span withheld — anchor status: not_found]
- [discussion] could be due to 5′ and 3′ biases in long read sequencing and library preparation that, when corrected for, harmonize the patterns: "could be due to 5′ and 3′ biases in long read sequencing and library preparation that, when corrected for, harmonize the patterns"
- [discussion] short reads can barely provide splicing order information beyond them as long reads can do: "short reads can barely provide splicing order information beyond them as long reads can do"
