---
name: fisher-exact-enrichment-testing
description: Use when two categorical classifications (e.g. dofi vs upfi splicing
  order) must be compared between a regulated set (DeltaPSI > 0.15, control PSI <=
  0.2) and a matched control set to test enrichment.
when_to_use_negative:
- When the comparison variable is continuous (e.g. PSI or Fupfi distributions) rather
  than a dichotomous dofi/upfi class — use a distribution test (e.g. via Matt feature
  comparison) instead.
- When counts per cell of the contingency table are expected to be very small and
  an exact 2x2 assumption is violated; the article grounds this test only in the dofi/upfi
  class counts.
- When testing AISO constancy across tissues or species rather than enrichment within
  one contrast; the article treats AISO as remarkably constant, so no enrichment test
  applies there.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0203
tools:
- name: Insplico
  role: Produces the upfi/dofi/bos/bus counts and Fupfi estimates whose dichotomization
    (Fupfi <= 0.2 for dofi, >= 0.8 for upfi) feeds the 2x2 contingency table
  repo: gitlab.com/aghr/insplico
- name: R
  role: Environment for running Fisher's exact test and tabulating class counts
- name: dplyr
  role: R package used to aggregate and filter microexon classifications into contingency-table
    counts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:d1fb7b28ab7238a000f458395f0742a22e70e82e64e1d75feec4276523e7ef54
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Insplico
  - R
  - STAR
  - dplyr
  - plyranges
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# fisher-exact-enrichment-testing

## Summary

Apply Fisher's exact test on 2x2 contingency tables to assess enrichment of categorical splicing-order classes (upfi vs dofi) in a regulated exon set versus matched controls, as done to show 58% downstream-intron-first splicing among SRRM4-dependent neural microexons (P = 0.013).

## When to use

Use when two categorical classifications (e.g. dofi vs upfi splicing order) must be compared between a regulated set (DeltaPSI > 0.15, control PSI <= 0.2) and a matched control set to test enrichment.

## When NOT to use

- When the comparison variable is continuous (e.g. PSI or Fupfi distributions) rather than a dichotomous dofi/upfi class — use a distribution test (e.g. via Matt feature comparison) instead.
- When counts per cell of the contingency table are expected to be very small and an exact 2x2 assumption is violated; the article grounds this test only in the dofi/upfi class counts.
- When testing AISO constancy across tissues or species rather than enrichment within one contrast; the article treats AISO as remarkably constant, so no enrichment test applies there.

## Inputs

- Per-exon Insplico output with upfi, dofi, bus and bos counts plus Fupfi/PSI/PIR estimates
- Microexon classification table with DeltaPSI > 0.15 and control PSI <= 0.2 assignments (SRRM4-dependent vs matched controls)
- Count matrix of microexons per splicing-order class (2x2 contingency table)

## Outputs

- Fisher's exact test P-value and odds ratio for dofi/upfi enrichment
- Fraction of SRRM4-dependent microexons classified dofi vs upfi (expected 58% dofi vs 26% upfi)
- Results table (e.g. srrm4_microexon_aiso_results.tsv) with AISO proportions, test statistics, and feature-comparison summaries

## How to apply

First classify each microexon by adjacent-intron splicing order using Insplico's upfi/dofi/bos/bus counts and Fupfi: strongly dofi when Fupfi <= 0.2, strongly upfi when Fupfi >= 0.8. Define the test set as SRRM4-dependent microexons (DeltaPSI > 0.15 between 3xFlag-SRRM4 and GFP HEK293 samples, control PSI <= 0.2) and assemble matched control microexons for comparison. Build a 2x2 contingency table crossing class (dofi vs upfi) by group (SRRM4-dependent vs control) and run Fisher's exact test in R to obtain an odds ratio and P-value; the reported expectation is 58% dofi vs 26% upfi among SRRM4-dependent microexons with P = 0.013. Write the resulting proportions, test statistics, and downstream feature comparisons (exon length, 3' splice-site strength, UGC RNA map via Matt) to a results table such as srrm4_microexon_aiso_results.tsv.

## Related tools

- **Insplico** (Produces the upfi/dofi/bos/bus counts and Fupfi estimates whose dichotomization (Fupfi <= 0.2 for dofi, >= 0.8 for upfi) feeds the 2x2 contingency table) — gitlab.com/aghr/insplico
- **R** (Environment for running Fisher's exact test and tabulating class counts)
- **dplyr** (R package used to aggregate and filter microexon classifications into contingency-table counts)

## Examples

```
fisher.test(matrix(c(n_srrm4_dofi, n_srrm4_upfi, n_ctrl_dofi, n_ctrl_upfi), nrow=2))
```

## Evaluation signals

- 2x2 contingency table cell counts sum to the total number of classified microexons in both groups with no unclassified entries
- Class thresholds applied exactly as reported: dofi requires Fupfi <= 0.2 and upfi requires Fupfi >= 0.8
- SRRM4-dependence criterion reproduced: DeltaPSI > 0.15 and control PSI <= 0.2 against GFP control samples
- Result direction matches the published finding — majority dofi (58%) over upfi (26%) among SRRM4-dependent microexons, Fisher P ~ 0.013
- Output table srrm4_microexon_aiso_results.tsv contains proportions, test statistics, and feature-comparison summaries

## Limitations

- Dichotomizing Fupfi into dofi (<= 0.2) and upfi (>= 0.8) discards intermediate/bos/bus categories, so microexons with ambiguous AISO are excluded from the test.
- Proposed mechanisms of SRRM4-dependent microexon splicing based on AISO enrichment remain untested hypotheses.
- The test is only as reliable as the Insplico PSI/PIR quantification; short reads limit the maximum quantifiable exon length and provide little splicing-order information beyond the directly flanking introns.

## Evidence

- [results] Finding: dofi enrichment among SRRM4-dependent microexons: [evidence span withheld — anchor status: not_found]
- [methods] Fisher's exact test workflow step: [evidence span withheld — anchor status: not_found]
- [methods] SRRM4-dependence classification thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] Insplico measurement outputs used for classification: "It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts)"
