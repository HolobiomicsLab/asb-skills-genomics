---
name: spearman-rank-correlation-statistics
description: Use when you need to test whether a monotonic relationship exists between
  an ordinal/dosage variable (e.g. duplication copy number across strain pairs) and
  a continuous outcome (e.g. log2 expression induction), without assuming linearity
  or normality.
when_to_use_negative:
- When the goal is to estimate the independent effect of one variant class (e.g. rearrangements)
  controlling for another (SNPs) — the paper uses partial correlations for that, not
  rank correlation
- When observations are not meaningfully pairable or the relationship is expected
  to be non-monotonic; rank correlation will miss sign changes across the range
- When a fully parametric effect-size estimate (e.g. variance explained by mixed models,
  as in the paper's 24%/7%/4% decomposition) is required rather than a monotonic association
  test
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SURVIVOR
  role: Supplies the filtered/curated duplication (CNV) genotype catalogue whose copy
    numbers serve as one side of the rank correlation; not the statistics engine itself
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:5d4679e48d042e559fb3f0d1e8c3ae8f9bb78373573b58ac816988a626a627ef
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
yes_rate: 1.0
n_claims: 7
---

# spearman-rank-correlation-statistics

## Summary

Non-parametric rank-correlation analysis used in this study to quantify monotonic associations between structural-variation dosage and continuous genomic measurements — e.g. Spearman ρ=0.22 (P=0.041) between CNV copy number and nearby SNP variation, and per-pair testing of duplication copy number versus expression induction of genes within duplications.

## When to use

Use when you need to test whether a monotonic relationship exists between an ordinal/dosage variable (e.g. duplication copy number across strain pairs) and a continuous outcome (e.g. log2 expression induction), without assuming linearity or normality.

## When NOT to use

- When the goal is to estimate the independent effect of one variant class (e.g. rearrangements) controlling for another (SNPs) — the paper uses partial correlations for that, not rank correlation
- When observations are not meaningfully pairable or the relationship is expected to be non-monotonic; rank correlation will miss sign changes across the range
- When a fully parametric effect-size estimate (e.g. variance explained by mixed models, as in the paper's 24%/7%/4% decomposition) is required rather than a monotonic association test

## Inputs

- Per-pair duplication copy numbers (integer dosage) for the eight S. pombe strain pairs
- Per-pair expression induction values (log phase microarray, E-MTAB-4019) for genes within the duplicated region, duplicated strain vs non-duplicated clonal counterpart
- Optionally: per-CNV copy-number estimates and SNP variation in nearby genomic regions (for the ρ=0.22 analysis)

## Outputs

- Spearman rank correlation coefficient (ρ) across the eight strain pairs
- Associated P-value for the correlation
- Per-pair summary table (strain pair, duplicated-gene set, induction statistic, significance) written to per_pair_expression_induction.tsv

## How to apply

First assemble paired observations: in this study, the eight S. pombe strain pairs (each a duplicated strain versus a non-duplicated strain from the same clonal population), with per-pair duplication copy number and the expression induction of genes located within the duplicated region from the E-MTAB-4019 microarray data. For each pair, compute the induction of duplicated genes in the duplicated strain relative to its clonal counterpart and assess significance of the induction. Then compute the Spearman rank correlation between duplication copy number and expression induction across the eight pairs, deriving both ρ and a P-value (the paper reports ρ with P, e.g. ρ=0.22, P=0.041 for CNV copy number versus nearby SNP variation). Interpret small sample sizes cautiously: with only eight pairs, significance is limited, so report the statistic with its exact P-value rather than a binary call. Do not conflate this rank correlation with the partial correlation analysis used separately in the paper to isolate the independent influence of SNPs versus rearrangements on hybrid viability.

## Related tools

- **SURVIVOR** (Supplies the filtered/curated duplication (CNV) genotype catalogue whose copy numbers serve as one side of the rank correlation; not the statistics engine itself) — https://github.com/fritzsedlazeck/SURVIVOR

## Examples

```
from scipy.stats import spearmanr; rho, p = spearmanr(copy_number, induction)  # n=8 strain pairs
```

## Evaluation signals

- The correlation must be reported with both ρ and P (paper convention: ρ=0.22, P=0.041), not the coefficient alone
- The number of ranked pairs must match the study design (eight strain pairs for the duplication–induction test); verify n before interpreting P
- Seven of eight pairs should show significant induction of genes within duplications before the copy-number correlation is computed
- per_pair_expression_induction.tsv must contain one row per strain pair with strain pair, duplicated-gene set, induction statistic, and significance

## Limitations

- With only eight strain pairs, the correlation has very low power; ρ may be unstable and P-values coarse
- Rank correlation captures monotonic association only — it cannot decompose variance contributions or establish causality of dosage effects
- The paper notes some genes outside duplicated regions also showed altered expression, so within-duplication induction estimates can be contaminated by trans effects

## Evidence

- [abstract] Spearman ρ for CNV copy number vs nearby SNP variation: "the variation in copy number of CNVs across the data set was only weakly correlated with SNP variation in nearby regions of the genome (Spearman rank correlation ρ=0.22, P=0.041)"
- [abstract] Significance of duplicated-gene induction across pairs: "In seven out of the eight strain pairs, the expression levels of genes within duplications were significantly induced"
- [methods] Workflow definition of the per-pair correlation step: [evidence span withheld — anchor status: not_found]
