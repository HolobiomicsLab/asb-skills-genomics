---
name: clinvar-variant-label-classification
description: Use when a variant set already carries ClinVar pathogenic/benign labels
  and the goal is to quantify and compare classifier performance (AUC) for distinguishing
  disease-causing from benign coding variants.
when_to_use_negative:
- The variant set lacks ClinVar pathogenic/benign (or equivalent trusted) labels —
  there is no ground truth to compute ROC/AUC
- Variants under evaluation overlap the eDiVA-Score training set (15,000 ClinVar pathogenic
  variants), which would inflate benchmark performance
- The goal is de novo prioritization of an uncharacterized disease trio rather than
  retrospective classifier benchmarking — use eDiVA-Prioritize inheritance filtering
  instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0634
- http://edamontology.org/topic_3577
tools:
- name: eDiVA-Score
  role: Random forest classifier under evaluation; combines genomic and evolutionary
    signatures to predict variant pathogenicity
  repo: https://github.com/mbosio85/ediva
- name: CADD
  role: Competing pathogenicity classifier whose scores are compared on the same ClinVar
    subsets
- name: DANN
  role: Competing pathogenicity classifier included in the benchmark
- name: Eigen
  role: Competing pathogenicity classifier included in the benchmark
- name: Revel
  role: Competing classifier whose score availability defines benchmark subset (a)
- name: M-CAP
  role: Competing classifier whose score availability defines benchmark subset (a)
- name: MetaLR
  role: Competing pathogenicity classifier included in the benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:84b1cc0b1ae49690beb827aecf746733f858a2dbb449319f0591d949a91c0ae8
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0634
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# clinvar-variant-label-classification

## Summary

Uses ClinVar's curated 'pathogenic' and 'benign' labels as binary ground-truth (TP/TN) to benchmark variant pathogenicity classifiers such as eDiVA-Score against CADD, DANN, Eigen, Revel, M-CAP, and MetaLR via ROC/AUC analysis.

## When to use

Use when a variant set already carries ClinVar pathogenic/benign labels and the goal is to quantify and compare classifier performance (AUC) for distinguishing disease-causing from benign coding variants.

## When NOT to use

- The variant set lacks ClinVar pathogenic/benign (or equivalent trusted) labels — there is no ground truth to compute ROC/AUC
- Variants under evaluation overlap the eDiVA-Score training set (15,000 ClinVar pathogenic variants), which would inflate benchmark performance
- The goal is de novo prioritization of an uncharacterized disease trio rather than retrospective classifier benchmarking — use eDiVA-Prioritize inheritance filtering instead

## Inputs

- ClinVar test set with 'pathogenic' and 'benign' labels (excluding eDiVA-Score training variants)
- Precomputed functional impact scores for eDiVA-Score, CADD, DANN, Eigen, Revel, M-CAP, and MetaLR
- Allele frequency annotations (e.g. dbSNP, 1000 Genomes Project, discovEHR, Exome Variant Server, GnomAD exomes) for the rare-variant filter

## Outputs

- CSV table of per-method TP/TN counts and AUC values for subsets (a), (b), and (c), sorted by condition and method
- ROC curves per classifier per test condition

## How to apply

Load the ClinVar test set while excluding all variants used to train the eDiVA-Score random forest (1,000 trees, five-fold cross validation, trained on 15,000 ClinVar pathogenic variants), treating 'pathogenic' variants as true positives and 'benign' variants as true negatives. Collect precomputed functional impact scores for eDiVA-Score, CADD, Eigen, DANN, Revel, M-CAP, and MetaLR from their publications or database tracks. Build three test conditions: subset (a) restricted to variants having Revel and M-CAP scores available (ClinVar: 3,887 TP / 10,494 TN); subset (b) a random subset of all variants with missing scores assigned a default of 0 (ClinVar: 19,888 TP / 16,694 TN); and subset (c) only rare variants with allele frequency <= 0.01 from that pool (ClinVar: 16,531 TP / 15,531 TN). For each tool and each subset, compute ROC curves and AUC values (e.g. with scikit-learn's roc_curve/roc_auc_score) and assemble per-method TP/TN counts and AUCs into a CSV sorted by condition and method.

## Related tools

- **eDiVA-Score** (Random forest classifier under evaluation; combines genomic and evolutionary signatures to predict variant pathogenicity) — https://github.com/mbosio85/ediva
- **CADD** (Competing pathogenicity classifier whose scores are compared on the same ClinVar subsets)
- **DANN** (Competing pathogenicity classifier included in the benchmark)
- **Eigen** (Competing pathogenicity classifier included in the benchmark)
- **Revel** (Competing classifier whose score availability defines benchmark subset (a))
- **M-CAP** (Competing classifier whose score availability defines benchmark subset (a))
- **MetaLR** (Competing pathogenicity classifier included in the benchmark)

## Examples

```
from sklearn.metrics import roc_curve, roc_auc_score; fpr, tpr, _ = roc_curve(y_true, ediva_scores); auc = roc_auc_score(y_true, ediva_scores)
```

## Evaluation signals

- Row counts of subsets (a), (b), and (c) must match the reported ClinVar sizes: 3,887/10,494; 19,888/16,694; 16,531/15,531 TP/TN
- Computed AUCs must reproduce the trends of the paper's Figure 2a-c, with eDiVA-Score highest (AUC 0.95 and 0.90) versus CADD (0.91/0.74), DANN, Eigen, Revel, M-CAP, and MetaLR
- Verify no benchmark variant was among the 15,000 ClinVar pathogenic variants used to train the random forest
- Check that missing scores in subset (b) are consistently coded as the default value 0
- Confirm subset (c) contains only variants with allele frequency <= 0.01

## Limitations

- Benchmark relies on semisynthetic trios combining real WES data of a parent-child trio with thousands of pathogenic ClinVar variants (6,811 semisynthetic disease trios) rather than fully real disease cohorts
- Only variants with Revel and M-CAP scores available can be used in subset (a); assigning a default value of 0 to missing scores in subset (b) may bias tools with sparser coverage
- Training data come from the same ClinVar source used for benchmarking, so label curation biases may transfer to the classifier

## Evidence

- [methods] eDiVA-Score is a random forest classifier with 1000 trees and five-fold cross validation using eleven features, trained on 15,000 ClinVar pathogenic variants: "training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees"
- [methods] Rare variant filter defines subset (c): "(c) using only rare variants (AF, ≤0.01) from the previous pool of variants"
- [methods] Population frequency sources for AF filtering: "Each variant is annotated with population allele frequency information from the dbSNP"
- [readme] Precomputed eDiVA-Score tracks available for all exonic variants: "There is an available repository of pre-calculated eDiVA-Scores for all exonic variants"
- [discussion] Semisynthetic benchmark composition: "a highly realistic benchmark test combining real WES data of a parent–child trio with thousands of pathogenic ClinVar variants to generate 6,811 semisynthetic disease trios"
