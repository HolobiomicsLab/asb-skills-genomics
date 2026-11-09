---
name: missing-score-imputation
description: Use when assembling a multi-classifier pathogenicity benchmark (e.g.,
  on ClinVar or HGMD variants) and some methods lack scores for particular variants,
  so a common comparison subset must be built via imputation.
when_to_use_negative:
- When you only need the complete-case subset (variants with Revel and M-CAP scores
  available) — zero-imputation is unnecessary and subset (a) sizes apply instead
- When scoring variants outside the exome, since pre-calculated eDiVA-Scores are only
  available for all exonic variants
- When the classification task is not binary pathogenic vs. benign — the imputation
  feeds ROC/AUC analysis that requires labeled positives and negatives
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: eDiVA-Score
  role: Random forest pathogenicity classifier (1000 trees, five-fold CV) whose scores
    are compared against competing tools; eDiVA-Annotate computes its input columns
    on the fly
  repo: https://github.com/mbosio85/ediva
- name: randomForest
  role: R package used to train the eDiVA-Score binary classification model that produces
    the scores being benchmarked
- name: scikit-learn
  role: roc_curve/roc_auc_score used to generate ROC curves and compute AUC per method
    on each subset
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:bee272c1950c3249d12d4049fc041234355a24724028b4b62ca7815d6c297c1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - CADD
  - DANN
  - Eigen
  - M-CAP
  - MetaLR
  - Revel
  - eDiVA
  - eDiVA-Score
  - randomForest
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# missing-score-imputation

## Summary

Handles per-variant pathogenicity scores that are available for some classifiers (eDiVA-Score, CADD, Revel, M-CAP, MetaLR, DANN, Eigen) but not others when building common benchmark subsets, by imputing a default value of 0 for missing scores. This allows ROC/AUC comparison of all seven tools on identical variant pools instead of restricting to variants with complete score coverage.

## When to use

Use when assembling a multi-classifier pathogenicity benchmark (e.g., on ClinVar or HGMD variants) and some methods lack scores for particular variants, so a common comparison subset must be built via imputation.

## When NOT to use

- When you only need the complete-case subset (variants with Revel and M-CAP scores available) — zero-imputation is unnecessary and subset (a) sizes apply instead
- When scoring variants outside the exome, since pre-calculated eDiVA-Scores are only available for all exonic variants
- When the classification task is not binary pathogenic vs. benign — the imputation feeds ROC/AUC analysis that requires labeled positives and negatives

## Inputs

- ClinVar variant sets labeled 'pathogenic' (TP) and 'benign' (TN), excluding eDiVA-Score training variants
- HGMD DM/DM? positive variants and less-likely-pathogenic HGMD categories or GnomAD negatives
- Per-variant pathogenicity scores for eDiVA-Score, CADD, DANN, Eigen, Revel, M-CAP, and MetaLR
- Population allele frequency annotations (dbSNP, discovEHR, 1000 Genomes, GnomAD) for the AF ≤ 0.01 rare-variant filter

## Outputs

- Benchmark subset tables per criterion (a/b/c) with per-method TP/TN counts
- pathogenicity_benchmark_auc_table.csv with AUC values for all seven methods across subset criteria
- ROC curves per tool per benchmark condition

## How to apply

After loading the ClinVar or HGMD variant sets (excluding all variants used to train the eDiVA-Score random forest) and collecting precomputed scores from the respective publications/database tracks, build the comparison subsets: (a) variants with both Revel and M-CAP scores available (e.g., ClinVar: 3,887 TP / 10,494 TN); (b) a random subset of all variants where missing scores are assigned a default value of 0 (e.g., ClinVar: 19,888 TP / 16,694 TN); (c) rare variants with AF ≤ 0.01 from pool (b) (e.g., ClinVar: 16,531 TP / 15,531 TN). The rationale is that subset (a) is complete-case analysis, while subset (b) with zero-imputation retains all variants so that eDiVA-Score can be compared against competitors that lack coverage. For the HGMD-only benchmark, label DM and DM? categories as true positives and other HGMD categories as negatives. Score variants with the eDiVA-Score random forest (R 'randomForest' package, 1000 binary classification trees, five-fold cross validation), then compute ROC curves and AUC per method per subset (e.g., scikit-learn's roc_curve/roc_auc_score) and assemble a CSV table sorted by condition and method.

## Related tools

- **eDiVA-Score** (Random forest pathogenicity classifier (1000 trees, five-fold CV) whose scores are compared against competing tools; eDiVA-Annotate computes its input columns on the fly) — https://github.com/mbosio85/ediva
- **randomForest** (R package used to train the eDiVA-Score binary classification model that produces the scores being benchmarked)
- **scikit-learn** (roc_curve/roc_auc_score used to generate ROC curves and compute AUC per method on each subset)

## Examples

```
auc = sklearn.metrics.roc_auc_score(y_true, df['MetaLR'].fillna(0))
```

## Evaluation signals

- Subset row counts must match the reported TP/TN sizes (ClinVar: 3,887/10,494 for (a); 19,888/16,694 for (b); 16,531/15,531 for (c); HGMD/GnomAD: 63,712/100,000 for (a); 96,569/100,000 for (b); 90,004/97,828 for (c))
- All computed AUC values must lie within [0,1] for all seven methods under all three subset criteria of both benchmark conditions
- Computed AUCs should reproduce the trends reported in Figure 2a-c (e.g., eDiVA-Score 0.95/0.90 on ClinVar vs CADD 0.91/0.74)
- Verify that zero-imputation only applies to subset (b)/(c) rows lacking a method's score, and that subset (a) contains only variants with both Revel and M-CAP scores available
- Confirm training variants used to build the eDiVA-Score random forest are excluded from all benchmark sets

## Limitations

- Assigning a default value of 0 to missing scores can distort AUC estimates for methods with incomplete coverage, so complete-case subset (a) and imputed subset (b) results should be reported side by side
- On the HGMD-only benchmark, imputed-subset performance differs from complete-case results (eDiVA-Score AUC 0.77, below MetaLR 0.80, Revel 0.82, M-CAP 0.85)
- Pre-calculated eDiVA-Scores are only available for exonic variants; non-exonic variants cannot be scored without running the annotation pipeline
- The benchmark relies on semisynthetic trios with ClinVar variants introduced into real WES data rather than fully real disease cohorts

## Evidence

- [methods] Construct subset (b) as a random subset of all variants, assigning a default value of 0 to missing scores (ClinVar: 19,888 TP and 16,694 TN).: [evidence span withheld — anchor status: not_found]
- [methods] Random forest classifier trained with 1000 trees and five-fold cross validation.: "training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees (Breiman, 2001; Hastie, Tibshirani, & Friedman, 2009) and five‐fold cross validation"
- [methods] Rare-variant filter defining subset (c).: "(c) using only rare variants (AF, ≤0.01) from the previous pool of variants"
- [readme] Pre-calculated eDiVA-Scores availability note.: "There is an available repository of pre-calculated eDiVA-Scores for all exonic variants"
- [discussion] Scope note about semisynthetic benchmark rather than real cohorts.: "a highly realistic benchmark test combining real WES data of a parent–child trio with thousands of pathogenic ClinVar variants to generate 6,811 semisynthetic disease trios"
