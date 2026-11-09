---
name: parental-genetic-distance-computation-from-variant-calls
description: Use when you have genotype-resolved variant calls (SNP VCFs, SV/rearrangement/CNV
  call sets) for parent strains plus cross-level hybrid offspring viability measurements,
  and want to test which variant classes correlate with reproductive isolation.
when_to_use_negative:
- The goal is trait association within strains (mixed-model GWAS on 68 SVs / SNPs
  with permutation thresholds), not between-strain distance-vs-hybrid-fitness correlation.
- Variants cannot be reliably genotyped in both parents — in the source study translocations
  and inversions were not re-examined because they were 'more challenging to accurately
  genotype'.
- No cross-level phenotype (hybrid spore viability) exists for the parent pairs, leaving
  only per-strain trait data.
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: SURVIVOR
  role: Merges and compares per-caller SV VCFs into a consensus multi-sample VCF so
    SV/rearrangement presence per parent strain can be tallied into unshared-variant
    distances
  repo: https://github.com/fritzsedlazeck/SURVIVOR
- name: cn.MOPS
  role: Provides copy-number states per strain used to derive CNV-only parental distances
- name: Delly
  role: One of the four SV callers whose calls are merged by SURVIVOR before distance
    computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:796cf77fecfe7669d1c96caf3d256bf5297ca60034003ee9d89b397324fc741b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - Delly
  - SURVIVOR
  - cn.MOPS
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# parental-genetic-distance-computation-from-variant-calls

## Summary

Computes pairwise parental genetic distances as the number of unshared variants between S. pombe parent strains for each variant class (SNPs, all SVs, rearrangements only, CNVs only) from genotyped variant calls, then tests each class against hybrid spore viability using Kendall tau rank correlations and partial correlations. Used to disentangle which variant classes contribute to intrinsic reproductive isolation.

## When to use

Use when you have genotype-resolved variant calls (SNP VCFs, SV/rearrangement/CNV call sets) for parent strains plus cross-level hybrid offspring viability measurements, and want to test which variant classes correlate with reproductive isolation.

## When NOT to use

- The goal is trait association within strains (mixed-model GWAS on 68 SVs / SNPs with permutation thresholds), not between-strain distance-vs-hybrid-fitness correlation.
- Variants cannot be reliably genotyped in both parents — in the source study translocations and inversions were not re-examined because they were 'more challenging to accurately genotype'.
- No cross-level phenotype (hybrid spore viability) exists for the parent pairs, leaving only per-strain trait data.

## Inputs

- Genotyped SNP VCF across all parent strains (139,396 SNPs and 22,058 indels with minor allele counts ≥5 in the source study)
- Consensus SV/rearrangement call set as a multi-sample VCF (e.g. from SURVIVOR merge: 113 curated SVs — 23 deletions, 64 duplications, 11 inversions, 15 translocations)
- CNV copy-number states per strain (e.g. from cn.MOPS, 49 segregating CNVs)
- Cross table mapping parent strain pairs to hybrid spore viability measurements

## Outputs

- Per-variant-class pairwise parental genetic-distance matrix (unshared-variant counts)
- Kendall tau and P-value per distance class vs spore viability
- Partial correlation P-values for each factor controlling for the other
- Results table, e.g. genetic_distance_viability_kendall_correlations.tsv

## How to apply

Load the deposited variant calls (SNPs, SVs, rearrangements, CNVs) and spore-viability measurements from the previous crosses, ensuring every variant is genotyped in both parents of each cross. For each variant class, compute pairwise parental genetic distance as the number of unshared variants between the two parent strains — SVs/rearrangements are binary presence/absence (VCF genotype per sample), CNVs are copy-number states from callers such as cn.MOPS, SNPs are biallelic differences. Compute Kendall tau between each distance class and cross spore viability, then estimate each factor's influence alone via partial correlations controlling for the other class (e.g. rearrangements controlling for SNPs). Interpret classes as contributors to reproductive isolation only if both the tau and the partial correlation remain significant; here rearrangements (tau=-0.36, P=2.0e-4; partial P=0.04) and SNPs (tau=-0.35, P=1.6e-4; partial P=0.02) qualify, whereas CNVs (tau=-0.10, P=0.28) do not.

## Related tools

- **SURVIVOR** (Merges and compares per-caller SV VCFs into a consensus multi-sample VCF so SV/rearrangement presence per parent strain can be tallied into unshared-variant distances) — https://github.com/fritzsedlazeck/SURVIVOR
- **cn.MOPS** (Provides copy-number states per strain used to derive CNV-only parental distances)
- **Delly** (One of the four SV callers whose calls are merged by SURVIVOR before distance computation)

## Examples

```
from scipy.stats import kendalltau; tau, p = kendalltau(parental_unshared_variants, spore_viability)
```

## Evaluation signals

- Recomputed Kendall taus match reported values: SNPs tau=-0.35 (P=1.6e-4), all SVs tau=-0.26, rearrangements tau=-0.36 (P=2.0e-4), CNVs tau=-0.10 (P=0.28, non-significant)
- Partial correlations controlling for the other variant class keep SNPs and rearrangements significant at P=0.02 and P=0.04 respectively
- CNV correlation is non-significant (P=0.28), consistent with CNVs not contributing to reproductive isolation
- Distances are counts of unshared variants per class, so every parent pair yields non-negative integer distances and each variant is counted only if genotyped in both parents
- Output table genetic_distance_viability_kendall_correlations.tsv contains one row per variant class with tau and P columns

## Limitations

- The SV catalogue prioritized high specificity over high sensitivity, so some SVs were missed and distances may be undercounts
- Translocations and inversions were not re-examined for genotyping accuracy because they were more challenging to genotype, adding noise to rearrangement-only distances
- CNV copy-number variation is only weakly correlated with nearby SNP variation (Spearman rho=0.22, P=0.041), so CNV distances are not proxies for SNP distances
- Correlation is over a limited set of crosses; significance of CNVs vs rearrangements rests on this small sample

## Evidence

- [abstract] Both SV-distance (number of unshared SVs between parents) and SNP-distance inversely correlated with hybrid viability: "Both SV-distance (number of unshared SVs between parents) and SNP-distance inversely correlated with hybrid viability"
- [abstract] Partial correlation of individual variant classes against viability: "we also estimated the influence of each factor alone using partial correlations."
- [abstract] Genotyping reliability limits for translocations and inversions: "Since translocations and inversions were more challenging to accurately genotype, we did not re-examine these variants."
- [readme] SURVIVOR merge provides the consensus SV calls used for distance tallies: [evidence span withheld — anchor status: external_doc]
