---
name: cox-proportional-hazards-modeling
description: Use when you have per-sample mutation status merged with survival/time-to-event
  group annotations and need to test whether a mutation is an independent prognostic
  factor after adjusting for other covariates.
when_to_use_negative:
- Input is only a mutational signature abundance matrix (P/S matrices) without any
  survival outcome or group labels — use RNMF's samPerMutSigs/genePerMutSigs association
  tests instead
- Goal is de novo extraction of SBS/DBS/ID mutational signatures from VCF/MAF data
  — that is the RNMF denovoNMF workflow, not survival modeling
- Clinical data lack survival or time-to-event information; the article notes treatment
  methods, medication information and extended follow-up time are incomplete, so Cox
  modeling is not defensible without a survival outcome
tools:
- name: R survival package
  role: Fits the multivariate Cox proportional hazards model with R248Q status and
    survival group (G2/G4) as covariates
- name: RNMF
  role: Supplies the example MAF file (maffile) and Groupfile (Gfile) whose per-sample
    mutation status and group annotations form the Cox covariate table
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Implementation language for loading the MAF/Groupfile, building the covariate
    table, and running the Cox model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:11b3bb64edd819e59a2567707b46217937b7f24d6b17c4476022c69dfc62625f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - R
  - R survival package
  - RNMF
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# cox-proportional-hazards-modeling

## Summary

Fit a multivariate Cox proportional hazards model in R to test whether a molecular feature (e.g. TP53 hotspot R248Q status from an RNMF-style MAF file) remains an independent prognosticator for survival when modeled alongside survival group labels.

## When to use

Use when you have per-sample mutation status merged with survival/time-to-event group annotations and need to test whether a mutation is an independent prognostic factor after adjusting for other covariates.

## When NOT to use

- Input is only a mutational signature abundance matrix (P/S matrices) without any survival outcome or group labels — use RNMF's samPerMutSigs/genePerMutSigs association tests instead
- Goal is de novo extraction of SBS/DBS/ID mutational signatures from VCF/MAF data — that is the RNMF denovoNMF workflow, not survival modeling
- Clinical data lack survival or time-to-event information; the article notes treatment methods, medication information and extended follow-up time are incomplete, so Cox modeling is not defensible without a survival outcome

## Inputs

- MAF-format somatic mutation file (maffile) with columns Tumor_Sample_Barcode, Hugo_Symbol (e.g. Regular.ESCC.exon.WES.maf-input-nonsilent.txt from RNMF)
- Groupfile (Gfile) mapping samples to survival group labels (G2/G4)
- Per-sample hotspot mutation status vector (TP53 R248Q indicator)

## Outputs

- cox_regression_results.csv containing regression coefficients, hazard ratios with confidence intervals, and p-values for each covariate
- Inference on whether TP53 R248Q is an independent prognosticator for poor survival in ESCC (HR > 1, p < 0.05)

## How to apply

Load the example MAF file (maffile, e.g. 'Regular.ESCC.exon.WES.maf-input-nonsilent.txt') and the Groupfile (Gfile, 'Groupfile.txt') from the RNMF package, keyed on sample.id = 'Tumor_Sample_Barcode' and Hugo = 'Hugo_Symbol', and extract per-sample TP53 hotspot mutation status including R248Q. Merge the binary R248Q mutation indicator with the survival group labels (G2/G4) into a single per-sample covariate table. Using the R survival package (2.40-1), fit a multivariate coxph model with R248Q status and group annotation as covariates, then extract coefficients, hazard ratios with confidence intervals, and p-values per covariate into cox_regression_results.csv. Validate by confirming the R248Q hazard ratio indicates poorer prognosis (HR > 1) with a p-value below the 0.05 significance threshold, establishing independence from the group covariate.

## Related tools

- **R survival package** (Fits the multivariate Cox proportional hazards model with R248Q status and survival group (G2/G4) as covariates)
- **RNMF** (Supplies the example MAF file (maffile) and Groupfile (Gfile) whose per-sample mutation status and group annotations form the Cox covariate table) — https://github.com/zhenzhang-li/RNMF
- **R** (Implementation language for loading the MAF/Groupfile, building the covariate table, and running the Cox model)

## Examples

```
library(survival); fit <- coxph(Surv(time, status) ~ R248Q + group, data = cov_table); write.csv(summary(fit)$coefficients, 'cox_regression_results.csv')
```

## Evaluation signals

- The per-sample covariate table is keyed on sample.id = 'Tumor_Sample_Barcode' and Hugo = 'Hugo_Symbol' as in the RNMF example files
- The R248Q covariate has hazard ratio HR > 1, indicating poorer prognosis
- The R248Q p-value is below the 0.05 significance threshold while adjusted alongside the group (G2/G4) covariate
- cox_regression_results.csv contains coefficient, HR, confidence interval, and p-value columns for every covariate in the model

## Limitations

- Complete clinical information (treatment methods, medication information, extended follow-up time) is incomplete in the underlying ESCC cohort, limiting survival-model refinement
- The relatively small number of individuals in the 1073-sample ESCC cohort is a shortcoming for model learning
- Only coding regions were uniformly analyzed due to inconsistent sequencing background, which may affect mutation-status extraction feeding the model

## Evidence

- [abstract] Survival outcome linkage of gene-level features in the cohort: "the CCA matrix image data of genes in the signatures New, SBS3*, and SBS17b* were helpful for the preliminary evaluation of shortened survival outcome"
- [methods] MAF as supported mutation input format: "somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file"
- [readme] Example MAF and Groupfile inputs from the RNMF README: "maffile = 'Regular.ESCC.exon.WES.maf-input-nonsilent.txt'

Groupfile = 'Groupfile.txt'"
- [discussion] Missing clinical covariates limit survival analyses: [evidence span withheld — anchor status: not_found]
