---
name: copy-number-expression-correlation-analysis
description: Use when matched clonal strain pairs exist where one genome carries a
  known duplication (CNV genotype) and matched expression data (e.g. microarray/RNA-seq
  log-phase cultures) are available to test dosage-driven induction.
when_to_use_negative:
- The strains compared are not from the same clonal population, so SNP background
  confounds the expression comparison
- The goal is differential expression outside duplicated regions (the paper notes
  some genes outside duplications also change, but that is a distinct analysis)
- No CNV genotypes are available to define which genes lie within each duplicated
  region
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3293
tools:
- name: SURVIVOR
  role: source of the curated CNV catalogue and copy-number genotypes whose gene content
    and dosage are correlated with expression in this skill
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:72e861fe06c44f3bb0f10875d328c8d5e0f63f92132ffb40495484a1edc0378c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - SURVIVOR
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# copy-number-expression-correlation-analysis

## Summary

Tests whether genes inside naturally occurring duplications are induced by comparing microarray expression of a duplicated strain to a non-duplicated strain from the same clonal S. pombe population, then ranks pairs by Spearman correlation between copy number and induction.

## When to use

Use when matched clonal strain pairs exist where one genome carries a known duplication (CNV genotype) and matched expression data (e.g. microarray/RNA-seq log-phase cultures) are available to test dosage-driven induction.

## When NOT to use

- The strains compared are not from the same clonal population, so SNP background confounds the expression comparison
- The goal is differential expression outside duplicated regions (the paper notes some genes outside duplications also change, but that is a distinct analysis)
- No CNV genotypes are available to define which genes lie within each duplicated region

## Inputs

- Deposited duplication genotypes defining the duplicated region per strain (copy number per CNV)
- Log-phase microarray expression dataset with duplicated/non-duplicated clonal pairs (e.g. ArrayExpress E-MTAB-4019)
- S. pombe gene annotation to map duplicated intervals to gene sets

## Outputs

- per_pair_expression_induction.tsv with strain pair, duplicated-gene set, induction statistic and significance per pair
- Spearman rank correlation (rho, P-value) between duplication copy number and expression induction across pairs
- Per-pair significance calls for induction of genes within duplications

## How to apply

For each duplicated/non-duplicated strain pair from the same clonal population, define the gene set within the duplicated region from the deposited duplication genotypes. Compute expression induction of those genes in the duplicated strain relative to its clonal counterpart from the log-phase DNA microarray data (e.g. ArrayExpress E-MTAB-4019) and assess significance of the per-pair induction. Then compute a Spearman rank correlation between duplication copy number and induction across all eight pairs to test for a dosage effect. Write a per-pair summary (strain pair, duplicated-gene set, induction statistic, significance) to per_pair_expression_induction.tsv. Expect induction in most but not all pairs: the paper found significant induction in seven of eight pairs, so a single non-significant pair does not invalidate the analysis.

## Related tools

- **SURVIVOR** (source of the curated CNV catalogue and copy-number genotypes whose gene content and dosage are correlated with expression in this skill) — https://github.com/fritzsedlazeck/SURVIVOR

## Examples

```
python -c "from scipy.stats import spearmanr; rho,p=spearmanr(copy_number, induction); print(rho,p)"
```

## Evaluation signals

- Induction should be significant in approximately 7 of 8 closely related pairs; a majority of pairs must show significant induction of genes within the duplication
- The Spearman rho between copy number and induction should be positive if dosage drives expression, as reported in the paper's related copy-number analyses
- per_pair_expression_induction.tsv must contain one row per strain pair with a non-empty duplicated-gene set, an induction statistic and a significance value
- Per-pair gene sets must exactly match the genes annotated within the deposited duplicated intervals (no genes outside the duplication included in the primary test)

## Limitations

- Some genes outside the duplicated regions also show altered expression, so whole-genome effects are not captured by the within-duplication test
- Copy number variation is only weakly correlated with SNP variation in nearby regions (Spearman rho=0.22, P=0.041), so SNP background cannot fully explain CNV-expression patterns
- With only eight strain pairs, the copy-number vs induction correlation has very low statistical power
- The study prioritized high specificity over high sensitivity in SV detection, so not all duplications may be represented in the genotype set

## Evidence

- [abstract] In seven out of the eight strain pairs, the expression levels of genes within duplications were significantly induced: "In seven out of the eight strain pairs, the expression levels of genes within duplications were significantly induced"
- [abstract] We assayed transcript expression from log phase cultures using DNA microarrays, each time comparing a duplicated to a non-duplicated strain from within the same clonal population.: "We assayed transcript expression from log phase cultures using DNA microarrays, each time comparing a duplicated to a non-duplicated strain from within the same clonal population."
- [readme] Transient structural variations have strong effects on quantitative traits and reproductive isolation in fission yeast.: "Transient structural variations have strong effects on quantitative traits and reproductive isolation in fission yeast."
