---
name: quantitative-trait-phenotype-processing
description: Use when normalized quantitative trait measurements are available together
  with both SNP/indel genotypes and structural variant genotypes, and the goal is
  to attribute trait variance to each variant class.
when_to_use_negative:
- Only SNP genotypes are available with no SV genotypes — the composite partitioning
  cannot be done and SNP-only estimates will overestimate SNP impact on traits
- The phenotype set is not quantitative trait measurements (the mixed-model narrow-sense
  heritability framing was applied to 228 quantitative traits)
- The goal is SV discovery or genotyping rather than heritability estimation — use
  the SV-calling/consensus pipeline (Delly, Lumpy, Pindel, cn.MOPs merged with SURVIVOR)
  instead
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: LDAK
  role: 'Mixed-model heritability estimation: builds the SNP-only kinship matrix and
    the composite model with separate SNP, CNV and rearrangement kinships, then fits
    per-trait variance components'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:34e8e4e56dfaf17788d7d0347de6521ecfd4471513ab2e6c91933284494ce7cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LDAK
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# quantitative-trait-phenotype-processing

## Summary

Mixed-model decomposition of narrow-sense heritability for 228 quantitative S. pombe traits using LDAK, comparing a SNP-only kinship against a composite model that partitions variance among SNPs, CNVs and rearrangements.

## When to use

Use when normalized quantitative trait measurements are available together with both SNP/indel genotypes and structural variant genotypes, and the goal is to attribute trait variance to each variant class.

## When NOT to use

- Only SNP genotypes are available with no SV genotypes — the composite partitioning cannot be done and SNP-only estimates will overestimate SNP impact on traits
- The phenotype set is not quantitative trait measurements (the mixed-model narrow-sense heritability framing was applied to 228 quantitative traits)
- The goal is SV discovery or genotyping rather than heritability estimation — use the SV-calling/consensus pipeline (Delly, Lumpy, Pindel, cn.MOPs merged with SURVIVOR) instead

## Inputs

- Normalized values for 228 quantitative traits (one phenotype column per trait)
- SNP/indel genotype calls: 139,396 SNPs and 22,058 indels with minor allele counts >=5
- Structural variant genotypes: 68 SVs (CNVs and rearrangements) with minor allele counts >=5

## Outputs

- variance_components.tsv with per-trait SNP-only heritability (range 0-74%, median 30%) and the SNP/CNV/rearrangement variance-component fractions per trait
- Class-average summary: SNPs 24%, CNVs 7%, rearrangements 4% of trait variance

## How to apply

Load the deposited SNP/indel calls (139,396 SNPs and 22,058 indels with minor allele counts >=5), the 68 SV genotypes with minor allele counts >=5, and the normalized values for the 228 quantitative traits. Use LDAK to build a SNP-only kinship matrix and, for the composite model, separate kinship matrices for SNPs, CNVs and rearrangements. Run LDAK mixed-model analysis per trait with the SNP-only kinship to get additive SNP heritability (reported range 0-74%, median 30%), then run the composite model per trait to partition variance into SNP, CNV and rearrangement components (averages: SNPs 24%, CNVs 7%, rearrangements 4%). Rationale: failure to call SVs can overestimate SNP contributions to traits and inflates the missing-heritability problem, so fitting structural variants explicitly is the decision point of this skill. Compile per-trait estimates into variance_components.tsv with one SNP-only heritability and three variance-component fractions per trait.

## Related tools

- **LDAK** (Mixed-model heritability estimation: builds the SNP-only kinship matrix and the composite model with separate SNP, CNV and rearrangement kinships, then fits per-trait variance components)

## Evaluation signals

- Every trait has exactly one SNP-only heritability and three composite fractions (SNP, CNV, rearrangement) in variance_components.tsv — 228 rows expected
- Composite component averages land near the reported values: SNPs ~24%, CNVs ~7%, rearrangements ~4%
- SNP-only heritability values fall within the reported 0-74% range with median ~30%
- All fitted variants respect the minor allele count >=5 filter (68 SVs, 139,396 SNPs, 22,058 indels)
- Estimated variance components are non-negative and SNP + CNV + rearrangement fractions are coherent (sum <= 1 per trait)

## Limitations

- A failure to call SVs can overestimate the impact of SNPs on traits and contributes to the missing-heritability problem, so composite estimates depend on the quality of the SV catalogue
- SVs are in weak linkage with SNPs and may be badly tagged by SNPs in GWAS-style analyses, so SNP-only models cannot proxy for unmodelled structural variation
- The study prioritized high specificity over high sensitivity, so not all SVs were detected — the CNV/rearrangement variance fractions are lower bounds

## Evidence

- [abstract] For each phenotype, we used mixed model analysis to estimate the total proportion of variance explained by the additive contribution of genomic variants: "For each phenotype, we used mixed model analysis to estimate the total proportion of variance explained by the additive contribution of genomic variants"
- [abstract] SNPs explained on average 24% of trait variance, CNVs 7% and rearrangements 4%: "On average, SNPs explained 24% of trait variance, CNVs 7% and rearrangements 4%"
- [discussion] Failure to call SVs leads to overestimation of SNP impact: "a failure to call SVs can lead to an overestimation of the impact of SNPs to traits or contribute to the problem that large proportions of the heritable component of trait variation are not"
