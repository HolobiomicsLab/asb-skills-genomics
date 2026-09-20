---
name: hgmd-gnomad-benchmark-set-construction
description: 'Use when benchmarking variant pathogenicity classifiers: you need labeled
  positive/negative variant sets from HGMD DM/DM? and GnomAD (or less-pathogenic HGMD
  categories), with defined subsets (all-scored, random with 0-fill, AF ≤ 0.01 rare)
  for AUC comparison.'
when_to_use_negative:
- The goal is causal variant prioritization in a family/trio (dominant de novo, compound
  heterozygous, X-linked, etc.) — use eDiVA-Prioritize's inheritance filters instead,
  not the benchmark construction.
- Variant calling or alignment quality is being evaluated (eDiVA-Predict benchmark
  with semisynthetic ClinVar-introduced trios), not pathogenicity-score discrimination.
- Input variants lack HGMD category labels and comparator tool scores, so neither
  the DM/DM? labeling scheme nor subset criterion (a) (Revel and M-CAP scores available)
  can be defined.
tools:
- name: eDiVA
  role: Source of the eDiVA-Score random forest pathogenicity predictor (one of the
    seven methods scored for AUC)
  repo: https://github.com/mbosio85/ediva
- name: randomForest
  role: R package used to train eDiVA-Score (1000 binary classification trees, five-fold
    cross validation) and produce scores for the ROC/AUC comparison
- name: Revel
  role: Competing pathogenicity classifier; score availability defines subset criterion
    (a), and its AUC (0.82 on HGMD-only) is a reference point
- name: M-CAP
  role: Competing pathogenicity classifier; score availability defines subset criterion
    (a), and it achieved the best AUC (0.85) on the HGMD-only benchmark
- name: MetaLR
  role: Competing pathogenicity classifier scored for AUC comparison (0.80 on HGMD-only
    benchmark)
- name: CADD
  role: Competing pathogenicity classifier scored for AUC comparison
- name: Eigen
  role: Competing pathogenicity classifier scored for AUC comparison
- name: DANN
  role: Competing pathogenicity classifier scored for AUC comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:afe9f190dd9eced9701d3d289a8a4121c7e9f82327fd08dfe1968f35d63f6c39
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
  - randomForest
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# hgmd-gnomad-benchmark-set-construction

## Summary

Constructs HGMD/GnomAD and HGMD-only (DM/DM? vs less pathogenic categories) benchmark sets with three subset criteria to compute ROC-AUC for eDiVA-Score versus six competing pathogenicity classifiers (CADD, Eigen, DANN, Revel, M-CAP, MetaLR).

## When to use

Use when benchmarking variant pathogenicity classifiers: you need labeled positive/negative variant sets from HGMD DM/DM? and GnomAD (or less-pathogenic HGMD categories), with defined subsets (all-scored, random with 0-fill, AF ≤ 0.01 rare) for AUC comparison.

## When NOT to use

- The goal is causal variant prioritization in a family/trio (dominant de novo, compound heterozygous, X-linked, etc.) — use eDiVA-Prioritize's inheritance filters instead, not the benchmark construction.
- Variant calling or alignment quality is being evaluated (eDiVA-Predict benchmark with semisynthetic ClinVar-introduced trios), not pathogenicity-score discrimination.
- Input variants lack HGMD category labels and comparator tool scores, so neither the DM/DM? labeling scheme nor subset criterion (a) (Revel and M-CAP scores available) can be defined.

## Inputs

- HGMD variant set with severity categories (DM and DM? as positives; other HGMD categories as negatives for the HGMD-only benchmark)
- GnomAD exome variant set (negatives for the HGMD vs GnomAD benchmark)
- Per-variant pathogenicity scores from eDiVA-Score, CADD, Eigen, DANN, Revel, M-CAP, and MetaLR (obtained from the respective publications)
- Exclusion list of variants used for eDiVA-Score training and variants present in ClinVar
- Allele frequency annotations (for the AF ≤ 0.01 rare-variant subset)

## Outputs

- pathogenicity_benchmark_auc_table.csv: per-method AUC values for eDiVA, CADD, Eigen, DANN, Revel, M-CAP, MetaLR across the three subset criteria of both benchmark conditions
- Per-tool ROC curves for each benchmark set and subset criterion

## How to apply

Load HGMD positives and GnomAD negatives (or label HGMD DM/DM? as TP and all other HGMD categories as negative for the HGMD-only benchmark), excluding all variants used to train eDiVA-Score or present in ClinVar to avoid label leakage. Build subset (a) restricted to variants with Revel and M-CAP scores available (HGMD/GnomAD: 63,712 TP / 100,000 TN; HGMD-only: 63,712 TP / 1,892 TN), subset (b) as a random subset with missing scores assigned a default value of 0 (HGMD/GnomAD: 96,569 TP / 100,000 TN; HGMD-only: 96,569 TP / 7,376 TN), and subset (c) restricted to rare variants with AF ≤ 0.01 (HGMD/GnomAD: 90,004 TP / 97,828 TN; HGMD-only: 96,004 TP / 2,817 TN). Generate a ROC curve per tool per subset, scoring variants with the eDiVA-Score random forest (R 'randomForest' package, 1000 binary classification trees, five-fold cross validation) and with CADD, Eigen, DANN, Revel, M-CAP, and MetaLR, then compute AUC per method and subset. Rationale: GnomAD variants serve as presumably-neutral negatives, while the HGMD-only design avoids GnomAD-derived confounding by contrasting highly likely pathogenic (DM/DM?) categories against less likely pathogenic HGMD categories.

## Related tools

- **eDiVA** (Source of the eDiVA-Score random forest pathogenicity predictor (one of the seven methods scored for AUC)) — https://github.com/mbosio85/ediva
- **randomForest** (R package used to train eDiVA-Score (1000 binary classification trees, five-fold cross validation) and produce scores for the ROC/AUC comparison)
- **Revel** (Competing pathogenicity classifier; score availability defines subset criterion (a), and its AUC (0.82 on HGMD-only) is a reference point)
- **M-CAP** (Competing pathogenicity classifier; score availability defines subset criterion (a), and it achieved the best AUC (0.85) on the HGMD-only benchmark)
- **MetaLR** (Competing pathogenicity classifier scored for AUC comparison (0.80 on HGMD-only benchmark))
- **CADD** (Competing pathogenicity classifier scored for AUC comparison)
- **Eigen** (Competing pathogenicity classifier scored for AUC comparison)
- **DANN** (Competing pathogenicity classifier scored for AUC comparison)

## Examples

```
R: library(randomForest); rf <- randomForest(label ~ ., data=benchmark_set, ntree=1000); auc(roc(rf$votes[,2], benchmark_set$label))
```

## Evaluation signals

- Output table pathogenicity_benchmark_auc_table.csv exists and contains AUC values within [0,1] for all seven methods (eDiVA, CADD, Eigen, DANN, Revel, M-CAP, MetaLR) under all three subset criteria of both benchmark conditions.
- Subset sizes match the reported TP/TN counts: e.g. subset (a) HGMD/GnomAD = 63,712 TP / 100,000 TN; subset (b) HGMD-only = 96,569 TP / 7,376 TN; subset (c) HGMD-only = 96,004 TP / 2,817 TN.
- No variant used for eDiVA-Score training or present in ClinVar appears in the benchmark sets (leakage check).
- HGMD-only benchmark reproduction sanity check: eDiVA-Score AUC ≈ 0.77, MetaLR ≈ 0.80, Revel ≈ 0.82, M-CAP ≈ 0.85 on the all-methods-scored subset.
- In subset (b), every missing comparator score was filled with the default value 0 (no NaNs propagate into the ROC computation).

## Limitations

- Score availability differs by tool, forcing three separate subset criteria; on the common-scored subset (a) eDiVA-Score (AUC 0.77) trails M-CAP (0.85), Revel (0.82), and MetaLR (0.80) on the HGMD-only benchmark.
- Assigning a default score of 0 to missing values in subset (b) can bias per-method AUCs relative to methods with complete score coverage.
- The benchmark excludes CNV analysis (not yet available in eDiVA) and relies on semisynthetic/curated databases rather than fully real disease cohorts.
- The Iberian allele frequency database (geevs.crg.eu) used alongside public frequency resources (dbSNP, discovEHR, 1000 Genomes Project, Exome Variant Server, GnomAD exomes) is unpublished.

## Evidence

- [methods] training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees: "training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees"
- [methods] using only rare variants (AF, ≤0.01) from the previous pool of variants: "using only rare variants (AF, ≤0.01) from the previous pool of variants"
- [intro] known disease variants from ClinVar (Landrum et al., 2014) or HGMD: "known disease variants from ClinVar (Landrum et al., 2014) or HGMD"
- [readme] There is an available repository of pre-calculated eDiVA-Scores for all exonic variants: "There is an available repository of pre-calculated eDiVA-Scores for all exonic variants"
