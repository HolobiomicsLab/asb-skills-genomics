---
name: quantitative-trait-effect-size-estimation
description: Use when estimating additive effects of SV genotypes on quantitative
  trait measurements across related strains, e.g. 68 SVs with minor allele count ≥5
  tested against a phenotype table via mixed-model GWAS with permutation thresholds.
when_to_use_negative:
- Input genotypes have not been filtered and curated — calls must already have passed
  consensus identification (SURVIVOR), filtering (at least two callers, breakpoints
  not in low-complexity regions or annotated LTRs) and manual inspection; this skill
  is not an SV-calling skill
- SVs have minor allele counts <5 (only 68 of 113 curated SVs qualified); very rare
  SVs cannot be tested with this mixed-model design
- The question is estimating narrow-sense heritability of traits rather than testing
  individual SV–trait associations
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: LDAK
  role: Runs the mixed-model association tests that estimate the additive effect of
    each SV on each quantitative trait, with permutation-based significance thresholds
- name: SURVIVOR
  role: 'Upstream: merges and compares caller VCFs to produce the consensus SV set
    that is genotyped and fed into the association tests'
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:c4477c7ed21a9a376ca87062a511aaa73df6326d7d7159edbcb23637beb0b683
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - LDAK
  - SURVIVOR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# quantitative-trait-effect-size-estimation

## Summary

Estimates the additive effect of structural variations (SVs) on quantitative traits by running mixed-model GWAS in LDAK across an S. pombe strain panel, with trait-specific significance set by permutation. Quantifies how much SVs contribute to trait variance beyond SNPs and indels.

## When to use

Use when estimating additive effects of SV genotypes on quantitative trait measurements across related strains, e.g. 68 SVs with minor allele count ≥5 tested against a phenotype table via mixed-model GWAS with permutation thresholds.

## When NOT to use

- Input genotypes have not been filtered and curated — calls must already have passed consensus identification (SURVIVOR), filtering (at least two callers, breakpoints not in low-complexity regions or annotated LTRs) and manual inspection; this skill is not an SV-calling skill
- SVs have minor allele counts <5 (only 68 of 113 curated SVs qualified); very rare SVs cannot be tested with this mixed-model design
- The question is estimating narrow-sense heritability of traits rather than testing individual SV–trait associations

## Inputs

- SV genotype matrix for 68 SVs with minor allele counts ≥5 (duplications, deletions, inversions, translocations) across the S. pombe strain panel
- Phenotype table of quantitative traits (e.g. the 228 traits) measured across the same strains
- SNP genotype matrix (139,396 SNPs with minor allele count ≥5) and indel matrix (22,058 indels) for covariate/background relatedness in the mixed model

## Outputs

- Full SV–trait association results table (e.g. sv_trait_associations.tsv) with additive effect sizes and P-values
- List of significant SV–trait pairs passing trait-specific 5% family-wise-error permutation thresholds
- Trait-level summaries of variance explained by SNPs, CNVs and rearrangements

## How to apply

Load the SV genotype matrix (restricted to the 68 SVs with minor allele counts ≥5) together with the phenotype table of quantitative traits measured across the S. pombe strain panel. For each SV–trait pair, run a mixed-model association test in LDAK to estimate the additive effect of the SV on the trait. Compute trait-specific significance thresholds for 5% family-wise error rates by permuting the trait values, and mark associations exceeding the permutation threshold as significant. Expect roughly 28% of SVs to reach significance (15 duplications, five deletions and one translocation) with a median effect size around 14% (range 6–33%). Include SNPs (139,396) and indels (22,058, both with minor allele counts ≥5) in the mixed model so SV effects are not confounded by background SNP variation, since only 55% of SVs are in strong linkage (r2 >0.6) with SNPs.

## Related tools

- **LDAK** (Runs the mixed-model association tests that estimate the additive effect of each SV on each quantitative trait, with permutation-based significance thresholds)
- **SURVIVOR** (Upstream: merges and compares caller VCFs to produce the consensus SV set that is genotyped and fed into the association tests) — https://github.com/fritzsedlazeck/SURVIVOR

## Evaluation signals

- The results table contains one effect size and P-value per SV–trait pair for all 68 tested SVs across all traits
- Significant SVs total 19 (28% of tested SVs), comprising 15 duplications, five deletions and one translocation
- Median effect size of significant associations is ~14%, with a range of 6–33%
- Per-variant minor allele count ≥5 is enforced for every variant entering the model (68 SVs, 139,396 SNPs, 22,058 indels)
- Variance-partitioning sanity check: SNPs should explain on average ~24% of trait variance, CNVs ~7% and rearrangements ~4%

## Limitations

- SVs in weak linkage with SNPs may be badly tagged by SNPs in GWAS studies, so omitting SVs from the model can overestimate SNP effects and inflate the missing-heritability problem
- SV calling prioritized high specificity over high sensitivity, so not all SVs in the population were detected and tested
- Translocations and inversions were not re-examined by genotyping because they were more challenging to accurately genotype
- Only 55% of SVs are in strong linkage with SNPs (r2 >0.6); the remaining 45% require direct SV genotyping for association testing

## Evidence

- [abstract] we performed mixed model genome-wide association studies, using all 68 SVs with minor allele counts ⩾5: "we performed mixed model genome-wide association studies, using all 68 SVs with minor allele counts ⩾5"
- [abstract] 19 significant SVs: "Nineteen SVs (28%) were significantly associated with traits (15 duplications, five deletions and one translocation)"
- [abstract] permutation thresholds: "Trait-specific significance thresholds for 5% family-wise error rates were computed via"
- [abstract] variance explained: "On average, SNPs explained 24% of trait variance, CNVs 7% and rearrangements 4%"
- [discussion] SVs badly tagged by SNPs: [evidence span withheld — anchor status: not_found]
