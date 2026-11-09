---
name: mixed-model-heritability-estimation
description: Use when quantitative trait measurements and multiple variant classes
  (SNPs, indels, CNVs, rearrangements) are available and you need to partition trait
  variance per variant class rather than run SNP-only GWA.
when_to_use_negative:
- Traits are binary/case-control rather than quantitative — the method here was calibrated
  on 228 quantitative trait values, not discrete phenotypes
- Structural variant genotypes are unreliable for the variant class of interest —
  the authors did not re-examine translocations and inversions because they were 'more
  challenging to accurately genotype'
- Only one variant class is available (e.g. SNPs alone) — a composite multi-kinship
  partition cannot be performed and only the SNP-only model applies
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: LDAK
  role: Constructs the SNP-only and composite (SNP/CNV/rearrangement) kinship matrices
    and runs the per-trait mixed-model variance decomposition
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:e3d4a0f8237d4332e0f8c65bd9e99795ccb4d8d4cf6ea103f47e6ea338decd2d
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# mixed-model-heritability-estimation

## Summary

Estimate narrow-sense heritability for 228 quantitative traits in S. pombe using LDAK mixed models, comparing a SNP-only kinship against a composite model with separate kinship matrices for SNPs, CNVs and rearrangements. This partitions trait variance across variant classes that are weakly linked to SNPs and would otherwise inflate SNP-only estimates.

## When to use

Use when quantitative trait measurements and multiple variant classes (SNPs, indels, CNVs, rearrangements) are available and you need to partition trait variance per variant class rather than run SNP-only GWA.

## When NOT to use

- Traits are binary/case-control rather than quantitative — the method here was calibrated on 228 quantitative trait values, not discrete phenotypes
- Structural variant genotypes are unreliable for the variant class of interest — the authors did not re-examine translocations and inversions because they were 'more challenging to accurately genotype'
- Only one variant class is available (e.g. SNPs alone) — a composite multi-kinship partition cannot be performed and only the SNP-only model applies

## Inputs

- SNP/indel genotype calls (139,396 SNPs and 22,058 indels, minor allele count >=5)
- Structural variant genotypes (68 SVs with minor allele count >=5, from the curated catalogue of 113 SVs)
- Normalized quantitative trait values for 228 traits across worldwide S. pombe strains

## Outputs

- Per-trait SNP-only heritability estimates (variance fraction, range 0-74%, median 30%)
- Per-trait composite variance-component fractions for SNPs (~24%), CNVs (~7%) and rearrangements (~4%)
- variance_components.tsv tabulating all per-trait estimates

## How to apply

Load the deposited genotypes — 139,396 SNPs and 22,058 indels (minor allele count >=5), 68 SV genotypes (minor allele count >=5) — and the normalized values for the 228 quantitative traits. Use LDAK to build a SNP-only kinship matrix, then a composite set of three kinship matrices separating SNPs, CNVs and rearrangements. Run LDAK mixed-model heritability estimation per trait under each model: the SNP-only kinship yields the additive SNP heritability (observed range 0-74%, median 30%), while the composite model yields three variance-component fractions per trait (reported averages: SNPs 24%, CNVs 7%, rearrangements 4%). Compile all per-trait estimates into variance_components.tsv. Rationale: CNV copy number is only weakly correlated with nearby SNP variation (Spearman rho=0.22, P=0.041) and 45% of SVs are weakly linked to SNPs (r2 <= 0.6), so omitting structural variants overestimates SNP contributions and aggravates the missing-heritability problem.

## Related tools

- **LDAK** (Constructs the SNP-only and composite (SNP/CNV/rearrangement) kinship matrices and runs the per-trait mixed-model variance decomposition)

## Evaluation signals

- Composite-model variance fractions average approximately SNPs 24%, CNVs 7%, rearrangements 4% across the 228 traits
- SNP-only heritability estimates fall in the reported 0-74% range with median ~30%
- variance_components.tsv contains one row per trait with the SNP-only estimate plus the three composite variance-component fractions
- Composite SNP-only fraction differs from the SNP-only model for traits where CNVs/rearrangements contribute — CVN/SNP linkage checks show rho=0.22, so large SV-driven traits should show a SNP-fraction drop in the composite model
- All variant sets were filtered to minor allele count >=5 before kinship construction — verify input counts (68 SVs, 139,396 SNPs, 22,058 indels)

## Limitations

- The study prioritized SV-calling specificity over sensitivity, so not all SVs were detected; missed SVs can overestimate the SNP contribution and contribute to the missing-heritability problem
- SVs are often weakly linked to SNPs (45% of SVs with r2 <= 0.6), so SNP-based tagging is poor and SV effects are easily absorbed by or missing from SNP-only models
- Translocations and inversions were not re-genotyped (excluded from CNV re-examination), so their genotypes may be less accurate in the composite kinship

## Evidence

- [abstract] For each phenotype, we used mixed model analysis to estimate the total proportion of variance explained by the additive contribution of genomic variants: "For each phenotype, we used mixed model analysis to estimate the total proportion of variance explained by the additive contribution of genomic variants"
- [abstract] SNP vs composite variance partition: "On average, SNPs explained 24% of trait variance, CNVs 7% and rearrangements 4%"
- [abstract] Variant input sets used in the models: [evidence span withheld — anchor status: not_found]
- [abstract] Weak CNV-SNP correlation motivates separate kinships: "the variation in copy number of CNVs across the data set was only weakly correlated with SNP variation in nearby regions of the genome (Spearman rank correlation ρ=0.22, P=0.041)"
- [discussion] Missing-SV bias rationale: "a failure to call SVs can lead to an overestimation of the impact of SNPs to traits or contribute to the problem that large proportions of the heritable component of trait variation are not"
