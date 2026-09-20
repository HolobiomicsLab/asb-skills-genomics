---
name: kendall-rank-correlation-analysis
description: Use when pairwise parental genetic distances per variant class (SNPs,
  SVs, rearrangements, CNVs) must be tested against hybrid offspring viability to
  identify which variant classes drive reproductive isolation.
when_to_use_negative:
- The question is single-variant association with quantitative traits rather than
  cross-level viability — use the mixed-model GWAS (68 SVs, 139,396 SNPs, 22,058 indels
  with minor allele count >=5) instead
- The aim is variance decomposition of trait heritability (SNPs 24%, CNVs 7%, rearrangements
  4% of trait variance) — use the mixed-model heritability analysis, not rank correlation
- Only CNV copy number vs nearby SNP variation is being compared (weak correlation,
  Spearman rho=0.22, P=0.041) — this uses Spearman rank correlation, not the hybrid-viability
  Kendall framework
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SURVIVOR
  role: Upstream consensus SV caller whose rigorously filtered, manually inspected
    SV call set (113 SVs) supplies the per-strain SV/rearrangement genotypes from
    which unshared-variant distances are counted
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:7a07b919e5c9991f38792a639f0763af7001975a93628f13c29d92bdcf475940
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - SURVIVOR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# kendall-rank-correlation-analysis

## Summary

A rank-based statistical workflow that tests whether parental genetic distances for specific variant classes (SNPs, all SVs, rearrangements, CNVs) inversely correlate with hybrid spore viability in S. pombe crosses, and disentangles independent contributions via partial correlations. Used in Jeffares et al. 2017 to show that rearrangements (tau=-0.36, P=2.0e-4) and SNPs (tau=-0.35, P=1.6e-4), but not CNVs (tau=-0.10, P=0.28), contribute to intrinsic reproductive isolation.

## When to use

Use when pairwise parental genetic distances per variant class (SNPs, SVs, rearrangements, CNVs) must be tested against hybrid offspring viability to identify which variant classes drive reproductive isolation.

## When NOT to use

- The question is single-variant association with quantitative traits rather than cross-level viability — use the mixed-model GWAS (68 SVs, 139,396 SNPs, 22,058 indels with minor allele count >=5) instead
- The aim is variance decomposition of trait heritability (SNPs 24%, CNVs 7%, rearrangements 4% of trait variance) — use the mixed-model heritability analysis, not rank correlation
- Only CNV copy number vs nearby SNP variation is being compared (weak correlation, Spearman rho=0.22, P=0.041) — this uses Spearman rank correlation, not the hybrid-viability Kendall framework

## Inputs

- Deposited per-strain variant call sets for S. pombe worldwide strains: 139,396 SNPs, indels, 113 curated SVs (23 deletions, 64 duplications, 11 inversions, 15 translocations) with rearrangement sub-classes, and 49 CNVs
- Spore-viability measurements (offspring viability) for each pairwise S. pombe cross
- Clonal-population/strain cluster assignments used to define parent pairs

## Outputs

- Pairwise parental genetic-distance matrix per variant class (number of unshared SNPs, SVs, rearrangements, CNVs)
- Table genetic_distance_viability_kendall_correlations.tsv with Kendall tau and P per variant class, plus partial-correlation P-values
- Conclusion distinguishing variant classes that independently contribute to intrinsic reproductive isolation (rearrangements, SNPs) from those that do not (CNVs)

## How to apply

Load the deposited variant calls (SNPs, SVs, rearrangements, CNVs) per strain and the spore-viability measurements from each cross. For each variant class, compute the pairwise parental genetic distance as the number of unshared variants between the two parent strains (separately counting all SVs, rearrangements only, and CNVs only). Compute Kendall tau rank correlations between each distance class and cross spore viability, then estimate the independent influence of each factor using partial correlations, controlling for the other variant class. Interpret negative tau as inverse correlation with viability and keep only classes whose association survives partial-correction (rearrangements P=0.04, SNPs P=0.02, CNVs non-significant P=0.28). Write the coefficients, P-values and partial-correlation results to genetic_distance_viability_kendall_correlations.tsv and diff them against the reported values (SNPs tau=-0.35, all SVs tau=-0.26, rearrangements tau=-0.36, CNVs tau=-0.10 ns).

## Related tools

- **SURVIVOR** (Upstream consensus SV caller whose rigorously filtered, manually inspected SV call set (113 SVs) supplies the per-strain SV/rearrangement genotypes from which unshared-variant distances are counted) — https://github.com/fritzsedlazeck/SURVIVOR

## Evaluation signals

- Reproduced Kendall tau values match the reported coefficients within tolerance: SNPs tau=-0.35 (P=1.6e-4), all SVs tau=-0.26, rearrangements tau=-0.36 (P=2.0e-4), CNVs tau=-0.10 (P=0.28)
- Partial-correlation results agree with reported significance: rearrangements P=0.04 and SNPs P=0.02 remain significant, CNVs remain non-significant
- All four correlation coefficients carry a negative sign, consistent with inverse correlation between genetic distance and hybrid viability
- Output TSV exists with one row per variant class (SNPs, all SVs, rearrangements, CNVs) and columns for tau, P and partial-correlation P
- Distances are non-negative integer counts of unshared variants per class and are symmetric for each parent pair

## Limitations

- CNVs showed no significant correlation with hybrid spore viability (tau=-0.10, P=0.28), so CNV distance should not be claimed as a driver of reproductive isolation
- Translocations and inversions were not re-examined by genotyping because they were 'more challenging to accurately genotype', so rearrangement-class distances can be incomplete or noisy
- The study prioritized high specificity over sensitivity of SV calling, meaning some true SVs were missed and SNP impact could be overestimated in related analyses
- Partial correlations control only the pairwise effect of the other variant class; higher-order interactions are not modelled

## Evidence

- [abstract] Both SV-distance and SNP-distance inversely correlate with hybrid viability: "Both SV-distance (number of unshared SVs between parents) and SNP-distance inversely correlated with hybrid viability"
- [abstract] Partial correlations estimate each factor's independent influence: "we also estimated the influence of each factor alone using partial correlations."
- [abstract] Variant classes explain differing trait variance: "On average, SNPs explained 24% of trait variance, CNVs 7% and rearrangements 4%"
- [discussion] Missing SV calls can bias SNP contributions: "a failure to call SVs can lead to an overestimation of the impact of SNPs to traits or contribute to the problem that large proportions of the heritable component of trait variation are not"
