---
name: clinvar-gnomad-training-set-interpretation
description: Use when you need to verify which of the 11 eDiVA-Score annotation features
  drive pathogenicity predictions, or when interpreting how ClinVar-derived pathogenicity
  classifiers weight allele frequency, functional impact, and conservation signals.
when_to_use_negative:
- Input is already a ranked feature-importance table or only a variant list without
  the trained model object
- The goal is variant filtering or prioritization for a pedigree (use eDiVA-Prioritize
  skills instead of model-interpretation)
- The classifier was not trained on ClinVar pathogenic variants with the paper's 11
  genomic and evolutionary features, making importance ranks incomparable to the reported
  ordering
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: eDiVA
  role: Source of the eDiVA-Score random forest pathogenicity predictor and its annotation
    pipeline whose feature importances are being interpreted
  repo: https://github.com/mbosio85/ediva
- name: randomForest
  role: R package used to train the 1000-tree random forest and to compute per-feature
    variable importance via randomForest::importance
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:5a64015bfa291e290eba78d1d3571756a743f107cbc6801119c5f7cd277ea3ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - eDiVA
  - randomForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# clinvar-gnomad-training-set-interpretation

## Summary

Reproduce and interpret the feature-importance ranking of the eDiVA-Score random forest, which was trained on ~15,000 ClinVar pathogenic variants using population allele frequencies (1000 Genomes, GnomAD), functional impact scores (Condel, CADD, Eigen, Mutation Assessor), and PhastCons/PhyloP conservation signatures.

## When to use

Use when you need to verify which of the 11 eDiVA-Score annotation features drive pathogenicity predictions, or when interpreting how ClinVar-derived pathogenicity classifiers weight allele frequency, functional impact, and conservation signals.

## When NOT to use

- Input is already a ranked feature-importance table or only a variant list without the trained model object
- The goal is variant filtering or prioritization for a pedigree (use eDiVA-Prioritize skills instead of model-interpretation)
- The classifier was not trained on ClinVar pathogenic variants with the paper's 11 genomic and evolutionary features, making importance ranks incomparable to the reported ordering

## Inputs

- Trained eDiVA-Score random forest model (R 'randomForest' object, 1000 trees, five-fold cross validation)
- 11-feature variant annotation table (maximum MAF from 1000 Genomes/GnomAD, PhastCons/PhyloP conservation, Condel, CADD, Eigen, Mutation Assessor, segmental duplication, ABB-score)
- Training set of ~15,000 ClinVar pathogenic variants

## Outputs

- variable_importance.csv with feature names, importance metrics (mean decrease in Gini impurity / accuracy), and rank positions
- Ranked feature-importance summary comparing integrated combination scores (Condel, Eigen, CADD) against specialized single predictors and conservation/allele-frequency features

## How to apply

Load the trained eDiVA-Score random forest model built with the R 'randomForest' package (1000 binary classification trees, five-fold cross validation) together with its 11-feature annotation input table. Compute per-feature variable importance with randomForest::importance using mean decrease in Gini impurity and/or mean decrease in accuracy for the features: maximum MAF (1000 Genomes + GnomAD), four PhastCons/PhyloP conservation measures (primates and mammals), four functional impact predictors (Condel, Phred-scaled CADD, Eigen, Mutation Assessor), segmental duplication likelihood, and the ABB-score sequencing-error estimator. Rank the 11 features and write the ranked table with feature names, importance metrics, and rank positions to variable_importance.csv. Validate that the top-ranked features correspond to population allele frequency, functional impact, and conservation in placental mammals, as reported in the paper (Figure 1a).

## Related tools

- **eDiVA** (Source of the eDiVA-Score random forest pathogenicity predictor and its annotation pipeline whose feature importances are being interpreted) — https://github.com/mbosio85/ediva
- **randomForest** (R package used to train the 1000-tree random forest and to compute per-feature variable importance via randomForest::importance)

## Examples

```
R> library(randomForest); imp <- importance(rf_model); write.csv(imp[order(-imp[, 'MeanDecreaseGini']), ], 'variable_importance.csv')
```

## Evaluation signals

- The exported variable_importance.csv places maximum MAF, functional impact predictors (Condel, CADD, Eigen), and mammalian conservation measures among the top-ranked features
- Importance metrics are computed for exactly 11 features matching the paper's feature list (max MAF, 4 conservation, 4 functional impact, segmental duplication, ABB-score)
- Ranks of the integrated combination scores (Condel, Eigen, CADD) can be compared against the specialized single predictors (Mutation Assessor, CADD) in a consistent ordering
- The model object reports 1000 binary classification trees and five-fold cross validation, matching the paper's training setup on 15,000 ClinVar pathogenic variants

## Limitations

- The reported importance ranking is tied to Figure 1a of the paper and the specific ClinVar training set; retraining on other variant sets may change the ordering
- Feature importances reflect Gini impurity/accuracy decreases on pathogenic variants and do not directly certify clinical classification performance
- eDiVA-Score scores are pre-computed only for exonic variants in the released repository, so interpretation is limited to that variant universe

## Evidence

- [methods] eDiVA-Score is a random forest classifier with 1000 trees and five-fold cross validation using eleven features, trained on 15,000 ClinVar pathogenic variants: "training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees (Breiman, 2001; Hastie, Tibshirani, & Friedman, 2009) and five‐fold cross validation"
- [results] Feature selection identified population allele frequency, functional impact, and conservation in placental mammals as the most important features of the eDiVA-Score random forest model: [evidence span withheld — anchor status: not_found]
- [readme] eDiVA-Annotate and eDiVA-Score run jointly in the pipeline so ranking scores are computed from annotation columns: "The two steps are joined because eDiVA-Score requires eDiVA-Annotate input columns to be calculated on the fly."
- [abstract] eDiVA features a machine learning-based variant pathogenicity predictor combining genomic and evolutionary signatures: "eDiVA features a machine learning-based variant pathogenicity predictor combining various genomic and evolutionary signatures."
