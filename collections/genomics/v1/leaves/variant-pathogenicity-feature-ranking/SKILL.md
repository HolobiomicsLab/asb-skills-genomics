---
name: variant-pathogenicity-feature-ranking
description: Use when a trained variant-classification random forest (e.g. eDiVA-Score
  with 11 annotation features) must be inspected to determine which features — allele
  frequency, functional impact, conservation — contribute most to pathogenicity prediction.
when_to_use_negative:
- The variant annotations have not yet been computed by eDiVA-Annotate — eDiVA-Score
  requires the annotation columns to be calculated first
- The goal is causal-variant prioritization in a family/pedigree (use the eDiVA-Prioritize
  inheritance-mode filtering workflow instead of model introspection)
- The classifier is not a tree ensemble — randomForest::importance Gini/accuracy metrics
  do not apply to non-forest models
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3676
- http://edamontology.org/topic_0625
tools:
- name: eDiVA
  role: Framework whose eDiVA-Score module trains the random forest pathogenicity
    predictor and produces the annotated feature table used for importance ranking
  repo: https://github.com/mbosio85/ediva
- name: randomForest
  role: R package used to train the 1000-tree binary classification forest with five-fold
    cross validation and to compute per-feature variable importance
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:01f5f9e0f8f2d8c78275dafc00da536ada9d6a754356b3e252ea3c31042c2339
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - eDiVA
  - randomForest
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3676
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# variant-pathogenicity-feature-ranking

## Summary

Reproduces the feature-importance ranking of the eDiVA-Score random forest pathogenicity predictor (R 'randomForest', 1000 trees, five-fold cross validation, 11 annotation features), identifying which genomic and evolutionary signatures drive variant pathogenicity classification.

## When to use

Use when a trained variant-classification random forest (e.g. eDiVA-Score with 11 annotation features) must be inspected to determine which features — allele frequency, functional impact, conservation — contribute most to pathogenicity prediction.

## When NOT to use

- The variant annotations have not yet been computed by eDiVA-Annotate — eDiVA-Score requires the annotation columns to be calculated first
- The goal is causal-variant prioritization in a family/pedigree (use the eDiVA-Prioritize inheritance-mode filtering workflow instead of model introspection)
- The classifier is not a tree ensemble — randomForest::importance Gini/accuracy metrics do not apply to non-forest models

## Inputs

- Trained eDiVA-Score random forest model object (R randomForest, 1000 binary classification trees, five-fold cross validation)
- 11-feature variant annotation table with maximum MAF (1000 Genomes + GnomAD), PhastCons/PhyloP conservation measures, Condel, CADD, Eigen, Mutation Assessor scores, segmental duplication likelihood, and ABB-score

## Outputs

- Ranked variable-importance table (variable_importance.csv) with feature names, importance metrics, and rank positions
- Summary statement of whether top-ranked features are allele frequency, functional impact, and mammalian conservation measures

## How to apply

Load the trained eDiVA-Score random forest model (built with the R 'randomForest' package using 1000 binary classification trees and five-fold cross validation, trained on ~15,000 ClinVar pathogenic variants) together with its 11-feature annotation input table. Compute per-feature variable importance via randomForest::importance (mean decrease in Gini impurity and/or mean decrease in accuracy) for the 11 features: maximum MAF (1000 Genomes + GnomAD), four PhastCons/PhyloP conservation measures (primates and mammals), four functional impact predictors (Condel, Phred-scaled CADD, Eigen, Mutation Assessor), segmental duplication likelihood, and the ABB-score sequencing-error estimator. Rank the 11 features and compare integrated combination scores (Condel, Eigen, CADD) against specialized single predictors and the conservation/allele-frequency features. Export the ranked importance table and verify that the top-ranked features correspond to population allele frequency, functional impact, and conservation in placental mammals, consistent with the paper's Figure 1a ordering.

## Related tools

- **eDiVA** (Framework whose eDiVA-Score module trains the random forest pathogenicity predictor and produces the annotated feature table used for importance ranking) — https://github.com/mbosio85/ediva
- **randomForest** (R package used to train the 1000-tree binary classification forest with five-fold cross validation and to compute per-feature variable importance)

## Examples

```
Rscript -e 'library(randomForest); rf <- readRDS("ediva_score_rf.rds"); imp <- importance(rf); write.csv(imp[order(-imp[,1]),,drop=FALSE], "variable_importance.csv")'
```

## Evaluation signals

- The exported ranking places maximum MAF, functional impact predictors, and mammalian conservation measures among the top features, matching the paper's Figure 1a ordering
- All 11 model features appear in the importance table: max MAF, four PhastCons/PhyloP measures, Condel, CADD, Eigen, Mutation Assessor, segmental duplication likelihood, and ABB-score
- Importance metrics are the standard randomForest outputs (mean decrease in Gini impurity and/or mean decrease in accuracy), not custom scores
- The model inspected was trained with the documented hyperparameters: 1000 binary classification trees and five-fold cross validation

## Limitations

- The ranking reflects the paper's training setup (ClinVar pathogenic variants); importance values may shift with different training sets or annotation sources
- The ABB-score is a sequencing-error estimator and segmental duplication likelihood is a technical artifact feature — their ranks must be interpreted as artifact-control signals rather than biological pathogenicity drivers
- Conservation features are split between primate and placental-mammal PhastCons/PhyloP measures, so ranks among the four conservation columns require separate interpretation

## Evidence

- [methods] eDiVA-Score is a random forest classifier with 1000 trees and five-fold cross validation using eleven features, trained on 15,000 ClinVar pathogenic variants: "training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees (Breiman, 2001; Hastie, Tibshirani, & Friedman, 2009) and five‐fold cross validation"
- [intro] eDiVA-Score estimates variant pathogenicity using a random forest model: "(c) eDiVA‐Score, which estimates variant pathogenicity using a random forest model"
- [abstract] The machine-learning predictor combines genomic and evolutionary signatures: "eDiVA features a machine learning-based variant pathogenicity predictor combining various genomic and evolutionary signatures."
- [readme] Annotation and scoring are run together because score features are computed on the fly from annotation columns: "The two steps are joined because eDiVA-Score requires eDiVA-Annotate input columns to be calculated on the fly."
