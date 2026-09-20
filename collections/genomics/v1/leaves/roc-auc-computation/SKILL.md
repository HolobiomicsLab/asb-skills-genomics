---
name: roc-auc-computation
description: Use when comparing binary variant classifiers (pathogenic vs benign)
  on labeled ClinVar or HGMD variant sets and you need ROC curves and AUC values per
  method per test subset.
when_to_use_negative:
- The task is variant calling, annotation, or prioritization rather than benchmarking
  a pathogenicity classifier (use the eDiVA-Predict/Annotate/Prioritize pipeline instead)
- Variant labels lack binary TP/TN definitions (e.g. no 'pathogenic' vs 'benign' ClinVar
  labels or HGMD DM/DM? categories), since ROC/AUC is undefined without class labels
- Only a single classifier is scored without competing methods, making ROC comparison
  across tools pointless
tools:
- name: eDiVA-Score
  role: Random forest pathogenicity classifier under evaluation (R 'randomForest'
    package, 1000 binary classification trees, five-fold cross validation); pre-calculated
    scores available for all exonic variants
  repo: https://github.com/mbosio85/ediva
- name: CADD
  role: Competing pathogenicity classifier whose scores are collected for ROC/AUC
    comparison
- name: Eigen
  role: Competing pathogenicity classifier whose scores are collected for ROC/AUC
    comparison
- name: DANN
  role: Competing pathogenicity classifier whose scores are collected for ROC/AUC
    comparison
- name: Revel
  role: Competing pathogenicity classifier; its availability defines subset criterion
    (a)
- name: M-CAP
  role: Competing pathogenicity classifier; its availability defines subset criterion
    (a)
- name: MetaLR
  role: Competing pathogenicity classifier whose scores are collected for ROC/AUC
    comparison
- name: randomForest
  role: R package used to score variants with the eDiVA-Score random forest during
    benchmarking
  repo: https://github.com/mbosio85/ediva
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:5af2416d198a42d64aefb1fbd4275010f8fd4d0770ca325f6fe3b5fd03bb1382
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# roc-auc-computation

## Summary

Computes receiver operating characteristic (ROC) curves and area under the curve (AUC) to benchmark variant pathogenicity classifiers such as eDiVA-Score against CADD, DANN, Eigen, Revel, M-CAP, and MetaLR on ClinVar/HGMD test sets with defined TP/TN subsets.

## When to use

Use when comparing binary variant classifiers (pathogenic vs benign) on labeled ClinVar or HGMD variant sets and you need ROC curves and AUC values per method per test subset.

## When NOT to use

- The task is variant calling, annotation, or prioritization rather than benchmarking a pathogenicity classifier (use the eDiVA-Predict/Annotate/Prioritize pipeline instead)
- Variant labels lack binary TP/TN definitions (e.g. no 'pathogenic' vs 'benign' ClinVar labels or HGMD DM/DM? categories), since ROC/AUC is undefined without class labels
- Only a single classifier is scored without competing methods, making ROC comparison across tools pointless

## Inputs

- ClinVar variant test set with 'pathogenic'/'benign' labels (excluding eDiVA-Score training variants)
- HGMD variant set with DM/DM? vs less likely pathogenic categories, and/or GnomAD negatives
- Precomputed per-variant scores for eDiVA-Score, CADD, Eigen, DANN, Revel, M-CAP, MetaLR
- Subset definition parameters (Revel/M-CAP availability, random subset with 0-imputed missing scores, AF <= 0.01)

## Outputs

- ROC curves per method per subset criterion
- AUC values per method (e.g. eDiVA-Score 0.95/0.90, CADD 0.91/0.74, MetaLR 0.80, Revel 0.82, M-CAP 0.85 on relevant subsets)
- pathogenicity_benchmark_auc_table.csv with per-method TP/TN counts and AUCs across the three subset conditions

## How to apply

Load labeled variant sets, excluding all variants used to train the eDiVA-Score random forest, labeling ClinVar 'pathogenic' as true positives and 'benign' as true negatives (or HGMD DM/DM? as positives vs less likely pathogenic categories). Collect precomputed scores for all seven methods, then construct three subset conditions: (a) only variants with Revel and M-CAP scores available, (b) a random subset with default value 0 assigned to missing scores, and (c) only rare variants with allele frequency <= 0.01 from the previous pool. Generate an ROC curve per tool per subset with scikit-learn's roc_curve and compute roc_auc_score, then assemble per-method TP/TN counts and AUC values into a CSV table sorted by condition and method. Validate by checking that subset row counts match the reported sizes (ClinVar: 3,887/10,494; 19,888/16,694; 16,531/15,531) and that computed AUCs reproduce trends such as eDiVA-Score's 0.95/0.90 vs CADD's 0.91/0.74 on ClinVar.

## Related tools

- **eDiVA-Score** (Random forest pathogenicity classifier under evaluation (R 'randomForest' package, 1000 binary classification trees, five-fold cross validation); pre-calculated scores available for all exonic variants) — https://github.com/mbosio85/ediva
- **CADD** (Competing pathogenicity classifier whose scores are collected for ROC/AUC comparison)
- **Eigen** (Competing pathogenicity classifier whose scores are collected for ROC/AUC comparison)
- **DANN** (Competing pathogenicity classifier whose scores are collected for ROC/AUC comparison)
- **Revel** (Competing pathogenicity classifier; its availability defines subset criterion (a))
- **M-CAP** (Competing pathogenicity classifier; its availability defines subset criterion (a))
- **MetaLR** (Competing pathogenicity classifier whose scores are collected for ROC/AUC comparison)
- **randomForest** (R package used to score variants with the eDiVA-Score random forest during benchmarking) — https://github.com/mbosio85/ediva

## Examples

```
from sklearn.metrics import roc_curve, roc_auc_score; fpr, tpr, _ = roc_curve(y_true, scores); auc = roc_auc_score(y_true, scores)
```

## Evaluation signals

- Row counts of subsets (a), (b), (c) match reported TP/TN sizes (ClinVar: 3,887/10,494; 19,888/16,694; 16,531/15,531; HGMD/GnomAD: 63,712/100,000; 96,569/100,000; 90,004/97,828)
- All computed AUC values fall within [0,1] for all seven methods under all three subset criteria of both benchmark conditions
- Computed AUCs reproduce the reported trends of Figure 2a-c (e.g. eDiVA-Score highest at 0.95/0.90 vs CADD 0.91/0.74 on ClinVar; eDiVA-Score 0.77 slightly below MetaLR 0.80, Revel 0.82, M-CAP 0.85 on the HGMD-only subset (a))
- Output CSV exists and contains one row per condition × method, sorted by condition and method
- Training variants used for the eDiVA-Score random forest are confirmed excluded from the test sets

## Limitations

- Missing scores are handled inconsistently across subset criteria: subset (a) drops variants lacking Revel/M-CAP scores, while subset (b) imputes a default value of 0 for missing scores, which affects AUC comparability
- eDiVA-Score training variants must be excluded from benchmark sets to avoid optimistic AUC inflation; failure to exclude invalidates the comparison
- HGMD 'DM?' categories are treated as true positives, so the HGMD-only benchmark relies on pathogenicity assertions of differing confidence

## Evidence

- [intro] c) eDiVA-Score, which estimates variant pathogenicity using a random forest model: "(c) eDiVA‐Score, which estimates variant pathogenicity using a random forest model"
- [methods] random forest with 1000 trees and five-fold cross validation: "training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees (Breiman, 2001; Hastie, Tibshirani, & Friedman, 2009) and five‐fold cross validation"
- [methods] rare variant subset criterion AF <= 0.01: "(c) using only rare variants (AF, ≤0.01) from the previous pool of variants"
- [abstract] machine learning pathogenicity predictor combining genomic and evolutionary signatures: "eDiVA features a machine learning-based variant pathogenicity predictor combining various genomic and evolutionary signatures."
- [readme] pre-calculated eDiVA-Scores repository for all exonic variants: "There is an available repository of pre-calculated eDiVA-Scores for all exonic variants [here](https://public_docs.crg.es/sossowski/MicrobeGenomes/human/eDiVA/eDiVA_score/)"
