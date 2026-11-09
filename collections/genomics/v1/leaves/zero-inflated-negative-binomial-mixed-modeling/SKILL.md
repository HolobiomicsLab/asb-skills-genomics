---
name: zero-inflated-negative-binomial-mixed-modeling
description: Use when analyzing raw (unnormalized) scRNA-seq count matrices with excess
  zeros and overdispersion, especially when cells are clustered within samples and
  covariate association testing (e.g., disease status) must account for within-sample
  correlation.
when_to_use_negative:
- Input is log-transformed or otherwise normalized data — TWO-SIGMA models observed
  counts directly and performs no normalization
- Only a two-group comparison with no repeated measures, covariates, or within-sample
  correlation is needed — simpler DE tools such as SCDE, scDD, or MAST may suffice
- Gene set testing where some genes get random effects and others do not — the ad
  hoc procedure is explicitly not recommended for twosigmag because a common gene-level
  null is required
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: twosigma
  role: R package implementing the two-component ZINB mixed model; workhorse functions
    twosigma, twosigma_custom, lr.twosigma, adhoc.twosigma, test.vc.twosigma
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Underlying engine used to fit the zero-inflated negative binomial model with
    random effects specified in equation (2)
- name: R
  role: Runtime environment; package requires R >= 3.6.3
  repo: https://cran.r-project.org/web/packages/twosigma/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:16a1be091b5171af8f8ced0a2e74732ccb151072ea3328049d8b8b77c1f6cdd4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edvanburen/twosigma
  tools:
  - R
  - glmmTMB
  - twosigma
  - twosigma (R package)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# zero-inflated-negative-binomial-mixed-modeling

## Summary

Fit the TWO-SIGMA zero-inflated negative binomial mixed-effects model to raw scRNA-seq counts, modeling drop-out via mixed-effects logistic regression and conditional mean via mixed-effects log-linear negative binomial regression using glmmTMB.

## When to use

Use when analyzing raw (unnormalized) scRNA-seq count matrices with excess zeros and overdispersion, especially when cells are clustered within samples and covariate association testing (e.g., disease status) must account for within-sample correlation.

## When NOT to use

- Input is log-transformed or otherwise normalized data — TWO-SIGMA models observed counts directly and performs no normalization
- Only a two-group comparison with no repeated measures, covariates, or within-sample correlation is needed — simpler DE tools such as SCDE, scDD, or MAST may suffice
- Gene set testing where some genes get random effects and others do not — the ad hoc procedure is explicitly not recommended for twosigmag because a common gene-level null is required

## Inputs

- Raw non-negative integer scRNA-seq count matrix (genes x cells), no normalization applied, no missing values
- mean_covar: covariate matrix for the conditional mean model without an intercept (e.g., from model.matrix)
- zi_covar: covariate matrix for the zero-inflation model without an intercept
- id: vector of individual-level IDs with length equal to the total number of cells
- Optional: disp_covar, custom mean_form/zi_form formulas (response must be named 'count'), lr.df for LRT degrees of freedom

## Outputs

- Per-gene fitted zero-inflated negative binomial mixed models (glmmTMB objects or failure strings)
- Gene-level Z-statistics, logFC estimates, and p-values extracted from the cond coefficient table
- Ad hoc one-way ANOVA F-test p-values on Pearson residuals for random-effect screening
- Likelihood ratio test results for fixed effects (joint null, e.g., H0: alpha1 = beta1 = 0) and variance component tests via test.vc.twosigma

## How to apply

Call the twosigma R function on a raw integer count matrix, supplying mean_covar and zi_covar design matrices WITHOUT intercept terms and a vector of individual-level IDs (id is required even if random effects are not used). By default the ad hoc procedure decides whether random intercepts are needed: Pearson's residuals from a zero-inflated NB model without random effects are regressed on the sample indicator via one-way ANOVA, and the ANOVA F-test p-value screens genes needing random effects. Set adhoc=FALSE to manually control mean_re and zi_re, or use twosigma_custom with lme4-style formulas such as count ~ mean_covar_matrix + (1|id) for the mean model and ~ zi_covar_matrix + (1|id) for the zero-inflation model. The model is fit with glmmTMB; when random effects are ignored in favor of fixed-only models, standard errors are underestimated and type-I errors inflate, which is the core rationale for the mixed-effects structure. Multiple cores via ncores are recommended across many genes.

## Related tools

- **twosigma** (R package implementing the two-component ZINB mixed model; workhorse functions twosigma, twosigma_custom, lr.twosigma, adhoc.twosigma, test.vc.twosigma) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Underlying engine used to fit the zero-inflated negative binomial model with random effects specified in equation (2))
- **R** (Runtime environment; package requires R >= 3.6.3) — https://cran.r-project.org/web/packages/twosigma/

## Examples

```
fits<-twosigma(count_matrix, mean_covar=mean_covar_matrix, zi_covar=zi_covar_matrix, mean_re=TRUE, zi_re=TRUE, id=id, adhoc=F)
```

## Evaluation signals

- Model fit objects are glmmTMB class or a character string indicating failure; genes returning character strings yield NA statistics and must be excluded
- Ignoring random effects should reproduce the failure mode of underestimated standard errors and inflated type-I errors when within-sample correlation is present
- Type-I error and power evaluation should use the likelihood ratio test on the joint null H0: alpha1 = beta1 = 0
- Included random intercepts in both components (mean_re=TRUE, zi_re=TRUE) with adhoc=FALSE should reproduce the default TWO-SIGMA specification identically to twosigma_custom with (1|id) terms
- Minimum dependency versions must hold: multcomp >= 1.4-13, pscl >= 1.5.5, pbapply >= 1.4.0, doParallel >= 1.0.15, R >= 3.6.3

## Limitations

- The ad hoc ANOVA-based screen assumes genes can be individually assigned random effects, which breaks the common gene-level null required for gene set testing
- No normalization is performed; batch must be controlled for by inclusion in the design matrices
- With adhoc=TRUE, mean_re and zi_re are ignored and a warning is printed
- lr.twosigma assumes the tested variable is in both model components; custom LRTs require the user to supply valid formulas and lr.df
- The paper defers detailed robustness discussion of the ZINB mixed model to the supplement (section 6, supplementary figure S10)

## Evidence
