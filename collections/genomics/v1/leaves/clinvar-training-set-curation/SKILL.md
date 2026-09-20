---
name: clinvar-training-set-curation
description: Use when training or retraining a variant pathogenicity classifier that
  needs a labeled ClinVar-derived case/control feature table with population-frequency,
  conservation, and functional-predictor features.
when_to_use_negative:
- The goal is to score variants for a new sample — a repository of pre-calculated
  eDiVA-Scores for all exonic variants already exists and can be used instead of retraining
- The analysis targets CNVs — CNV analysis is not yet available in eDiVA (integration
  pending)
- Prioritization of candidate variants by phenotype/inheritance is needed — that is
  eDiVA-Prioritize, not training-set curation
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3474
tools:
- name: eDiVA
  role: Provides the eDiVA-Annotate/eDiVA-DB annotations from which the 11 training
    features are computed
  repo: https://github.com/mbosio85/ediva
- name: randomForest
  role: R package used to train the binary classification random forest with 1000
    trees and five-fold cross validation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:13f9afb83ffe4d23f9718716e52f7a0445ba39d6753734308164e3e6a82b9081
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - eDiVA
  - randomForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# clinvar-training-set-curation

## Summary

Curates a balanced ClinVar/GnomAD training set and computes the 11 eDiVA-Annotate/eDiVA-DB features used to train the eDiVA-Score random forest variant pathogenicity classifier, so pathogenicity can be estimated independently of prior clinical information.

## When to use

Use when training or retraining a variant pathogenicity classifier that needs a labeled ClinVar-derived case/control feature table with population-frequency, conservation, and functional-predictor features.

## When NOT to use

- The goal is to score variants for a new sample — a repository of pre-calculated eDiVA-Scores for all exonic variants already exists and can be used instead of retraining
- The analysis targets CNVs — CNV analysis is not yet available in eDiVA (integration pending)
- Prioritization of candidate variants by phenotype/inheritance is needed — that is eDiVA-Prioritize, not training-set curation

## Inputs

- ClinVar variant records with clinical significance labels ('pathogenic', 'likely pathogenic', 'nonpathogenic')
- GnomAD exome variant list (variants not contained in ClinVar, with allele frequencies)
- eDiVA-Annotate/eDiVA-DB annotation table per variant (MAF, conservation, functional predictor, segmental duplication, ABB-score columns)

## Outputs

- Curated training-set feature table (11 features per variant with binary pathogenic/benign labels)
- Fitted random forest model object (R 'randomForest') with 1000 trees
- Five-fold cross-validated ROC/AUC for held-out pathogenic vs benign discrimination

## How to apply

Extract 15,000 random 'pathogenic' and 'likely pathogenic' variants from ClinVar as positive cases. Build the control set from 15,000 'nonpathogenic' ClinVar variants plus 100,000 random GnomAD exome variants absent from ClinVar, keeping the majority rare (allele frequency < 1%) so allele frequency does not dominate the classifier. From eDiVA-Annotate/eDiVA-DB annotations compute 11 features: maximum MAF of 1000 Genomes and GnomAD; PhastCons and PhyloP conservation in primates and mammals; Condel, Phred-scaled CADD, Eigen, and Mutation Assessor functional predictors; segmental duplication likelihood; and the ABB-score sequencing-error estimator. Impute missing feature values with the expected values for nonpathogenic (neutral) variants, except AF, where a missing value means zero (novel variant). Train a binary random forest with the R 'randomForest' package using 1000 classification trees and five-fold cross validation, then save the fitted model object and the curated feature table.

## Related tools

- **eDiVA** (Provides the eDiVA-Annotate/eDiVA-DB annotations from which the 11 training features are computed) — https://github.com/mbosio85/ediva
- **randomForest** (R package used to train the binary classification random forest with 1000 trees and five-fold cross validation)

## Examples

```
library(randomForest); rf <- randomForest(label ~ ., data = training_feature_table, ntree = 1000)
```

## Evaluation signals

- Class balance: exactly 15,000 pathogenic/likely pathogenic positives and 15,000 nonpathogenic ClinVar + 100,000 GnomAD controls
- Majority of the control set has allele frequency < 1% so AF does not dominate the classifier
- Feature table contains exactly the 11 described features; missing values are imputed with neutral-variant expected values, except AF where missing maps to zero
- The saved model object loads and produces pathogenicity probabilities in [0,1]
- Five-fold cross-validated ROC/AUC of held-out pathogenic vs benign variants is reported

## Limitations

- Training positives come only from ClinVar pathogenic/likely pathogenic labels, so classifier bias tracks ClinVar curation
- Adding 100,000 GnomAD variants as controls is an assumption that unreported variants are largely benign; the set is deliberately kept mostly rare (AF < 1%) to limit allele-frequency dominance
- Missing-feature imputation uses expected values for neutral variants, which can distort features for incompletely annotated pathogenic variants

## Evidence

- [methods] Random forest training setup: "training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees (Breiman, 2001; Hastie, Tibshirani, & Friedman, 2009) and five‐fold cross validation"
- [intro] eDiVA-Score module purpose: "(c) eDiVA‐Score, which estimates variant pathogenicity using a random forest model"
- [readme] eDiVA-Score coupled to annotation in the pipeline: "The two steps are joined because eDiVA-Score requires eDiVA-Annotate input columns to be calculated on the fly."
