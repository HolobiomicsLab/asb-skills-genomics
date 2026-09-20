---
name: zero-inflated-negative-binomial-regression
description: Use when raw integer scRNA-seq count data exhibit excess zeros and overdispersion,
  cells are nested within samples (within-sample correlation), and inference is needed
  on covariates such as a binary disease status beyond a two-group comparison.
when_to_use_negative:
- Input is log-normalized or otherwise transformed expression (e.g. MAST-style log-scale
  input) — twosigma expects raw integer counts and performs no normalization itself
- The data are bulk RNA-seq or otherwise lack within-sample cell correlation — bulk
  pipelines like DESeq2 or edgeR are the appropriate alternative, and the random-effect
  machinery buys nothing
- The goal is competitive gene set testing where only some genes get random effects
  — the ad hoc procedure is explicitly not recommended there because the common gene-level
  null hypothesis is then unclear
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3053
tools:
- name: twosigma
  role: R package whose workhorse functions twosigma and twosigma_custom fit the two-component
    zero-inflated NB mixed model per gene via glmmTMB, with ad hoc random-effect selection
    and variance-component LRT helpers
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Underlying fitting engine employed by twosigma to estimate the zero-inflated
    negative binomial model with random intercepts in both the conditional mean (log
    link) and zero-inflation (logit link) components
- name: R
  role: Runtime environment; package version 1.0.2 requires R >= 3.6.3 and minimum
    versions of multcomp (>= 1.4-13), pscl (>= 1.5.5), pbapply (>= 1.4.0), doParallel
    (>= 1.0.15)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:5ae3de65eb945f7e2adb03bf455251f30c4c6adc786ab194f2ba7522c583d928
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edvanburen/twosigma
  tools:
  - R
  - glmmTMB
  - twosigma
  - twosigma (R package)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# zero-inflated-negative-binomial-regression

## Summary

Fit the TWO-SIGMA two-component zero-inflated negative binomial mixed model to raw scRNA-seq counts per gene: a mixed-effects logistic regression for drop-out probability and a mixed-effects log-linear negative binomial regression for the conditional mean, both fitted with glmmTMB and admitting sample-level random intercepts.

## When to use

Use when raw integer scRNA-seq count data exhibit excess zeros and overdispersion, cells are nested within samples (within-sample correlation), and inference is needed on covariates such as a binary disease status beyond a two-group comparison.

## When NOT to use

- Input is log-normalized or otherwise transformed expression (e.g. MAST-style log-scale input) — twosigma expects raw integer counts and performs no normalization itself
- The data are bulk RNA-seq or otherwise lack within-sample cell correlation — bulk pipelines like DESeq2 or edgeR are the appropriate alternative, and the random-effect machinery buys nothing
- The goal is competitive gene set testing where only some genes get random effects — the ad hoc procedure is explicitly not recommended there because the common gene-level null hypothesis is then unclear

## Inputs

- Raw non-negative integer scRNA-seq count matrix (no normalization, no missing values), one row/cell per gene column
- mean_covar: design matrix (e.g. from model.matrix) of covariates for the conditional mean model, without an intercept column
- zi_covar: design matrix of covariates for the zero-inflation (logit drop-out) model, without an intercept column
- id: vector of individual-level sample IDs, one entry per cell (required even if random effects are not used)
- Optional: custom two-sided mean formula and one-sided zi formula (lme4 syntax) for twosigma_custom; disp_covar and covar_to_test for lr.twosigma

## Outputs

- Per-gene glmmTMB fit objects from twosigma/twosigma_custom with fixed-effect estimates, SEs, z values, and p-values for each model component
- Component-wise Z statistics (Z_mean, Z_ZI) and their Stouffer combination Z_combined = (Z_mean + Z_ZI)/sqrt(2) with two-sided standard-normal p-values (e.g. a table like stouffer_z_table.csv)
- Likelihood-ratio test p-values for joint fixed-effect hypotheses (e.g. H0: alpha1 = beta1 = 0) from lr.twosigma / lr.twosigma_custom
- ANOVA F-test p-values from adhoc.twosigma screening genes for needed random effects; variance-component LRT results from test.vc.twosigma

## How to apply

For each gene, pass the raw count vector with no normalization to twosigma (or twosigma_custom with mean_form=count~...+(1|id) and a one-sided zi_form), supplying mean_covar and zi_covar design matrices WITHOUT intercepts and a vector of individual-level id values. By default the ad hoc procedure runs: a zero-inflated NB model without random effects is fitted, and a one-way ANOVA of Pearson's residuals on the sample indicator decides whether random intercepts are needed in each component; alternatively set adhoc=FALSE and force mean_re=TRUE, zi_re=TRUE. Each component's fixed-effect estimate and standard error (e.g. for disease status, age, and cellular detection rate/CDR covariates) yields component-wise Z statistics Z_mean = beta1_hat/SE and Z_ZI = alpha1_hat/SE, which can be combined via Stouffer as (Z_mean + Z_ZI)/sqrt(2) or tested jointly via the likelihood-ratio test of H0: alpha1 = beta1 = 0 using lr.twosigma. Rationale: fitting observed counts directly (rather than MAST's hurdle model on the log scale) and including sample-level random effects preserves type-I error when cells from the same sample are even moderately correlated; ignoring them underestimates standard errors and inflates type-I error.

## Related tools

- **twosigma** (R package whose workhorse functions twosigma and twosigma_custom fit the two-component zero-inflated NB mixed model per gene via glmmTMB, with ad hoc random-effect selection and variance-component LRT helpers) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Underlying fitting engine employed by twosigma to estimate the zero-inflated negative binomial model with random intercepts in both the conditional mean (log link) and zero-inflation (logit link) components)
- **R** (Runtime environment; package version 1.0.2 requires R >= 3.6.3 and minimum versions of multcomp (>= 1.4-13), pscl (>= 1.5.5), pbapply (>= 1.4.0), doParallel (>= 1.0.15))

## Examples

```
fits<-twosigma(count_matrix, mean_covar=mean_covar_matrix, zi_covar=zi_covar_matrix, mean_re=TRUE, zi_re=TRUE, id=id, adhoc=F)
```

## Evaluation signals

- Fits reproduce the default TWO-SIGMA specification: random intercepts (1|id) in BOTH components, mean_covar and zi_covar matrices without intercept columns, and response named 'count' in custom formulas
- Type-I error for the joint test H0: alpha1 = beta1 = 0 holds at the nominal 0.05 rate (Bonferroni 1/10000 as a stricter check) when within-sample correlation is present
- Omitting the random effects should visibly inflate type-I error / underestimate SEs relative to the full mixed model (as shown in the paper's table 3)
- The ad hoc ANOVA F-test p-value from Pearson's residuals-on-sample regression flags genes whose residuals differ across individuals, matching genes that benefit from random intercepts
- Stouffer combination equals (Z_mean + Z_ZI)/sqrt(2) exactly for every gene, and genes returning NA correspond to failed/non-glmmTMB fits in the lapply extraction pattern

## Limitations

- The ad hoc procedure must not be used for gene set testing: when some genes get random effects and others do not, the common gene-level null hypothesis required by twosigmag is not clearly met
- twosigma_custom uses user-supplied formulas directly; the user is responsible that they define a valid likelihood ratio test and must supply lr.df for lr.twosigma_custom
- lr.twosigma assumes the tested covariate appears in BOTH model components and that the zero-inflation component contains more than an intercept; integer column positions are not checked to match the same covariate across components
- Ignoring random effects can lead to underestimated standard errors and inflated type-I errors; incorrectly failing to include them had dramatic impacts in the pancreatic islet analysis
- Contrast-matrix testing at the gene level is still ongoing functionality; the README suggests using twosigmag with index_test and all_as_ref=TRUE as a workaround

## Evidence
