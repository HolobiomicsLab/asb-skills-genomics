---
name: variant-pathogenicity-classifier-benchmarking
description: Use when comparing pathogenicity prediction scores across methods on
  labeled variant sets (ClinVar pathogenic/benign, HGMD DM/DM? vs other categories,
  GnomAD negatives) and need reproducible AUC tables and ROC curves.
when_to_use_negative:
- Input is an unlabeled variant list with no pathogenic/benign or DM/non-DM assignments
  — ROC/AUC benchmarking cannot be computed
- You want to train a new pathogenicity classifier rather than compare existing scored
  classifiers on fixed benchmark sets
- The evaluation requires CNV or structural variant classification, which is not covered
  (CNV analysis was not yet available in eDiVA)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: eDiVA-Score
  role: Random forest pathogenicity predictor under benchmarking; scores variants
    using eleven genomic/evolutionary features
  repo: https://github.com/mbosio85/ediva
- name: randomForest
  role: R package used to train the eDiVA-Score random forest (1000 binary classification
    trees, five-fold cross validation) whose held-out discrimination is evaluated
    as ROC/AUC
- name: CADD
  role: Competing variant deleteriousness predictor whose scores are benchmarked against
    eDiVA-Score
- name: Revel
  role: Competing pathogenicity classifier; availability of Revel scores defines subset
    criterion (a)
- name: M-CAP
  role: Competing pathogenicity classifier; availability of M-CAP scores defines subset
    criterion (a)
- name: MetaLR
  role: Competing pathogenicity classifier included in the AUC comparison
- name: DANN
  role: Competing pathogenicity classifier included in the AUC comparison
- name: Eigen
  role: Competing pathogenicity classifier included in the AUC comparison
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_002
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:10a789ada19758bbd3b85459e462f025493a6607e15369bc89d791789ef15446
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# variant-pathogenicity-classifier-benchmarking

## Summary

Benchmark variant pathogenicity classifiers (eDiVA-Score, CADD, DANN, Eigen, Revel, M-CAP, MetaLR) by computing ROC/AUC over ClinVar and HGMD-derived positive/negative variant sets under three defined subset criteria (a/b/c).

## When to use

Use when comparing pathogenicity prediction scores across methods on labeled variant sets (ClinVar pathogenic/benign, HGMD DM/DM? vs other categories, GnomAD negatives) and need reproducible AUC tables and ROC curves.

## When NOT to use

- Input is an unlabeled variant list with no pathogenic/benign or DM/non-DM assignments — ROC/AUC benchmarking cannot be computed
- You want to train a new pathogenicity classifier rather than compare existing scored classifiers on fixed benchmark sets
- The evaluation requires CNV or structural variant classification, which is not covered (CNV analysis was not yet available in eDiVA)

## Inputs

- ClinVar variant sets labeled pathogenic (TP) and benign (TN), excluding eDiVA-Score training variants
- HGMD variant sets (DM/DM? positives vs less likely pathogenic categories as negatives)
- GnomAD exome variants not present in ClinVar as negatives for the HGMD/GnomAD benchmark
- Precomputed per-variant scores for eDiVA-Score, CADD, DANN, Eigen, Revel, M-CAP, and MetaLR from publications/database tracks
- Allele frequency annotations for the rare-variant (AF ≤ 0.01) subset criterion

## Outputs

- pathogenicity_benchmark_auc_table.csv with per-method AUC values for all three subset criteria of both benchmark conditions
- ROC curves per tool per subset
- Per-method TP/TN counts for subsets (a), (b), and (c)

## How to apply

Load labeled TP/TN variant sets, excluding all variants used to train the eDiVA-Score random forest (R 'randomForest' package, 1000 binary classification trees, five-fold cross validation) or present in ClinVar. Collect per-variant scores for the seven tools, then build three subset criteria: (a) only variants with Revel and M-CAP scores available, (b) a random subset of all variants with missing scores assigned a default value of 0, and (c) only rare variants with AF ≤ 0.01 from the previous pool. For the HGMD-only benchmark, label DM and DM? as true positives and any other HGMD category as negatives. Compute ROC curves and AUC per method and subset (e.g. scikit-learn roc_curve/roc_auc_score) and assemble per-method TP/TN counts and AUC values into a CSV sorted by condition and method. Validate that subset sizes match the reported TP/TN counts (e.g. ClinVar: 3,887/10,494; 19,888/16,694; 16,531/15,531) and that AUCs reproduce the reported trends.

## Related tools

- **eDiVA-Score** (Random forest pathogenicity predictor under benchmarking; scores variants using eleven genomic/evolutionary features) — https://github.com/mbosio85/ediva
- **randomForest** (R package used to train the eDiVA-Score random forest (1000 binary classification trees, five-fold cross validation) whose held-out discrimination is evaluated as ROC/AUC)
- **CADD** (Competing variant deleteriousness predictor whose scores are benchmarked against eDiVA-Score)
- **Revel** (Competing pathogenicity classifier; availability of Revel scores defines subset criterion (a))
- **M-CAP** (Competing pathogenicity classifier; availability of M-CAP scores defines subset criterion (a))
- **MetaLR** (Competing pathogenicity classifier included in the AUC comparison)
- **DANN** (Competing pathogenicity classifier included in the AUC comparison)
- **Eigen** (Competing pathogenicity classifier included in the AUC comparison)

## Examples

```
from sklearn.metrics import roc_curve, roc_auc_score; fpr, tpr, _ = roc_curve(y_true, scores); auc = roc_auc_score(y_true, scores)
```

## Evaluation signals

- Row counts of subsets (a), (b), and (c) match reported TP/TN sizes (ClinVar: 3,887/10,494; 19,888/16,694; 16,531/15,531; HGMD/GnomAD: 63,712/100,000; HGMD: 63,712/1,892)
- All computed AUC values fall within [0,1] for all seven methods under all three subset criteria of both benchmark conditions
- Computed AUCs reproduce the trends reported in Figure 2a-c, e.g. eDiVA-Score AUC 0.95/0.90 on ClinVar versus CADD 0.91/0.74
- Output table exists with one row per condition × method combination, sorted by condition and method
- Trained random forest produces pathogenicity probabilities in [0,1]

## Limitations

- Missing scores must be handled explicitly: subset (a) restricts to variants with Revel and M-CAP scores, while subset (b) imputes a default value of 0 for missing scores, which can distort AUC comparisons
- HGMD-only benchmark is asymmetric (63,712 TP vs only 1,892 TN for subset (a)) and on that subset eDiVA-Score (AUC 0.77) was slightly below MetaLR (0.80), Revel (0.82), and M-CAP (0.85)
- All variants used to train eDiVA-Score and any variants present in ClinVar must be excluded from the HGMD/GnomAD benchmark to avoid circularity
- CNV analysis was not yet available in eDiVA, so benchmarking is limited to point variants

## Evidence

- [methods] training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees: "training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees (Breiman, 2001; Hastie, Tibshirani, & Friedman, 2009) and five‐fold cross validation"
- [methods] (c) using only rare variants (AF, ≤0.01) from the previous pool of variants: "(c) using only rare variants (AF, ≤0.01) from the previous pool of variants"
- [readme] There is an available repository of pre-calculated eDiVA-Scores for all exonic variants: "There is an available repository of pre-calculated eDiVA-Scores for all exonic variants"
- [abstract] eDiVA features a machine learning-based variant pathogenicity predictor combining various genomic and evolutionary signatures.: "eDiVA features a machine learning-based variant pathogenicity predictor combining various genomic and evolutionary signatures."
