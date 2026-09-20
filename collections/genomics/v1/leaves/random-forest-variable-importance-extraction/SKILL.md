---
name: random-forest-variable-importance-extraction
description: Use when a trained random forest model (e.g., eDiVA-Score's 1000-tree
  RF over 11 annotation features such as maximum MAF, PhastCons/PhyloP conservation,
  Condel, CADD, Eigen) must be interrogated to reproduce or report its feature-importance
  ranking.
when_to_use_negative:
- The input is already a ranked variable-importance table or the goal is only to read
  off a published ranking (Figure 1a) rather than recompute it
- The predictor is not a tree ensemble (e.g., a linear model, SVM, or logistic regression),
  where Gini-impurity importance is undefined
- The goal is scoring new variants' pathogenicity with eDiVA-Score (run eDiVA-Annotate
  + eDiVA-Score in the nextflow pipeline) rather than interpreting the model
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: randomForest
  role: R package used to train the 1000-tree random forest and to compute per-feature
    variable importance (mean decrease in Gini impurity / accuracy)
- name: eDiVA
  role: Supplies the eDiVA-Score random forest pathogenicity predictor and the 11-feature
    annotation table whose importance is extracted; ranks variants by the resulting
    pathogenicity probability
  repo: https://github.com/mbosio85/ediva
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:8a5182a485fc67b107c7f95dcc067f84f6cf34cc047712d1d2c6cad9f6a4dd4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - eDiVA
  - randomForest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# random-forest-variable-importance-extraction

## Summary

Extracts and ranks per-feature variable importance from a trained random forest classifier, specifically the eDiVA-Score pathogenicity predictor (1000 trees, R 'randomForest' package, five-fold cross validation), to report which of its 11 genomic/evolutionary annotation features drive variant pathogenicity prediction.

## When to use

Use when a trained random forest model (e.g., eDiVA-Score's 1000-tree RF over 11 annotation features such as maximum MAF, PhastCons/PhyloP conservation, Condel, CADD, Eigen) must be interrogated to reproduce or report its feature-importance ranking.

## When NOT to use

- The input is already a ranked variable-importance table or the goal is only to read off a published ranking (Figure 1a) rather than recompute it
- The predictor is not a tree ensemble (e.g., a linear model, SVM, or logistic regression), where Gini-impurity importance is undefined
- The goal is scoring new variants' pathogenicity with eDiVA-Score (run eDiVA-Annotate + eDiVA-Score in the nextflow pipeline) rather than interpreting the model

## Inputs

- Trained eDiVA-Score random forest model object (R 'randomForest' object, 1000 binary classification trees)
- 11-feature variant annotation table (maximum MAF from 1000 Genomes + GnomAD, PhastCons/PhyloP primate and mammal conservation, Condel, CADD, Eigen, Mutation Assessor, segmental duplication likelihood, ABB-score)

## Outputs

- variable_importance.csv with feature names, importance metrics (mean decrease in Gini impurity and/or accuracy), and rank positions
- Summary statement of whether top-ranked features correspond to allele frequency, functional impact, and mammalian conservation

## How to apply

Load the trained random forest object built with the R 'randomForest' package (1000 binary classification trees, five-fold cross validation, trained on 15,000 ClinVar pathogenic variants) together with its 11-feature annotation input table. Compute per-feature importance via randomForest::importance using mean decrease in Gini impurity and/or mean decrease in accuracy. Rank the 11 features — maximum MAF (1000 Genomes + GnomAD), four PhastCons/PhyloP conservation measures (primates and mammals), four functional impact predictors (Condel, Phred-scaled CADD, Eigen, Mutation Assessor), segmental duplication likelihood, and the ABB-score sequencing-error estimator — and compare integrated combination scores against specialized single predictors and conservation/allele-frequency features. Write feature names, importance metrics, and rank positions to a CSV, then validate that maximum MAF, functional impact, and mammalian conservation rank among the top features as reported in the paper (Figure 1a).

## Related tools

- **randomForest** (R package used to train the 1000-tree random forest and to compute per-feature variable importance (mean decrease in Gini impurity / accuracy))
- **eDiVA** (Supplies the eDiVA-Score random forest pathogenicity predictor and the 11-feature annotation table whose importance is extracted; ranks variants by the resulting pathogenicity probability) — https://github.com/mbosio85/ediva

## Examples

```
Rscript -e 'library(randomForest); rf <- readRDS("ediva_score_rf.rds"); imp <- importance(rf); write.csv(imp[order(-imp[,1]),,drop=FALSE], "variable_importance.csv")'
```

## Evaluation signals

- The exported ranking contains exactly the 11 features: maximum MAF, four PhastCons/PhyloP measures, Condel, CADD, Eigen, Mutation Assessor, segmental duplication, and ABB-score
- Top-ranked features include maximum MAF, functional impact predictors, and mammalian conservation measures, consistent with the paper's reported ordering (Figure 1a)
- Importance metric values are non-negative and every feature carries a rank position in the output CSV
- The RF used to derive importance matches the paper's configuration: 1000 binary classification trees with five-fold cross validation, trained on 15,000 ClinVar pathogenic variants

## Limitations

- Importance reflects the specific training set (15,000 ClinVar pathogenic variants) and 11-feature annotation scheme; scores are not transferable to models trained on different features or cohorts
- Mean decrease in Gini impurity can be biased toward high-cardinality or continuous features; the article does not report exact numeric importance values, only the qualitative top-feature ordering
- The eDiVA-Score is part of a pipeline in which eDiVA-Score requires eDiVA-Annotate input columns to be calculated on the fly, so the annotation table must be reproduced exactly to retrain and re-evaluate importance

## Evidence

- [methods] RF model with 1000 trees and five-fold cross validation using eleven features, trained on 15,000 ClinVar pathogenic variants: "training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees"
- [intro] eDiVA-Score is a random forest pathogenicity predictor: "eDiVA‐Score, which estimates variant pathogenicity using a random forest model"
- [abstract] eDiVA-Score is a machine learning predictor combining genomic and evolutionary signatures: "eDiVA features a machine learning-based variant pathogenicity predictor combining various genomic and evolutionary signatures."
- [readme] eDiVA-Annotate and eDiVA-Score are joined in the pipeline and rank variants: "This routine performs the eDiVA-Annotate part, adding annotations from several information sources for all variants in the VCF, and automatically ranks them with eDiVA-Score script."
