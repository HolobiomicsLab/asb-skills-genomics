---
name: mixed-model-gwas-association-testing
description: Use when testing genotype variants (SVs with minor allele counts ≥5,
  SNPs or indels) against quantitative trait measurements across related strains or
  individuals, where population structure requires a mixed-model GWAS with family-wise
  error control.
when_to_use_negative:
- The goal is heritability estimation rather than association testing — use the separate
  mixed-model analysis estimating total proportion of variance explained by additive
  genomic variants.
- Variants have minor allele counts <5, which were explicitly excluded from the GWAS
  in this study.
- SVs were not curated first — the pipeline prioritized high specificity (consensus
  of Delly, Lumpy, Pindel and cn.MOPs calls, filtering, manual vetting) before genotyping,
  so uncurated raw SV calls should not be fed directly into association testing.
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: LDAK
  role: Performs the mixed-model association tests estimating the additive effect
    of each SV on each quantitative trait
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:acd574d9a18cfce30fd4055aade825b7fb0220b56aa125cf500bb09981b5da5a
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
n_claims: 6
---

# mixed-model-gwas-association-testing

## Summary

Runs mixed-model genome-wide association studies of structural variants (and SNPs/indels) against quantitative traits in a strain panel, using permutation-derived trait-specific significance thresholds. Used in the S. pombe study to show that 19 of 68 SVs (28%) were significantly trait-associated.

## When to use

Use when testing genotype variants (SVs with minor allele counts ≥5, SNPs or indels) against quantitative trait measurements across related strains or individuals, where population structure requires a mixed-model GWAS with family-wise error control.

## When NOT to use

- The goal is heritability estimation rather than association testing — use the separate mixed-model analysis estimating total proportion of variance explained by additive genomic variants.
- Variants have minor allele counts <5, which were explicitly excluded from the GWAS in this study.
- SVs were not curated first — the pipeline prioritized high specificity (consensus of Delly, Lumpy, Pindel and cn.MOPs calls, filtering, manual vetting) before genotyping, so uncurated raw SV calls should not be fed directly into association testing.

## Inputs

- SV genotype matrix for 68 SVs with minor allele count ≥5 (from the curated set of 113 SVs: 23 deletions, 64 duplications, 11 inversions, 15 translocations)
- Phenotype table of 228 quantitative traits measured across the S. pombe strain panel
- SNP genotype data (139,396 SNPs, minor allele count ≥5) and indel genotypes (22,058 indels, minor allele count ≥5)
- Strain relatedness/kinship structure for the mixed model

## Outputs

- Full association results table (sv_trait_associations.tsv) with additive effect sizes and P-values for each SV-trait pair
- Trait-specific permutation significance thresholds at 5% family-wise error rate
- List of significant SV-trait associations (expected: 19 SVs, 15 duplications, five deletions, one translocation)

## How to apply

Restrict the genotype matrix to the 68 SVs with minor allele counts ≥5 (plus 139,396 SNPs and 22,058 indels with minor allele counts ≥5) and pair it with the phenotype table of 228 quantitative traits measured across the S. pombe strain panel. For each SV-trait pair, run a mixed-model association test in LDAK to estimate the additive effect of the SV on the trait, accounting for relatedness among clonal strains. Compute trait-specific significance thresholds for 5% family-wise error rates by permuting each trait's values across strains. Mark SV-trait associations exceeding the permutation threshold as significant; the reference study found 19 significant SVs (15 duplications, five deletions and one translocation) with median effect size 14% (range 6–33%). Rationale: SVs are often in weak linkage with SNPs, so they should be tested directly rather than assumed to be tagged by SNP associations.

## Related tools

- **LDAK** (Performs the mixed-model association tests estimating the additive effect of each SV on each quantitative trait)

## Evaluation signals

- Genotype matrix contains exactly 68 SVs after applying the minor allele count ≥5 filter.
- Each trait has its own permutation-derived significance threshold at 5% family-wise error rate, not a single global cutoff.
- Number of significant SVs is in the expected range: 19 SVs (28%), comprising 15 duplications, five deletions and one translocation.
- Effect sizes of significant SV-trait associations have median ~14% with range 6–33%.
- Results table includes effect sizes and P-values for all SV-trait pairs and is written to sv_trait_associations.tsv.

## Limitations

- Failure to call SVs can lead to overestimation of the impact of SNPs on traits and contribute to the missing-heritability problem.
- SVs are in weak linkage with SNPs and may be badly tagged by SNPs in GWAS studies, so SNP-only GWAS misses SV effects.
- SVs explain a smaller share of trait variance on average (CNVs 7%, rearrangements 4%) than SNPs (24%), so expect modest effect sizes.

## Evidence

- [abstract] GWAS restricted to SVs with minor allele count ≥5: [evidence span withheld — anchor status: not_found]
- [abstract] Nineteen SVs significantly associated with traits: "Nineteen SVs (28%) were significantly associated with traits (15 duplications, five deletions and one translocation)"
- [abstract] Permutation-based trait-specific thresholds: "Trait-specific significance thresholds for 5% family-wise error rates were computed via"
- [abstract] Mixed model analysis for variance components: "For each phenotype, we used mixed model analysis to estimate the total proportion of variance explained by the additive contribution of genomic variants"
- [discussion] SVs badly tagged by SNPs: [evidence span withheld — anchor status: not_found]
