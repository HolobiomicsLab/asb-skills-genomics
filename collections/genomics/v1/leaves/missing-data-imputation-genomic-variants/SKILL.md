---
name: missing-data-imputation-genomic-variants
description: Use when constructing a variant pathogenicity training set (e.
when_to_use_negative:
- When the analysis goal is not variant classification/model training — e.g., routine
  filtering or prioritization of already-annotated variants where missing values can
  be handled by filtering (AF <= 0.01 rare-variant filter) instead of imputation
- When missing AF should genuinely be treated as unknown rather than as evidence of
  a novel variant (AF = 0) — this imputation semantics only holds for variants absent
  from population databases
- When a feature table is already complete and imputation would distort the neutral-variant
  expectation values
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: eDiVA
  role: Framework whose eDiVA-Annotate/eDiVA-DB modules compute the 11 variant features
    that require imputation before model training
  repo: https://github.com/mbosio85/ediva
- name: randomForest
  role: R package used to train the binary random forest classifier (1000 trees, five-fold
    cross validation) on the imputed feature table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:5ee288c589510184ddea63766e5f582acc1d5b50dcb58860a0ab209f3be6d583
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - eDiVA
  - randomForest
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# missing-data-imputation-genomic-variants

## Summary

Imputation strategy for building the eDiVA-Score training feature table: missing values among the 11 variant annotation features (maximum MAF, PhastCons/PhyloP conservation, Condel/CADD/Eigen/Mutation Assessor, segmental duplication likelihood, ABB-score) are replaced with the expected values for nonpathogenic (neutral) variants, except allele frequency where missing is coded as zero.

## When to use

Use when constructing a variant pathogenicity training set (e.g., for eDiVA-Score) from eDiVA-Annotate/eDiVA-DB annotation tables in which some of the 11 features are missing for ClinVar pathogenic or neutral/GnomAD control variants and a complete numeric feature matrix is required before training the random forest classifier.

## When NOT to use

- When the analysis goal is not variant classification/model training — e.g., routine filtering or prioritization of already-annotated variants where missing values can be handled by filtering (AF <= 0.01 rare-variant filter) instead of imputation
- When missing AF should genuinely be treated as unknown rather than as evidence of a novel variant (AF = 0) — this imputation semantics only holds for variants absent from population databases
- When a feature table is already complete and imputation would distort the neutral-variant expectation values

## Inputs

- ClinVar variant records annotated as 'pathogenic'/'likely pathogenic' (15,000 positives) and 'nonpathogenic' (15,000 negatives)
- GnomAD exome variants not present in ClinVar (100,000, majority AF < 1%) as additional controls
- eDiVA-Annotate/eDiVA-DB annotation columns: maximum MAF (1000 Genomes, GnomAD), PhastCons and PhyloP (primates and mammals), Condel, Phred-scaled CADD, Eigen, Mutation Assessor, segmental duplication likelihood, ABB-score sequencing-error estimator

## Outputs

- Complete (imputed) 11-feature training-set table with binary pathogenic/benign class labels
- Fitted randomForest model object (R) with 1000 classification trees and five-fold cross-validation results

## How to apply

Assemble the 11 training features for 15,000 ClinVar 'pathogenic'/'likely pathogenic' positive cases and 15,000 ClinVar 'nonpathogenic' variants plus 100,000 GnomAD exome variants absent from ClinVar as controls (majority rare, allele frequency < 1%, so allele frequency does not dominate the classifier). For any feature with a missing value, impute using the expected value estimated for nonpathogenic (neutral) variants — i.e., impute toward the neutral expectation rather than the dataset mean. The single exception is AF (maximum MAF of 1000 Genomes and GnomAD): a missing AF is imputed as zero, representing a novel variant absent from population databases. After imputation, train the random forest binary classifier with the R 'randomForest' package using 1000 binary classification trees and five-fold cross validation. This imputation choice is justified by keeping neutral-variant expectations for missing features while correctly encoding novel variants via AF = 0.

## Related tools

- **eDiVA** (Framework whose eDiVA-Annotate/eDiVA-DB modules compute the 11 variant features that require imputation before model training) — https://github.com/mbosio85/ediva
- **randomForest** (R package used to train the binary random forest classifier (1000 trees, five-fold cross validation) on the imputed feature table)

## Examples

```
nextflow run eDiVA-Annotate.nf --OUTF ./ --VCF testname.vcf --CPU 2
```

## Evaluation signals

- No missing (NA) values remain in the 11-column feature table after imputation
- Missing AF entries were coded as 0 (novel variant), not as the neutral expected AF — check the AF column distribution for excess zeros
- The fitted model object loads in R and produces pathogenicity probabilities strictly in [0,1]
- Five-fold cross-validated discrimination of held-out pathogenic vs benign variants is reported as ROC/AUC
- Class balance reflects the intended training design: ~15,000 positives vs 115,000 controls, with controls predominantly rare (AF < 1%)

## Limitations

- Imputation toward neutral (nonpathogenic) expected values assumes missingness is unbiased with respect to pathogenic class; features systematically missing in pathogenic variants may be biased
- AF = 0 imputation conflates 'absent from queried databases' with 'novel variant' — depends on the completeness of 1000 Genomes/GnomAD coverage
- eDiVA-Score requires eDiVA-Annotate output columns to be calculated on the fly, so imputation quality depends on the annotation pipeline running correctly
- CNV analysis is not yet available in eDiVA, so the classifier scope is limited to coding point-variant features

## Evidence

- [methods] imputing missing values with expected values for nonpathogenic (neutral) variants except AF, where missing means zero (novel variant): [evidence span withheld — anchor status: not_found]
- [methods] training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees: "training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees (Breiman, 2001; Hastie, Tibshirani, & Friedman, 2009) and five‐fold cross validation"
- [readme] This routine performs the eDiVA-Annotate part, adding annotations from several information sources for all variants in the VCF, and automatically ranks them: "This routine performs the eDiVA-Annotate part, adding annotations from several information sources for all variants in the VCF, and automatically ranks them with eDiVA-Score script."
