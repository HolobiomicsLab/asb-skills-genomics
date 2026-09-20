---
name: kinship-matrix-construction-from-genotypes
description: Use when you have genotype calls (SNPs/indels plus SV genotypes) and
  multiple quantitative traits, and need a mixed-model decomposition of how much variance
  each variant class explains.
when_to_use_negative:
- Input is still raw short-read data needing SV calls — this skill requires curated,
  genotyped variants (SV discovery/consensus is done by the Delly/Lumpy/Pindel/cn.MOPS
  + SURVIVOR pipeline instead)
- Goal is single-variant association testing (mixed-model GWAS with permutation-based
  5% family-wise error thresholds), not variance partitioning across variant classes
- Goal is hybrid-viability analysis — the study used partial correlations of SV-distance/SNP-distance
  there, not kinship-based heritability models
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_2885
tools:
- name: LDAK
  role: Constructs the SNP-only and class-specific (SNP/CNV/rearrangement) kinship
    matrices and runs the per-trait mixed-model heritability estimation and variance
    partitioning
- name: SURVIVOR
  role: 'Upstream: produced the consensus SV catalogue (113 SVs) whose CNV/rearrangement
    genotypes feed the composite kinship matrices'
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:f51200da4151678e2a41e6b430594ba7f85fcbcc94a0559df0c0a0178d911337
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - LDAK
  - SURVIVOR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_2885
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# kinship-matrix-construction-from-genotypes

## Summary

Builds genetic relatedness (kinship) matrices from genotype data — a SNP-only matrix, or separate per-variant-class matrices — as random-effect inputs for LDAK mixed-model heritability estimation across 228 quantitative traits in S. pombe, partitioning trait variance among SNPs, CNVs and rearrangements.

## When to use

Use when you have genotype calls (SNPs/indels plus SV genotypes) and multiple quantitative traits, and need a mixed-model decomposition of how much variance each variant class explains.

## When NOT to use

- Input is still raw short-read data needing SV calls — this skill requires curated, genotyped variants (SV discovery/consensus is done by the Delly/Lumpy/Pindel/cn.MOPS + SURVIVOR pipeline instead)
- Goal is single-variant association testing (mixed-model GWAS with permutation-based 5% family-wise error thresholds), not variance partitioning across variant classes
- Goal is hybrid-viability analysis — the study used partial correlations of SV-distance/SNP-distance there, not kinship-based heritability models

## Inputs

- SNP/indel genotype calls (139,396 SNPs and 22,058 indels, minor allele count ≥5)
- SV genotype matrix (68 SVs with minor allele count ≥5, split into CNVs and rearrangements)
- Normalized phenotype table for 228 quantitative traits
- Sample/covariate metadata (e.g. clonal population structure) for the mixed model

## Outputs

- SNP-only kinship matrix
- Three class-specific kinship matrices (SNPs, CNVs, rearrangements)
- Per-trait SNP-only heritability estimates (range 0–74%, median 30%)
- variance_components.tsv with per-trait SNP, CNV and rearrangement variance fractions

## How to apply

Load the deposited SNP/indel calls (139,396 SNPs and 22,058 indels with minor allele count ≥5), the 68 SV genotypes with minor allele count ≥5, and normalized values for the 228 quantitative traits. Use LDAK to construct one kinship matrix from SNPs only, and — for the composite model — three separate kinship matrices from SNPs, CNVs and rearrangements respectively, so each class enters the mixed model as its own random effect. Run LDAK mixed-model heritability estimation per trait twice: once with the SNP-only kinship to obtain the additive SNP heritability (reported range 0–74%, median 30%), and once with the composite model to partition variance among SNPs, CNVs and rearrangements. Compile all per-trait estimates into variance_components.tsv containing the SNP-only heritability and the three variance-component fractions (reported class averages: SNPs 24%, CNVs 7%, rearrangements 4%). Restrict variant inputs to minor allele count ≥5, mirroring the study's GWAS/heritability filtering.

## Related tools

- **LDAK** (Constructs the SNP-only and class-specific (SNP/CNV/rearrangement) kinship matrices and runs the per-trait mixed-model heritability estimation and variance partitioning)
- **SURVIVOR** (Upstream: produced the consensus SV catalogue (113 SVs) whose CNV/rearrangement genotypes feed the composite kinship matrices) — https://github.com/fritzsedlazeck/SURVIVOR

## Evaluation signals

- SNP-only heritabilities per trait fall in [0, 1] with median ≈ 30% and no estimates above 74% (matching the reported range)
- Per trait, the three composite variance fractions sum to ≤ 1 and the class averages reproduce ≈24% (SNPs), ≈7% (CNVs), ≈4% (rearrangements)
- Each kinship matrix is square, symmetric, positive-definite, and off-diagonal values reflect known clonal-population structure of the S. pombe strains
- Composite-model CNV and rearrangement fractions are non-zero, confirming the composite model captures variance beyond SNPs alone
- variance_components.tsv has one row per trait (228 rows) with all four estimate columns populated

## Limitations

- Failure to call SVs can lead to overestimation of the impact of SNPs on traits and contributes to the missing-heritability problem — SNP-only heritability may be inflated by uncatalogued SVs
- 45% of SVs are only weakly linked to SNPs (r² ≤ 0.6), so SV effects are badly tagged by SNPs in SNP-only models and cannot be fully recovered from SNP genotypes
- Translocations and inversions are hard to genotype accurately (excluded from genotyping re-examination), so the rearrangement kinship component may be incomplete

## Evidence

- [abstract] On average, SNPs explained 24% of trait variance, CNVs 7% and rearrangements 4%: "On average, SNPs explained 24% of trait variance, CNVs 7% and rearrangements 4%"
- [abstract] For each phenotype, we used mixed model analysis to estimate the total proportion of variance explained by the additive contribution of genomic variants: "For each phenotype, we used mixed model analysis to estimate the total proportion of variance explained by the additive contribution of genomic variants"
- [discussion] Failure to call SVs overestimates SNP impact: "a failure to call SVs can lead to an overestimation of the impact of SNPs to traits or contribute to the problem that large proportions of the heritable component of trait variation are not"
