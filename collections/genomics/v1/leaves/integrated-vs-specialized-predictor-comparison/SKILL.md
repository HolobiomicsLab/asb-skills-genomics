---
name: integrated-vs-specialized-predictor-comparison
description: Use when a trained random forest variant-pathogenicity model's feature
  importance must be extracted, ranked, and interpreted to identify which of the 11
  annotation features drive pathogenicity predictions.
when_to_use_negative:
- Input is already a ranked importance table — no need to recompute; go straight to
  comparison/interpretation
- The model is not a random forest (e.g. a logistic or boosting model) — randomForest::importance
  Gini/accuracy metrics do not apply
- Features outside the 11-feature eDiVA annotation schema (e.g. HPO phenotype or pedigree
  features used by eDiVA-Prioritize) are being evaluated — they are not part of the
  eDiVA-Score RF
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: eDiVA
  role: Source of the eDiVA-Score random forest pathogenicity predictor and the eDiVA-Annotate
    annotation columns it consumes
  repo: https://github.com/mbosio85/ediva
- name: randomForest
  role: R package used to train the 1000-tree model and compute per-feature variable
    importance (mean decrease in Gini impurity / accuracy)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:69c77e68a645e9a904f5223eea1643d9515970e386fe17341007dad8751905d3
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# integrated-vs-specialized-predictor-comparison

## Summary

Reproduce and interpret the feature-importance ranking of the eDiVA-Score random forest pathogenicity predictor, comparing integrated combination scores (Condel, Eigen, CADD) against specialized single predictors and conservation/allele-frequency features.

## When to use

Use when a trained random forest variant-pathogenicity model's feature importance must be extracted, ranked, and interpreted to identify which of the 11 annotation features drive pathogenicity predictions.

## When NOT to use

- Input is already a ranked importance table — no need to recompute; go straight to comparison/interpretation
- The model is not a random forest (e.g. a logistic or boosting model) — randomForest::importance Gini/accuracy metrics do not apply
- Features outside the 11-feature eDiVA annotation schema (e.g. HPO phenotype or pedigree features used by eDiVA-Prioritize) are being evaluated — they are not part of the eDiVA-Score RF

## Inputs

- Trained eDiVA-Score random forest model (R randomForest object, 1000 binary classification trees, five-fold cross validation, trained on 15,000 ClinVar pathogenic variants)
- 11-feature variant annotation table produced by eDiVA-Annotate (MAF, PhastCons/PhyloP conservation, Condel, CADD, Eigen, Mutation Assessor, segmental duplication, ABB-score)

## Outputs

- Ranked feature-importance table (variable_importance.csv) with feature names, importance metrics (mean decrease in Gini/accuracy), and rank positions
- Summary statement of whether top-ranked features are allele frequency, functional impact, and mammalian conservation features

## How to apply

Load the trained eDiVA-Score random forest model (R 'randomForest' package, 1000 binary classification trees, five-fold cross validation) together with its 11-feature annotation table built by eDiVA-Annotate. Compute per-feature variable importance with randomForest::importance using mean decrease in Gini impurity and/or mean decrease in accuracy for the 11 features: maximum MAF (1000 Genomes + GnomAD), four PhastCons/PhyloP conservation measures (primates and mammals), four functional impact predictors (Condel, Phred-scaled CADD, Eigen, Mutation Assessor), segmental duplication likelihood, and the ABB-score sequencing-error estimator. Rank the 11 features and specifically compare the ranks of integrated combination scores (Condel, Eigen, CADD) against specialized single predictors (Mutation Assessor, CADD) and conservation/allele-frequency features. Write the ranked table (feature names, importance metrics, rank positions) to variable_importance.csv and check whether top-ranked features correspond to allele frequency, functional impact, and mammalian conservation, consistent with the paper's reported ordering (Figure 1a).

## Related tools

- **eDiVA** (Source of the eDiVA-Score random forest pathogenicity predictor and the eDiVA-Annotate annotation columns it consumes) — https://github.com/mbosio85/ediva
- **randomForest** (R package used to train the 1000-tree model and compute per-feature variable importance (mean decrease in Gini impurity / accuracy))

## Examples

```
Rscript -e 'library(randomForest); rf <- readRDS("ediva_score_rf.rds"); imp <- importance(rf); write.csv(imp[order(-imp[,1]),,drop=FALSE], "variable_importance.csv")'
```

## Evaluation signals

- The exported variable_importance.csv contains exactly 11 rows matching the paper's feature set (maximum MAF, 4 conservation, 4 functional impact, segmental duplication, ABB-score)
- Maximum MAF, functional impact predictors, and mammalian conservation measures rank among the top features, matching the paper's reported ordering (Figure 1a)
- Importance metric values are non-negative and ranks are monotone with the metric values (no ties or ordering contradictions)
- The integrated combination scores (Condel, Eigen, CADD) appear alongside (not dominated by) specialized single predictors (Mutation Assessor, CADD) as reported in the paper's comparison

## Limitations

- Validation depends on the paper's reported ordering (Figure 1a); the provided section text confirms the model setup (1000 trees, five-fold CV, 11 features) but the exact per-feature importance values are not stated
- Variable importance from a random forest trained on 15,000 ClinVar pathogenic variants may not generalize to non-ClinVar variant distributions or non-exonic variants
- eDiVA-Score columns are calculated on the fly during eDiVA-Annotate; pre-computed scores exist only for all exonic variants, so non-exonic features cannot be reproduced from the released score repository

## Evidence

- [methods] eDiVA-Score is a random forest classifier with 1000 trees and five-fold cross validation using eleven features, trained on 15,000 ClinVar pathogenic variants: "training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees (Breiman, 2001; Hastie, Tibshirani, & Friedman, 2009) and five‐fold cross validation"
- [abstract] The model integrates genomic and evolutionary signatures as features: "eDiVA features a machine learning-based variant pathogenicity predictor combining various genomic and evolutionary signatures."
- [readme] Score features are computed during annotation, joining Annotate and Score steps: "The two steps are joined because eDiVA-Score requires eDiVA-Annotate input columns to be calculated on the fly."
