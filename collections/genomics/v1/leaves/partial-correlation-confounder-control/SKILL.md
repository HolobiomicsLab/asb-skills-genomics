---
name: partial-correlation-confounder-control
description: Use when two variant classes (e.g. SNPs vs rearrangements) both correlate
  with an outcome such as offspring viability and you must estimate each factor's
  influence alone via partial correlations on unshared-variant distances.
when_to_use_negative:
- Input is per-strain GWAS genotype/phenotype data rather than paired parental distances
  and cross-level spore-viability measurements — use mixed-model GWAS instead
- The candidate predictors are near-perfectly collinear, so partial correlations are
  unstable and cannot separate the factors' influence
- The variant classes of interest were not reliably genotyped (the authors did not
  re-examine translocations and inversions because they were challenging to accurately
  genotype)
edam_topics:
- http://edamontology.org/topic_2269
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:4c01294d625eaac63d548b120164fa2ccec9a44b3610bd21092055b966ccc507
schema_version: 0.2.0
metadata:
  license_tier: open
  tools: []
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# partial-correlation-confounder-control

## Summary

Quantifies whether two correlated genetic-distance predictors (e.g. parental SNP-distance and rearrangement-distance) each independently explain hybrid spore viability, by computing Kendall tau rank correlations per variant class and then partial correlations that control for the other class.

## When to use

Use when two variant classes (e.g. SNPs vs rearrangements) both correlate with an outcome such as offspring viability and you must estimate each factor's influence alone via partial correlations on unshared-variant distances.

## When NOT to use

- Input is per-strain GWAS genotype/phenotype data rather than paired parental distances and cross-level spore-viability measurements — use mixed-model GWAS instead
- The candidate predictors are near-perfectly collinear, so partial correlations are unstable and cannot separate the factors' influence
- The variant classes of interest were not reliably genotyped (the authors did not re-examine translocations and inversions because they were challenging to accurately genotype)

## Inputs

- Deposited variant call sets per S. pombe strain: SNPs, SVs (curated 113-call catalogue), rearrangements, and CNVs
- Spore-viability measurements for each parental cross pair
- Parent-strain pairing table linking each cross to its two parent strains

## Outputs

- Table of Kendall tau coefficients and P-values per variant class (SNPs, all SVs, rearrangements, CNVs) vs hybrid spore viability (genetic_distance_viability_kendall_correlations.tsv)
- Partial-correlation P-values for each variant class controlling for the other (e.g. rearrangements P=0.04, SNPs P=0.02)
- Statement of which variant classes independently contribute to intrinsic reproductive isolation

## How to apply

Load the deposited variant calls (SNPs, all SVs, rearrangements only, CNVs only) and the spore-viability measurements from the crosses, then compute pairwise parental genetic distances as the number of unshared variants between parent strains for each variant class. Compute Kendall tau rank correlations between each genetic-distance class and cross spore viability (in this study: SNPs tau=-0.35, P=1.6e-4; SVs tau=-0.26; rearrangements tau=-0.36, P=2.0e-4; CNVs tau=-0.10, P=0.28 non-significant). Because SV-distance and SNP-distance are themselves correlated, estimate the influence of each factor alone using partial correlations controlling for the other variant class, keeping those with P<0.05 (here rearrangements P=0.04 and SNPs P=0.02 remain significant, while CNVs drop out). Write the coefficients and P-values for all four classes plus partial-correlation results as genetic_distance_viability_kendall_correlations.tsv and check them against the reported values.

## Examples

```
from scipy.stats import kendalltau; tau, p = kendalltau(parent_snp_distance, hybrid_spore_viability)
```

## Evaluation signals

- Reproduced Kendall tau values fall on the reported values: SNPs tau=-0.35 (P=1.6e-4), SVs tau=-0.26, rearrangements tau=-0.36 (P=2.0e-4), CNVs tau=-0.10 (P=0.28)
- Partial correlations controlling for the other class remain significant for rearrangements (P=0.04) and SNPs (P=0.02), and non-significant for CNVs
- Inverse direction of every significant tau (all negative), consistent with greater parental distance correlating with lower hybrid viability
- Output file genetic_distance_viability_kendall_correlations.tsv exists with one row per variant class and columns for tau and P-value
- Sign/conclusion check: rearrangements and SNPs are retained as contributors to reproductive isolation while CNVs are excluded (tau=-0.10, P=0.28)

## Limitations

- The SV catalogue prioritized high specificity over high sensitivity, so some real SVs were missed and the counted unshared-variant distances may be underestimates
- Failure to call SVs can overestimate the apparent impact of SNPs on traits, so partial correlations between SNP-distance and rearrangement-distance may be biased
- CNV copy-number variation is only weakly correlated with nearby SNP variation (Spearman rho=0.22, P=0.041), so residual confounding between variant classes is possible; rank correlations also do not establish causation

## Evidence

- [abstract] Both SV-distance (number of unshared SVs between parents) and SNP-distance inversely correlated with hybrid viability: "Both SV-distance (number of unshared SVs between parents) and SNP-distance inversely correlated with hybrid viability"
- [abstract] we also estimated the influence of each factor alone using partial correlations.: "we also estimated the influence of each factor alone using partial correlations."
- [abstract] the variation in copy number of CNVs across the data set was only weakly correlated with SNP variation in nearby regions: "the variation in copy number of CNVs across the data set was only weakly correlated with SNP variation in nearby regions"
- [discussion] a failure to call SVs can lead to an overestimation of the impact of SNPs to traits: "a failure to call SVs can lead to an overestimation of the impact of SNPs to traits"
