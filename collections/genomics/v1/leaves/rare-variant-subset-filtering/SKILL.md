---
name: rare-variant-subset-filtering
description: Use when benchmarking variant pathogenicity classifiers (e.g., eDiVA-Score
  vs CADD, DANN, Eigen, Revel, M-CAP, MetaLR) on ClinVar or HGMD variant sets and
  you need comparable test subsets with defined TP/TN labels.
when_to_use_negative:
- Input is already a scored, size-verified evaluation table — re-filtering would change
  the reported subset sizes and AUCs
- Variants lack allele frequency annotations or Revel/M-CAP scores, so subset criteria
  (a) and (c) cannot be applied as defined
- The goal is de novo variant discovery or trio prioritization rather than classifier
  benchmarking — use eDiVA-Prioritize inheritance-mode filtering instead
edam_topics:
- http://edamontology.org/topic_2885
- http://edamontology.org/topic_0625
tools:
- name: eDiVA-Score
  role: Random forest pathogenicity classifier (1000 binary classification trees,
    five-fold cross validation) whose ROC/AUC is evaluated on each filtered subset
  repo: https://github.com/mbosio85/ediva
- name: Revel
  role: Competing pathogenicity classifier; its score availability defines the complete-coverage
    subset (a)
- name: M-CAP
  role: Competing pathogenicity classifier; its score availability jointly with Revel
    defines subset (a)
- name: eDiVA
  role: Pipeline whose annotation output (eDiVA-Annotate combined.variants.supplement.ranked.csv)
    provides the variant-level scores and AF fields used for filtering
  repo: https://github.com/mbosio85/ediva
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:2dd7d8475614a95ba5960b7f15cf66b51d62c623acdd04cfead4e3cd020762c7
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2885
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# rare-variant-subset-filtering

## Summary

Construction of controlled benchmark subsets for comparing variant pathogenicity classifiers (eDiVA-Score, CADD, DANN, Eigen, Revel, M-CAP, MetaLR): a complete-score subset (Revel/M-CAP available), a random subset with missing scores defaulted to 0, and a rare-variant subset restricted to AF ≤ 0.01, each evaluated by ROC/AUC.

## When to use

Use when benchmarking variant pathogenicity classifiers (e.g., eDiVA-Score vs CADD, DANN, Eigen, Revel, M-CAP, MetaLR) on ClinVar or HGMD variant sets and you need comparable test subsets with defined TP/TN labels.

## When NOT to use

- Input is already a scored, size-verified evaluation table — re-filtering would change the reported subset sizes and AUCs
- Variants lack allele frequency annotations or Revel/M-CAP scores, so subset criteria (a) and (c) cannot be applied as defined
- The goal is de novo variant discovery or trio prioritization rather than classifier benchmarking — use eDiVA-Prioritize inheritance-mode filtering instead

## Inputs

- ClinVar variant set with 'pathogenic' (TP) and 'benign' (TN) labels, excluding eDiVA-Score training variants
- HGMD variant set with DM/DM? vs other categories as TP/TN labels (HGMD vs GnomAD and HGMD-only conditions)
- GnomAD negative variant set (100,000 TN) for the HGMD vs GnomAD benchmark
- Precomputed per-variant scores for eDiVA-Score, CADD, Eigen, DANN, Revel, M-CAP, and MetaLR
- Per-variant allele frequency annotations (dbSNP, 1000 Genomes Project, Exome Variant Server, GnomAD exomes, discovEHR)

## Outputs

- Three benchmark subsets per benchmark condition: (a) complete Revel/M-CAP coverage, (b) random subset with 0-imputed missing scores, (c) AF ≤ 0.01 rare-variant subset
- ROC curves per method per subset
- Per-method TP/TN counts and AUC values assembled into pathogenicity_benchmark_auc_table.csv, sorted by condition and method

## How to apply

Load the labeled variant sets (ClinVar 'pathogenic' as TP and 'benign' as TN, or HGMD DM/DM? as TP), excluding all variants used to train the eDiVA-Score random forest (R 'randomForest' package, 1000 trees, five-fold cross validation). Build subset (a) from variants having both Revel and M-CAP scores available, subset (b) as a random subset of all variants with a default score of 0 assigned to missing values, and subset (c) by retaining only rare variants with allele frequency ≤ 0.01 from the (b) pool. Score each variant with all seven methods, generate ROC curves per method and subset, and compute AUC (e.g., with scikit-learn's roc_curve/roc_auc_score). The rationale is to separate performance differences due to score coverage from performance on the rare-variant regime relevant to clinical diagnostics.

## Related tools

- **eDiVA-Score** (Random forest pathogenicity classifier (1000 binary classification trees, five-fold cross validation) whose ROC/AUC is evaluated on each filtered subset) — https://github.com/mbosio85/ediva
- **Revel** (Competing pathogenicity classifier; its score availability defines the complete-coverage subset (a))
- **M-CAP** (Competing pathogenicity classifier; its score availability jointly with Revel defines subset (a))
- **eDiVA** (Pipeline whose annotation output (eDiVA-Annotate combined.variants.supplement.ranked.csv) provides the variant-level scores and AF fields used for filtering) — https://github.com/mbosio85/ediva

## Examples

```
from sklearn.metrics import roc_auc_score; auc = roc_auc_score(y_true, scores)
```

## Evaluation signals

- Subset row counts match the reported TP/TN sizes: ClinVar 3,887/10,494 (a), 19,888/16,694 (b), 16,531/15,531 (c); HGMD/GnomAD 63,712/100,000 (a), 96,569/100,000 (b), 90,004/97,828 (c)
- Output table pathogenicity_benchmark_auc_table.csv contains AUC values within [0,1] for all seven methods (eDiVA-Score, CADD, Eigen, DANN, Revel, M-CAP, MetaLR) under all three subset criteria
- Computed AUCs reproduce the trends of Figure 2a-c: eDiVA-Score best on ClinVar (AUC 0.95 and 0.90 vs CADD 0.91/0.74)
- On HGMD-only complete-coverage subset, eDiVA-Score AUC ≈ 0.77, below MetaLR (0.80), Revel (0.82), and M-CAP (0.85)
- Missing scores in subset (b) are verifiably imputed with a default value of 0

## Limitations

- Assigning a default score of 0 to missing values in subset (b) can distort ROC/AUC for methods with incomplete coverage
- Benchmark relies on exclusion of all eDiVA-Score training variants; residual label leakage from public databases (ClinVar/HGMD) cannot be fully ruled out
- The AF ≤ 0.01 cutoff substantially shrinks the negative sets (e.g., HGMD-only TN drops from 7,376 to 2,817), reducing statistical power
- Benchmark is based on precomputed published scores rather than uniform re-scoring, so track versions of CADD/DANN/Eigen/Revel/M-CAP/MetaLR may differ

## Evidence

- [methods] Rare variant subset criterion: "(c) using only rare variants (AF, ≤0.01) from the previous pool of variants"
- [methods] Random forest scoring: "training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees"
- [readme] Filtered output sorted by eDiVA-Score: "combined.variants.supplement.filtered.dominant_inherited.csv : Only the variants passing the filters are reported here, sorted by eDiVA-Score"
- [methods] Functional and population-genomic filtering: "filtering based on functional and population genomic features"
