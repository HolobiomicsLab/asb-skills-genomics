---
name: training-set-leakage-exclusion
description: Use when benchmarking a ClinVar-trained pathogenicity classifier (e.g.
  eDiVA-Score) on HGMD-derived benchmarks (DM/DM? positives vs GnomAD negatives) to
  remove training-set variants and avoid leakage-inflated AUC.
when_to_use_negative:
- The classifier under evaluation was not trained on ClinVar or HGMD-derived data,
  so no training-set overlap with the benchmark positives is possible
- The variant set has already been de-duplicated against the classifier's training
  variants and ClinVar annotations
- You are using the benchmark variants themselves for model training or feature engineering
  — exclusion is meant for evaluation only, not as a data-cleaning step
tools:
- name: eDiVA (eDiVA-Score)
  role: Random forest pathogenicity classifier trained on 15,000 ClinVar pathogenic
    variants; its training set defines the variants that must be excluded from benchmarks
  repo: https://github.com/mbosio85/ediva
- name: randomForest
  role: R package used to train the eDiVA-Score random forest (1000 binary classification
    trees, five-fold cross validation), whose training variants are excluded from
    evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:b109b84c38f66c2649ad2e0ac5f839b309659416cf1efe51f15187fa70a72542
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - eDiVA
  - eDiVA (eDiVA-Score)
  - randomForest
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# training-set-leakage-exclusion

## Summary

Excludes variants used to train a pathogenicity classifier (eDiVA-Score's 15,000 ClinVar pathogenic variants) and all ClinVar-annotated variants from HGMD/GnomAD evaluation sets before computing ROC/AUC, so benchmark results are not inflated by variants the model saw during training.

## When to use

Use when benchmarking a ClinVar-trained pathogenicity classifier (e.g. eDiVA-Score) on HGMD-derived benchmarks (DM/DM? positives vs GnomAD negatives) to remove training-set variants and avoid leakage-inflated AUC.

## When NOT to use

- The classifier under evaluation was not trained on ClinVar or HGMD-derived data, so no training-set overlap with the benchmark positives is possible
- The variant set has already been de-duplicated against the classifier's training variants and ClinVar annotations
- You are using the benchmark variants themselves for model training or feature engineering — exclusion is meant for evaluation only, not as a data-cleaning step

## Inputs

- HGMD variant set with pathogenicity categories (DM and DM? as positives; less likely pathogenic HGMD categories as negatives)
- GnomAD exome variants as negative (TN) set
- List of variants used for training eDiVA-Score (15,000 ClinVar pathogenic variants)
- Per-variant pathogenicity scores for eDiVA-Score, CADD, Eigen, DANN, Revel, M-CAP, and MetaLR

## Outputs

- Leakage-free benchmark variant sets per subset criterion (a), (b), (c) with reported TP/TN counts (e.g. HGMD/GnomAD: 63,712 TP and 100,000 TN for criterion (a))
- Per-method ROC curves and AUC values
- pathogenicity_benchmark_auc_table.csv with AUCs for all seven methods across three subset criteria of both benchmark conditions

## How to apply

eDiVA-Score is a random forest model (R 'randomForest' package, 1000 binary classification trees, five-fold cross validation) trained on 15,000 ClinVar pathogenic variants, so any benchmark variant present in ClinVar or in the training set must be removed before evaluation. Load the HGMD positive (DM/DM? as true positives, less likely pathogenic HGMD categories as negatives) and GnomAD negative variant sets, excluding all variants used for training of eDiVA-Score or present in ClinVar. Build the comparison subsets: (a) only variants having Revel and M-CAP scores available (HGMD/GnomAD: 63,712 TP and 100,000 TN; HGMD-only: 63,712 TP and 1,892 TN), (b) a random subset with missing scores assigned a default of 0 (HGMD: 96,569 TP and 7,376 TN), and (c) only rare variants with AF ≤ 0.01 (HGMD: 96,004 TP and 2,817 TN). Score variants with eDiVA-Score and CADD, Eigen, DANN, Revel, M-CAP, and MetaLR, generate ROC curves, and report per-method AUC (e.g. HGMD-only subset (a): eDiVA-Score AUC 0.77 vs M-CAP 0.85, Revel 0.82, MetaLR 0.80).

## Related tools

- **eDiVA (eDiVA-Score)** (Random forest pathogenicity classifier trained on 15,000 ClinVar pathogenic variants; its training set defines the variants that must be excluded from benchmarks) — https://github.com/mbosio85/ediva
- **randomForest** (R package used to train the eDiVA-Score random forest (1000 binary classification trees, five-fold cross validation), whose training variants are excluded from evaluation)

## Examples

```
git clone https://github.com/mbosio85/ediva.git
```

## Evaluation signals

- Verify zero overlap between the evaluation variants and the 15,000 ClinVar pathogenic variants used to train eDiVA-Score (no variant present in ClinVar remains in the benchmark sets)
- All AUC values in the output table lie within [0,1] for all seven methods (eDiVA, CADD, Eigen, DANN, Revel, M-CAP, MetaLR) under all three subset criteria of both benchmark conditions
- Subset sizes match the reported TP/TN counts (e.g. HGMD-only criterion (a): 63,712 TP and 1,892 TN; criterion (b): 96,569 TP and 7,376 TN; criterion (c): 96,004 TP and 2,817 TN)
- Expected performance range on the HGMD-only benchmark, subset (a): eDiVA-Score AUC ≈ 0.77, MetaLR ≈ 0.80, Revel ≈ 0.82, M-CAP ≈ 0.85
- Compare AUC with and without exclusion: leakage-containing runs should show suspiciously higher AUC for the ClinVar-trained classifier

## Limitations

- eDiVA-Score was trained on ClinVar pathogenic variants, so any ClinVar-annotated variant left in an HGMD/GnomAD benchmark inflates apparent performance; the exclusion is only as complete as the ClinVar/HGMD overlap mapping
- Variant scoring coverage differs across methods: criterion (a) restricts to variants with Revel and M-CAP scores, while criterion (b) assigns a default score of 0 to missing values, which can bias per-method comparisons
- The HGMD-only benchmark labels DM/DM? as positives and other HGMD categories as negatives, but HGMD categories are curated rather than ground-truth labels, and the wider benchmark relies on semisynthetic trios rather than fully real disease cohorts

## Evidence

- [methods] eDiVA-Score is a random forest classifier with 1000 trees and five-fold cross validation using eleven features, trained on 15,000 ClinVar pathogenic variants: [evidence span withheld — anchor status: not_found]
- [intro] known disease variants from ClinVar (Landrum et al., 2014) or HGMD: "known disease variants from ClinVar (Landrum et al., 2014) or HGMD (Stenson et al., 2017)"
- [readme] There is an available repository of pre-calculated eDiVA-Scores for all exonic variants: "There is an available repository of pre-calculated eDiVA-Scores for all exonic variants [here](https://public_docs.crg.es/sossowski/MicrobeGenomes/human/eDiVA/eDiVA_score/)"
