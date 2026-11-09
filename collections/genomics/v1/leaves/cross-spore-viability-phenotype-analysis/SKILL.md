---
name: cross-spore-viability-phenotype-analysis
description: Use when you have spore-viability measurements for crosses between strains
  and pairwise variant counts between parents, and need to test which variant classes
  (SNPs, SVs, rearrangements, CNVs) independently correlate with offspring viability.
when_to_use_negative:
- The goal is estimating heritability or trait variance explained (mixed-model GWAS
  on 228 quantitative traits), not cross-level viability association — use the mixed
  model / GWAS workflow instead.
- You lack per-cross viability measurements or per-parent genotype/variant calls,
  so distances cannot be computed as unshared variant counts.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: SURVIVOR
  role: Source of the curated consensus SV catalogue (merge/filter of Delly, Lumpy,
    Pindel, cn.MOPs calls) from which parental SV and rearrangement distances are
    counted
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:ce2c49c116bc76a13e481aa956e77f0ec91117a29c24bf4628325dff6828b070
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - SURVIVOR
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cross-spore-viability-phenotype-analysis

## Summary

Quantifies whether parental genetic distances (SNPs, all SVs, rearrangements only, CNVs only) correlate inversely with hybrid spore viability in S. pombe crosses, using Kendall tau rank correlations plus partial correlations to disentangle the independent contribution of each variant class to intrinsic reproductive isolation.

## When to use

Use when you have spore-viability measurements for crosses between strains and pairwise variant counts between parents, and need to test which variant classes (SNPs, SVs, rearrangements, CNVs) independently correlate with offspring viability.

## When NOT to use

- The goal is estimating heritability or trait variance explained (mixed-model GWAS on 228 quantitative traits), not cross-level viability association — use the mixed model / GWAS workflow instead.
- You lack per-cross viability measurements or per-parent genotype/variant calls, so distances cannot be computed as unshared variant counts.

## Inputs

- Deposited variant calls for S. pombe strains (SNPs, SVs from the curated 113-SV catalogue of deletions/duplications/inversions/translocations, rearrangements, CNVs)
- Spore viability measurements for each cross between S. pombe strains

## Outputs

- Table of Kendall tau coefficients and P values for each variant class vs. spore viability (genetic_distance_viability_kendall_correlations.tsv)
- Partial correlation P values for SNP-distance and rearrangement-distance controlling for each other

## How to apply

Compute the parental distance for each variant class as the number of unshared variants between the two parent strains (SNPs, all SVs, rearrangements only, CNVs only). Compute Kendall tau rank correlations between each genetic-distance class and cross spore viability, since the paper reports tau statistics with exact P values (e.g. rearrangements tau=-0.36, P=2.0e-4; SNPs tau=-0.35, P=1.6e-4; CNVs tau=-0.10, P=0.28). Because SV- and SNP-distances are correlated, estimate the influence of each factor alone using partial correlations, controlling for the other variant class (the paper reports P=0.04 for rearrangements and P=0.02 for SNPs after controlling). Report all coefficients and P values in a table (e.g. genetic_distance_viability_kendall_correlations.tsv) and compare computed values against the published ones (SNPs tau=-0.35, SVs tau=-0.26, rearrangements tau=-0.36, CNVs tau=-0.10 non-significant) to verify the reproduction.

## Related tools

- **SURVIVOR** (Source of the curated consensus SV catalogue (merge/filter of Delly, Lumpy, Pindel, cn.MOPs calls) from which parental SV and rearrangement distances are counted) — https://github.com/fritzsedlazeck/SURVIVOR

## Examples

```
from scipy import stats; tau_r, p_r = stats.kendalltau(rearrangement_distance, spore_viability); tau_s, p_s = stats.kendalltau(snp_distance, spore_viability)
```

## Evaluation signals

- SNP-distance Kendall tau should reproduce at approximately -0.35 (P=1.6e-4) and rearrangement-distance tau at approximately -0.36 (P=2.0e-4).
- SV-distance tau should reproduce at approximately -0.26; CNV-distance tau at approximately -0.10 with non-significant P (~0.28).
- Partial correlations controlling for the other variant class should remain significant: P=0.04 for rearrangements and P=0.02 for SNPs.
- Output table must contain a row for each of the four variant classes (SNPs, all SVs, rearrangements, CNVs) with tau and P columns.

## Limitations

- The correlation captures association only, not causation; rearrangements and SNPs are correlated with each other, so univariate taus must be interpreted jointly with partial correlations.
- The paper's SV set prioritized high specificity over sensitivity, so some true SVs may be missing from the distance counts and can bias associations.

## Evidence

- [abstract] Both SV-distance and SNP-distance inversely correlate with hybrid viability: "Both SV-distance (number of unshared SVs between parents) and SNP-distance inversely correlated with hybrid viability"
- [abstract] Partial correlations disentangle independent effects: "we also estimated the influence of each factor alone using partial correlations."
- [discussion] High-specificity curation of SV calls: "we applied a consensus approach to identify SVs (SURVIVOR), followed by rigorous filtering and manual inspection of all calls"
- [discussion] Specificity over sensitivity caveat: "We focused on high specificity (the correctness of the inferred SV) rather than high sensitivity (attempting to detect all SVs)"
- [results] Kendall tau values to reproduce: [evidence span withheld — anchor status: not_found]
