---
name: conservation-score-feature-analysis
description: Use when interpreting, reproducing, or re-ranking feature importance
  of the eDiVA-Score random forest variant pathogenicity model, especially to quantify
  how conservation scores (PhastCons/PhyloP, primates vs mammals) compare with allele-frequency
  and functional-impact predictors.
when_to_use_negative:
- You only have raw sequencing reads and have not yet run eDiVA-Annotate to compute
  the 11 annotation features on the fly.
- You need variant calling or filtering by inheritance mode — that is the eDiVA-Predict/eDiVA-Prioritize
  scope, not model feature-importance analysis.
- The model you are analyzing is not a random forest classifier with the 11-feature
  eDiVA annotation schema (feature importances are not transferable across model families).
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0199
tools:
- name: eDiVA
  role: Pipeline whose eDiVA-Annotate module generates the conservation and functional
    annotations required by eDiVA-Score, and which runs the random forest ranking
    automatically.
  repo: https://github.com/mbosio85/ediva
- name: randomForest
  role: R package used to train the eDiVA-Score classifier (1000 binary classification
    trees, five-fold cross validation) and to compute per-feature variable importance.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:8f5cba143cc437bcf83f0dd09a65b2b28538cb425b62784a28331a24917c04ba
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
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# conservation-score-feature-analysis

## Summary

Analyzes the contribution of evolutionary conservation features (PhastCons/PhyloP measures for primates and mammals) within the eDiVA-Score random forest pathogenicity predictor, which combines genomic and evolutionary signatures across 11 annotation features.

## When to use

Use when interpreting, reproducing, or re-ranking feature importance of the eDiVA-Score random forest variant pathogenicity model, especially to quantify how conservation scores (PhastCons/PhyloP, primates vs mammals) compare with allele-frequency and functional-impact predictors.

## When NOT to use

- You only have raw sequencing reads and have not yet run eDiVA-Annotate to compute the 11 annotation features on the fly.
- You need variant calling or filtering by inheritance mode — that is the eDiVA-Predict/eDiVA-Prioritize scope, not model feature-importance analysis.
- The model you are analyzing is not a random forest classifier with the 11-feature eDiVA annotation schema (feature importances are not transferable across model families).

## Inputs

- Trained eDiVA-Score random forest model (R randomForest object, 1000 trees, five-fold CV)
- 11-feature variant annotation table (MAF, PhastCons/PhyloP conservation, Condel, CADD, Eigen, Mutation Assessor, segmental duplication, ABB-score)
- Variant VCF annotated by eDiVA-Annotate (combined.variants.supplement.vcf)

## Outputs

- Per-feature variable-importance metrics (mean decrease in Gini impurity / accuracy)
- Ranked feature-importance table (e.g. variable_importance.csv) with feature names, metrics, and rank positions
- Comparison of integrated combination scores vs specialized predictors vs conservation features

## How to apply

Load the trained eDiVA-Score random forest model — built with the R 'randomForest' package using 1000 binary classification trees and five-fold cross validation — together with its 11-feature annotation table (maximum MAF from 1000 Genomes/GnomAD/dbSNP, four PhastCons/PhyloP conservation measures for primates and mammals, four functional-impact predictors Condel/CADD/Eigen/Mutation Assessor, segmental duplication likelihood, and the ABB-score sequencing-error estimator). Compute per-feature variable importance via randomForest::importance (mean decrease in Gini impurity and/or mean decrease in accuracy), rank the features, and compare integrated combination scores (Condel, Eigen, CADD) against specialized predictors and the conservation/allele-frequency features. The paper reports that population allele frequency, functional impact, and conservation in placental mammals rank as the most important features (Figure 1a), so a correct analysis must recover that ordering.

## Related tools

- **eDiVA** (Pipeline whose eDiVA-Annotate module generates the conservation and functional annotations required by eDiVA-Score, and which runs the random forest ranking automatically.) — https://github.com/mbosio85/ediva
- **randomForest** (R package used to train the eDiVA-Score classifier (1000 binary classification trees, five-fold cross validation) and to compute per-feature variable importance.)

## Examples

```
nextflow run eDiVA-Annotate.nf --OUTF ./ --VCF testname.vcf --CPU 2
```

## Evaluation signals

- The ranked importance table places maximum MAF, functional-impact predictors (Condel, CADD, Eigen, Mutation Assessor), and mammalian conservation measures among the top features, matching the paper's reported ordering (Figure 1a).
- The feature set analyzed contains exactly 11 features matching the eDiVA annotation schema (one MAF measure, four conservation measures, four functional-impact scores, segmental duplication, ABB-score).
- Importance values are computed on the trained 1000-tree random forest, not on a refitted or subsampled model, and both Gini and accuracy-based metrics are reported where possible.
- Feature names and values are traceable to columns in the eDiVA-Annotate output CSV (combined.variants.supplement.ranked.csv).

## Limitations

- Feature importance was derived from a semisynthetic benchmark setup (known disease variants introduced into real WES trio data) rather than fully real disease cohorts.
- Pre-calculated eDiVA-Scores exist only for exonic variants; non-exonic variants lack coverage in the pre-computed repository.
- CNV analysis is not part of the scoring model; the model is limited to coding/point-variant features.

## Evidence

- [abstract] eDiVA features a machine learning-based variant pathogenicity predictor combining various genomic and evolutionary signatures.: "eDiVA features a machine learning-based variant pathogenicity predictor combining various genomic and evolutionary signatures."
- [methods] RF model trained with randomForest, 1000 trees, 5-fold CV: "training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees (Breiman, 2001; Hastie, Tibshirani, & Friedman, 2009) and five‐fold cross validation"
- [methods] Filtering on functional and population genomic features: "filtering based on functional and population genomic features"
- [readme] Annotate module computes annotations needed by the scoring step: "This routine performs the eDiVA-Annotate part, adding annotations from several information sources for all variants in the VCF, and automatically ranks them with eDiVA-Score script."
- [readme] Pre-calculated scores cover only exonic variants: "There is an available repository of pre-calculated eDiVA-Scores for all exonic variants"
