---
name: genomic-annotation-feature-engineering
description: Use when building or retraining a variant pathogenicity predictor that
  needs annotated per-variant features (allele frequency, conservation, deleteriousness
  scores) extracted from VCF-level variant calls.
when_to_use_negative:
- Input is already a feature-annotated table (e.g., combined.variants.supplement.ranked.csv
  from eDiVA-Annotate) — proceed directly to scoring/prioritization
- Goal is family-based causal variant filtering or prioritization by inheritance model
  and HPO phenotypes — that is eDiVA-Prioritize, not feature engineering
- Analysis targets CNVs — CNV analysis is currently being integrated in eDiVA and
  is not yet available
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3275
tools:
- name: eDiVA
  role: Annotation framework whose eDiVA-Annotate/eDiVA-DB modules supply the columns
    from which the 11 classifier features are computed on the fly
  repo: https://github.com/mbosio85/ediva
- name: randomForest
  role: R package used to train the binary classification random forest (1000 trees,
    five-fold cross validation) on the engineered features
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:4fb820346c9b374aad1c2583e0de41464431e33a37c8a233c5c2b67286f49362
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - eDiVA
  - randomForest
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3275
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# genomic-annotation-feature-engineering

## Summary

Computes the 11 annotation-derived features that feed eDiVA-Score's random forest pathogenicity classifier, combining population allele frequency, conservation, functional-prediction, segmental duplication, and sequencing-error signals from eDiVA-Annotate/eDiVA-DB.

## When to use

Use when building or retraining a variant pathogenicity predictor that needs annotated per-variant features (allele frequency, conservation, deleteriousness scores) extracted from VCF-level variant calls.

## When NOT to use

- Input is already a feature-annotated table (e.g., combined.variants.supplement.ranked.csv from eDiVA-Annotate) — proceed directly to scoring/prioritization
- Goal is family-based causal variant filtering or prioritization by inheritance model and HPO phenotypes — that is eDiVA-Prioritize, not feature engineering
- Analysis targets CNVs — CNV analysis is currently being integrated in eDiVA and is not yet available

## Inputs

- Multisample VCF of called variants (combined.variants.supplement.vcf)
- eDiVA-Annotate output columns (population AF from 1000GP, GnomAD, dbSNP, discovEHR; ClinVar/OMIM clinical data from eDiVA-DB)
- Pre-computed functional scores: Condel, CADD (Phred-scaled), Eigen, Mutation Assessor, PhastCons/PhyloP, segmental duplication and ABB-score tracks

## Outputs

- 11-column per-variant feature table with imputed missing values
- Fitted random forest model object (R randomForest, 1000 trees)
- Per-variant pathogenicity probability in [0,1] used to rank variants

## How to apply

Annotate each called variant via eDiVA-Annotate/eDiVA-DB, then compute 11 features: maximum MAF across 1000 Genomes and GnomAD; four conservation measures (PhastCons and PhyloP in primates and mammals); four functional predictors (Condel, Phred-scaled CADD, Eigen, Mutation Assessor); the segmental duplication likelihood; and the ABB-score sequencing-error estimator. Impute missing feature values with expected values for nonpathogenic (neutral) variants, except allele frequency where a missing value means zero (i.e., a novel variant absent from population databases). These features are calculated on the fly from eDiVA-Annotate output columns and fed to a random forest binary classifier trained with the R 'randomForest' package using 1000 classification trees and five-fold cross validation, producing a pathogenicity probability in [0,1] for each variant.

## Related tools

- **eDiVA** (Annotation framework whose eDiVA-Annotate/eDiVA-DB modules supply the columns from which the 11 classifier features are computed on the fly) — https://github.com/mbosio85/ediva
- **randomForest** (R package used to train the binary classification random forest (1000 trees, five-fold cross validation) on the engineered features)

## Examples

```
nextflow run eDiVA-Annotate.nf --OUTF ./ --VCF testname.vcf --CPU 2
```

## Evaluation signals

- All 11 features are populated for every variant, with missing AF imputed as 0 (novel variant) and other missing values imputed with expected values for neutral variants
- Fitted model object loads and produces pathogenicity probabilities bounded in [0,1]
- Five-fold cross-validated discrimination of held-out pathogenic vs benign variants is reported as ROC/AUC
- Training set composition matches the described design: 15,000 ClinVar pathogenic/'likely pathogenic' positives, with controls keeping the majority rare (AF < 1%) so allele frequency does not dominate the classifier
- Output CSV (combined.variants.supplement.ranked.csv) contains the annotation columns plus the eDiVA-Score column

## Limitations

- Control set relies on GnomAD exome variants not contained in ClinVar plus nonpathogenic ClinVar calls, so allele-frequency balance (majority rare, AF < 1%) must be maintained to prevent frequency from dominating the classifier
- eDiVA-Score is exonic/coding-variant oriented; the annotation stack does not yet cover CNV analysis
- Feature scores depend on external databases (ClinVar, OMIM, dbSNP, 1000GP, GnomAD, discovEHR) whose incompleteness can reduce classification and downstream recall

## Evidence

- [methods] training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees: "training a random forest (RF) model using the R “randomForest” package with 1000 binary classification trees"
- [readme] eDiVA-Score requires eDiVA-Annotate columns on the fly: "The two steps are joined because eDiVA-Score requires eDiVA-Annotate input columns to be calculated on the fly."
- [intro] eDiVA-Score estimates pathogenicity via random forest: "(c) eDiVA‐Score, which estimates variant pathogenicity using a random forest model"
- [methods] Annotation sources include dbSNP and population frequencies: "Each variant is annotated with population allele frequency information from the dbSNP (Sherry et al., 2001)"
- [methods] eDiVA-DB provides ClinVar/OMIM clinical data per variant: "eDiVA‐DB provides clinical data from ClinVar (Landrum et al., 2014) and OMIM related to each variant and affected gene"
