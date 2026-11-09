---
name: family-wise-error-rate-correction
description: Use when running many association tests (e.g. 68 SVs × 228 quantitative
  traits in a mixed-model GWAS) and you need trait-specific significance cutoffs at
  a 5% family-wise error rate instead of a single genome-wide P-value.
when_to_use_negative:
- You have very few tests or a single pre-specified hypothesis, where a simple Bonferroni
  correction or uncorrected threshold suffices and permuting trait values adds no
  value
- Your variant set includes SVs below the reliability threshold of the study (minor
  allele count <5), which were excluded from the mixed-model GWAS
- You need to call or filter structural variants themselves — that is handled upstream
  by the SURVIVOR consensus pipeline, not by this significance-correction step
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: LDAK
  role: Mixed-model association testing of each SV-trait pair and estimation of additive
    effects; supports the permutation framework used to derive trait-specific 5% family-wise
    error thresholds
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:7b0e22ccf372b2fc83d212bccdfbc1cfa61b07045d19dee69c6e99376462fbc2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LDAK
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# family-wise-error-rate-correction

## Summary

Permutation-based trait-specific significance thresholds that control the 5% family-wise error rate in mixed-model GWAS. In the Jeffares et al. S. pombe study this was applied to associate 68 structural variants (plus 139,396 SNPs and 22,058 indels) with quantitative traits using LDAK.

## When to use

Use when running many association tests (e.g. 68 SVs × 228 quantitative traits in a mixed-model GWAS) and you need trait-specific significance cutoffs at a 5% family-wise error rate instead of a single genome-wide P-value.

## When NOT to use

- You have very few tests or a single pre-specified hypothesis, where a simple Bonferroni correction or uncorrected threshold suffices and permuting trait values adds no value
- Your variant set includes SVs below the reliability threshold of the study (minor allele count <5), which were excluded from the mixed-model GWAS
- You need to call or filter structural variants themselves — that is handled upstream by the SURVIVOR consensus pipeline, not by this significance-correction step

## Inputs

- Variant genotype matrix for 68 SVs with minor allele counts ≥5 (duplications, deletions, translocations) in VCF-like format
- Phenotype table of 228 quantitative traits measured across the S. pombe strain panel
- Population structure / kinship matrix for the mixed model (as used by LDAK)
- Concomitant variant sets: 139,396 SNPs and 22,058 indels with minor allele counts ≥5

## Outputs

- Full association results table (e.g. sv_trait_associations.tsv) with effect sizes and P-values for every SV-trait pair
- Per-trait permutation-based significance thresholds at 5% family-wise error rate
- Set of significant associations (19 SVs: 15 duplications, five deletions, one translocation) with effect sizes (median 14%, range 6–33%)

## How to apply

For each quantitative trait, run the mixed-model association test in LDAK against the variant set (here all 68 SVs with minor allele counts ≥5, alongside 139,396 SNPs and 22,058 indels with MAC ≥5). Rather than applying a fixed P-value cutoff, compute a trait-specific threshold by permuting the trait values across strains and re-running the association scans to obtain the null distribution of test statistics. Define the 5% family-wise error threshold as the statistic level exceeded by 5% of permuted scans, so the threshold accounts for the number of variants tested for that trait. Mark variant-trait pairs whose P-values or effect statistics exceed the permutation threshold as significant. In the source study this identified 19 SVs (15 duplications, five deletions and one translocation) significantly associated with traits, with a median effect size of 14% (range 6–33%).

## Related tools

- **LDAK** (Mixed-model association testing of each SV-trait pair and estimation of additive effects; supports the permutation framework used to derive trait-specific 5% family-wise error thresholds)

## Evaluation signals

- The number of significant SV-trait pairs should be reported against trait-specific (not global) thresholds; in the source study this yields 19 significant SVs out of 68 (28%)
- The composition of significant SVs should match expectation when replicated: 15 duplications, five deletions and one translocation
- Effect sizes of significant associations should fall in the reported range of 6–33% with a median of 14%
- Permutation null distributions must be generated per trait by shuffling trait values across strains; a single pooled threshold across traits indicates the procedure was applied incorrectly
- Only SVs with minor allele count ≥5 should enter the test set (68 SVs), together with SNPs and indels passing the same MAC filter

## Limitations

- Permutation thresholds are computationally expensive because the full association scan must be repeated for each permuted trait
- SVs are in weak linkage with SNPs (only 63 of SVs have r2 >0.6 with SNPs), so significance of an SV-trait pair may partly reflect tagging by untested variants and failure to call SVs can inflate apparent SNP effects
- The study prioritized high specificity over high sensitivity in SV detection, so rare SVs missed upstream cannot be tested and could shift the significance landscape

## Evidence

- [abstract] we performed mixed model genome-wide association studies, using all 68 SVs with minor allele counts ⩾5 ... as well as 139,396 SNPs and 22,058 indels with minor allele counts ⩾5: "we performed mixed model genome-wide association studies, using all 68 SVs with minor allele counts ⩾5"
- [abstract] Trait-specific significance thresholds for 5% family-wise error rates were computed via permutation of the trait values.: "Trait-specific significance thresholds for 5% family-wise error rates were computed via"
- [abstract] Nineteen SVs (28%) were significantly associated with traits (15 duplications, five deletions and one translocation).: "Nineteen SVs (28%) were significantly associated with traits (15 duplications, five deletions and one translocation)"
- [abstract] For each phenotype, we used mixed model analysis to estimate the total proportion of variance explained by the additive contribution of genomic variants: "For each phenotype, we used mixed model analysis to estimate the total proportion of variance explained by the additive contribution of genomic variants"
