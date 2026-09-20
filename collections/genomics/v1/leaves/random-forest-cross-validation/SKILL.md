---
name: random-forest-cross-validation
description: Use when a variant prioritization pipeline needs an ex-ante pathogenicity
  probability per coding variant computed only from genomic and evolutionary annotations
  (allele frequency, conservation, deleteriousness predictors), before any phenotype
  or inheritance information is used.
when_to_use_negative:
- Variant annotations (allele frequency, conservation, CADD/Eigen/Mutation Assessor
  columns) have not yet been computed — eDiVA-Score requires eDiVA-Annotate input
  columns to be calculated first
- The goal is prioritization using disease phenotype (HPO IDs) or inheritance-model
  segregation — that belongs to eDiVA-Prioritize, not the pathogenicity score
- Variants are structural/CNV calls — CNV analysis is not yet available in eDiVA
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0203
tools:
- name: eDiVA
  role: Framework whose eDiVA-Score module implements the random forest pathogenicity
    estimator, joined to eDiVA-Annotate
  repo: https://github.com/mbosio85/ediva
- name: randomForest
  role: R package used to train the binary classifier with 1000 classification trees
    and five-fold cross validation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:7515990514bd92315ffeabbd6a7b033443a8d540bf62fc9e740c6ebdb65df857
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - eDiVA
  - randomForest
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# random-forest-cross-validation

## Summary

Train and apply eDiVA-Score, a random forest variant pathogenicity classifier built with the R 'randomForest' package (1000 binary classification trees, five-fold cross validation) on 11 genomic/evolutionary features, to estimate the probability that a variant is pathogenic independently of prior clinical information.

## When to use

Use when a variant prioritization pipeline needs an ex-ante pathogenicity probability per coding variant computed only from genomic and evolutionary annotations (allele frequency, conservation, deleteriousness predictors), before any phenotype or inheritance information is used.

## When NOT to use

- Variant annotations (allele frequency, conservation, CADD/Eigen/Mutation Assessor columns) have not yet been computed — eDiVA-Score requires eDiVA-Annotate input columns to be calculated first
- The goal is prioritization using disease phenotype (HPO IDs) or inheritance-model segregation — that belongs to eDiVA-Prioritize, not the pathogenicity score
- Variants are structural/CNV calls — CNV analysis is not yet available in eDiVA

## Inputs

- VCF of called variants (e.g. combined.variants.supplement.vcf from eDiVA-Predict)
- eDiVA-Annotate annotation columns (max MAF of 1000GP/GnomAD, PhastCons/PhyloP primate and mammal, Condel, CADD Phred, Eigen, Mutation Assessor, segmental duplication likelihood, ABB score)
- ClinVar clinical variant classifications (pathogenic, likely pathogenic, nonpathogenic) for the training set
- GnomAD exome variants not present in ClinVar for the benign control set

## Outputs

- Fitted random forest model object (R 'randomForest')
- Curated training-set feature table (30,000 positive / 115,000 control variants x 11 features)
- Per-variant pathogenicity probability in [0,1] used to rank candidates (e.g. combined.variants.supplement.ranked.csv)

## How to apply

Extract 15,000 random 'pathogenic'/'likely pathogenic' ClinVar variants as positive training cases and build the control set from 15,000 'nonpathogenic' ClinVar variants plus 100,000 GnomAD exome variants absent from ClinVar, keeping the majority rare (allele frequency < 1%) so allele frequency does not dominate the classifier. Compute the 11 training features from eDiVA-Annotate/eDiVA-DB annotations: maximum MAF of 1000 Genomes and GnomAD, PhastCons and PhyloP in primates and mammals, Condel, Phred-scaled CADD, Eigen and Mutation Assessor, segmental duplication likelihood, and the ABB-score sequencing-error estimator; impute missing values with expected values for nonpathogenic (neutral) variants except AF, where missing means zero (novel variant). Train the random forest with the R 'randomForest' package using 1000 binary classification trees and five-fold cross validation. Save the fitted model object and the curated training-set feature table, then apply the model to annotate and rank variants (in eDiVA this step is joined to eDiVA-Annotate because the input columns are calculated on the fly).

## Related tools

- **eDiVA** (Framework whose eDiVA-Score module implements the random forest pathogenicity estimator, joined to eDiVA-Annotate) — https://github.com/mbosio85/ediva
- **randomForest** (R package used to train the binary classifier with 1000 classification trees and five-fold cross validation)

## Examples

```
nextflow run eDiVA-Annotate.nf --OUTF ./ --VCF testname.vcf --CPU 2
```

## Evaluation signals

- The loaded model object produces pathogenicity probabilities strictly within [0,1] for held-out variants
- Five-fold cross-validated discrimination of held-out pathogenic vs benign variants is reported as ROC/AUC
- Training set composition matches specification: 15,000 ClinVar pathogenic/likely pathogenic positives vs 15,000 nonpathogenic ClinVar plus 100,000 GnomAD controls, majority with AF < 1%
- Feature table has exactly the 11 described features with missing values imputed (AF missing treated as 0, others as expected neutral values)
- In the pipeline, ranking output (combined.variants.supplement.ranked.csv) exists only after eDiVA-Annotate columns were computed on the fly

## Limitations

- Missing feature values must be imputed with expected values for nonpathogenic (neutral) variants; AF missingness is treated as zero (novel variant), which embeds assumptions about unseen variants
- The benign control set relies partly on GnomAD variants absent from ClinVar, which may include unobserved pathogenic variants
- The score is deliberately kept independent of prior clinical information; phenotype/inheritance refinement happens later in eDiVA-Prioritize

## Evidence

- [methods] training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees and five‐fold cross validation: "training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees (Breiman, 2001; Hastie, Tibshirani, & Friedman, 2009) and five‐fold cross validation"
- [intro] eDiVA-Score: estimates variant pathogenicity using a random forest model: "(c) eDiVA‐Score, which estimates variant pathogenicity using a random forest model"
- [readme] The two steps are joined because eDiVA-Score requires eDiVA-Annotate input columns to be calculated on the fly.: "The two steps are joined because eDiVA-Score requires eDiVA-Annotate input columns to be calculated on the fly."
- [readme] There is an available repository of pre-calculated eDiVA-Scores for all exonic variants: "There is an available repository of pre-calculated eDiVA-Scores for all exonic variants [here](https://public_docs.crg.es/sossowski/MicrobeGenomes/human/eDiVA/eDiVA_score/)"
