---
name: benjamini-hochberg-fdr-correction
description: Use when you have a vector of per-region p-values (e.g. Fisher's exact
  tests of methylated vs unmethylated CpG counts per haplotype) and must control the
  expected fraction of false allele-specific calls at 10% FDR before reporting final
  region counts.
when_to_use_negative:
- Only a small handful of pre-specified loci are tested, where a family-wise or per-test
  threshold (e.g. p < 0.05) is more appropriate and BH's power/conservatism tradeoff
  is unnecessary
- The tests are not independent or positively correlated hypotheses over thousands
  of genomic regions — BH's assumptions and the per-region table structure from this
  workflow would not apply
- Input data have not passed the coverage filter (region read coverage ≥20) or haplotagging/phasing
  (Margin v2.2, WhatsHap v1.4); applying FDR correction upstream of valid per-region
  p-values produces meaningless calls
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: nanoHiMe
  role: Produces the per-read CpG methylation (mCG mode, LLR +1.5/−1.5) and 6mA-containing-region
    calls whose aggregate per-haplotype counts feed the Fisher's exact tests being
    corrected
  repo: github.com/YinLabTJ/nanoHiMe
- name: Margin
  role: v2.2 — haplotags and phases heterozygous variants so per-haplotype contingency
    tables can be built prior to testing and FDR correction
- name: WhatsHap
  role: v1.4 — performs phasing of variants used to assign reads to haplotypes before
    allele-specific testing
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:b9b83a2511fb4993e15c0847dbf731a12eae9223d5c749f5b0b696f8aba770ba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Margin
  - WhatsHap
  - nanoHiMe
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# benjamini-hochberg-fdr-correction

## Summary

Apply Benjamini–Hochberg (BH) false-discovery-rate correction at 10% FDR to the per-region p-values from the allele-specific methylation/enrichment analysis (Fisher's exact tests plus 250-bp-window bootstrap resampling) in order to call allele-specific CpG-methylation and 6mA-enrichment regions from haplotagged nanoHiMe-seq data.

## When to use

Use when you have a vector of per-region p-values (e.g. Fisher's exact tests of methylated vs unmethylated CpG counts per haplotype) and must control the expected fraction of false allele-specific calls at 10% FDR before reporting final region counts.

## When NOT to use

- Only a small handful of pre-specified loci are tested, where a family-wise or per-test threshold (e.g. p < 0.05) is more appropriate and BH's power/conservatism tradeoff is unnecessary
- The tests are not independent or positively correlated hypotheses over thousands of genomic regions — BH's assumptions and the per-region table structure from this workflow would not apply
- Input data have not passed the coverage filter (region read coverage ≥20) or haplotagging/phasing (Margin v2.2, WhatsHap v1.4); applying FDR correction upstream of valid per-region p-values produces meaningless calls

## Inputs

- Per-region Fisher's exact test p-values from 2×2 contingency tables (methylated/unmethylated × haplotype1/haplotype2)
- Bootstrap resampling p-values from 250-bp window resampling within each region
- Region coverage table retaining only regions with read coverage ≥20
- Per-mark region sets: CpG islands and ENCODE H3K27me3/H3K4me3 peak BED regions (HepG2 and GM12878)

## Outputs

- BH-adjusted p-values (q-values) at 10% FDR for each region
- List of significant allele-specific CpG-methylation CpG islands per cell line
- List of significant allele-specific 6mA-enrichment H3K27me3/H3K4me3 regions per cell line
- Per-cell-line, per-mark summary counts for verification against published values (1236/800/476 for HepG2; 1292 CGIs and 1416 H3K27me3 for GM12878)

## How to apply

After building a 2×2 contingency table (methylated vs unmethylated counts) for each CpG island or H3K27me3/H3K4me3 region per haplotype, compute Fisher's exact test p-values, then reassess significance by bootstrap resampling of 250-bp windows within each region. Pool the per-region p-values per cell line and per mark, and apply the Benjamini–Hochberg step-up procedure at 10% FDR to convert them to adjusted p-values (q-values). Only regions whose tests retain significance after BH correction are declared allele-specific; this is what produced the reported counts (HepG2: 1236 CGIs, 800 H3K27me3, 476 H3K4me3; GM12878: 1292 CGIs with one-third on the X chromosome, and 1416 H3K27me3 6mA-enrichment regions). Rationale: thousands of genomic regions are tested simultaneously, so raw p < 0.05 would inflate false positives; BH controls the FDR while remaining less conservative than family-wise correction. The underlying methylation calls feeding the tests come from nanoHiMe with log-likelihood-ratio thresholds of +1.5 (methylated) / −1.5 (unmethylated) for CpGs, and regions must retain read coverage ≥20 before any testing.

## Related tools

- **nanoHiMe** (Produces the per-read CpG methylation (mCG mode, LLR +1.5/−1.5) and 6mA-containing-region calls whose aggregate per-haplotype counts feed the Fisher's exact tests being corrected) — github.com/YinLabTJ/nanoHiMe
- **Margin** (v2.2 — haplotags and phases heterozygous variants so per-haplotype contingency tables can be built prior to testing and FDR correction)
- **WhatsHap** (v1.4 — performs phasing of variants used to assign reads to haplotypes before allele-specific testing)

## Examples

```
from statsmodels.stats.multitest import multipletests; reject, qvals, _, _ = multipletests(region_pvals, alpha=0.10, method='fdr_bh')
```

## Evaluation signals

- Region counts after BH 10% FDR correction reproduce the published values: HepG2 1236 CGIs, 800 H3K27me3, 476 H3K4me3; GM12878 1292 CGIs and 1416 H3K27me3 regions
- Adjusted p-values are bounded in [0,1] and monotone non-decreasing relative to the ordered raw p-values (a property of the BH step-up procedure)
- Approximately one-third of the significant GM12878 allele-specific CpG islands fall on the X chromosome, matching the reported finding
- Every corrected region passed the upstream coverage ≥20 filter and its contingency table had non-degenerate per-haplotype counts
- Significant regions are stable under the 250-bp-window bootstrap resampling significance check (both the Fisher's exact and bootstrap p-values survive BH correction)

## Limitations

- Haplotype phasing of 6mA-containing regions and CpG methylation depends on the density of heterozygous SNVs/INDELs, so loci with low variant density cannot be tested and will be absent from the corrected set
- Regions with read coverage <20 are excluded before testing, so low-coverage loci can never become FDR-significant calls
- nanoHiMe collapses k-mers with methyladenines in different positions to an averaged k-mer because partial adenine methylation prevents precise parameter assignment, adding noise to the underlying calls being tested

## Evidence

- [methods] Correct p-values with Benjamini–Hochberg at 10% FDR and call allele-specific regions.: [evidence span withheld — anchor status: not_found]
- [discussion] region coverage ≥20 filter: "the regions with reads coverage ≥20 were selected for the following analysis"
- [discussion] CpG methylation call thresholds: "we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs"
- [methods] bootstrap reassessment step: [evidence span withheld — anchor status: not_found]
- [readme] LLR sign convention underlying the contingency-table counts: "A positive value in Log(LIKELIHOOD_RATIO)_mCG shows the support for mCG methylation, and a negative value indicates the support for unmethylation."
