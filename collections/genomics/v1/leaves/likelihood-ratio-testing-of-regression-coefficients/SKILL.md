---
name: likelihood-ratio-testing-of-regression-coefficients
description: Use when a covariate (e.g. t2d_sim) is entered in BOTH the mean_covar
  and zi_covar design matrices of a TWO-SIGMA ZINB mixed-model fit and you need per-gene
  LR statistics and p-values for its joint association with expression.
when_to_use_negative:
- The tested covariate is in only one component, or the zero-inflation component contains
  only an Intercept — lr.twosigma explicitly assumes the variable is in both components
  and that the ZI component has more than an Intercept; use lr.twosigma_custom with
  an explicit lr.df instead.
- The target is a random-effect variance component rather than a fixed effect — the
  package designates test.vc.twosigma for variance-component likelihood-ratio tests.
- Counts are already normalized or log-transformed — twosigma/lr.twosigma expect raw
  non-negative integer counts and perform no normalization.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3360
tools:
- name: twosigma
  role: R package providing lr.twosigma and lr.twosigma_custom for the joint fixed-effect
    likelihood-ratio test on the two-component ZINB mixed-model fits
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: GLMM engine used by twosigma to fit the alternative and null zero-inflated
    negative binomial models whose log-likelihoods form the LR statistic
- name: R
  role: Runtime environment; the twosigma package requires R >= 3.6.3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:4280d2fb267e16238d272f4fb07d3197e182b001faf70a93bebf2a493a58fb52
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edvanburen/twosigma
  tools:
  - R
  - glmmTMB
  - twosigma
  - twosigma (R package)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# likelihood-ratio-testing-of-regression-coefficients

## Summary

Per-gene joint likelihood-ratio testing of a fixed-effect covariate (e.g. binary disease status t2d_sim) in the TWO-SIGMA zero-inflated negative binomial mixed model, testing the joint null H0: alpha1 = beta1 = 0 across both the drop-out and conditional-mean components via the twosigma R package.

## When to use

Use when a covariate (e.g. t2d_sim) is entered in BOTH the mean_covar and zi_covar design matrices of a TWO-SIGMA ZINB mixed-model fit and you need per-gene LR statistics and p-values for its joint association with expression.

## When NOT to use

- The tested covariate is in only one component, or the zero-inflation component contains only an Intercept — lr.twosigma explicitly assumes the variable is in both components and that the ZI component has more than an Intercept; use lr.twosigma_custom with an explicit lr.df instead.
- The target is a random-effect variance component rather than a fixed effect — the package designates test.vc.twosigma for variance-component likelihood-ratio tests.
- Counts are already normalized or log-transformed — twosigma/lr.twosigma expect raw non-negative integer counts and perform no normalization.

## Inputs

- Raw scRNA-seq count matrix of non-negative integer counts (no normalization, no missing values), e.g. the two-gene simulated counts from twosigma example data
- mean_covar: covariate matrix for the conditional-mean model without an intercept (columns e.g. t2d_sim, age, cellular detection rate), one row per cell
- zi_covar: covariate matrix for the zero-inflation model without an intercept, containing the same covariate to test
- id: vector of individual-level IDs, one entry per cell (required)
- covar_to_test: column-name string (e.g. "t2d_sim") or integer column position present in both mean_covar and zi_covar
- For lr.twosigma_custom: mean_form_alt, mean_form_null (two-sided, response named 'count'), zi_form_alt, zi_form_null formulas and lr.df (non-negative integer)

## Outputs

- Per-gene likelihood-ratio statistic (lr.fit$LR_stat) for the joint null H0: alpha1 = beta1 = 0
- Per-gene likelihood-ratio p-value (lr.fit$LR_p.val)
- Optional per-gene results table (gene, mean-component coefficient, Z value, Wald p-value, LR statistic, LR p-value), e.g. written to twosigma_lr_results.csv

## How to apply

Build mean_covar and zi_covar matrices (model.matrix output WITHOUT intercept terms) that both contain the covariate to test, supply raw non-negative integer counts (no normalization) and the individual-level id vector, then call lr.twosigma(count_matrix, mean_covar, zi_covar, covar_to_test="t2d_sim"), which fits the alternative and null ZINB mixed models with glmmTMB and evaluates the joint null H0: alpha1 = beta1 = 0 (covariate absent from both the drop-out logit and conditional-mean log-linear components). covar_to_test may be a column-name string or an integer column position, but the integer path is not checked for correspondence across the two matrices, so verify column order first. Extract per-gene results as lr.fit$LR_stat and lr.fit$LR_p.val; with lr.twosigma_custom you must supply mean_form_alt/mean_form_null and zi_form_alt/zi_form_null yourself plus lr.df (a non-negative integer giving the LRT degrees of freedom, i.e. 2 constrained coefficients for the joint test) and ensure the formulas form a valid likelihood ratio test. Keep the random-effect specification deliberate: default adhoc=TRUE screens Pearson residuals via a one-way ANOVA F test on sample ID to decide random intercepts, and the paper warns that omitting random effects under within-sample correlation underestimates standard errors and inflates type-I error. Optionally pair the LR output with Wald-based summaries (coefficient, Z value, p from summary(fit)$coefficients$cond['t2d_sim',]) and assemble a per-gene CSV such as twosigma_lr_results.csv.

## Related tools

- **twosigma** (R package providing lr.twosigma and lr.twosigma_custom for the joint fixed-effect likelihood-ratio test on the two-component ZINB mixed-model fits) — https://github.com/edvanburen/twosigma
- **glmmTMB** (GLMM engine used by twosigma to fit the alternative and null zero-inflated negative binomial models whose log-likelihoods form the LR statistic)
- **R** (Runtime environment; the twosigma package requires R >= 3.6.3)

## Examples

```
lr.fit <- lr.twosigma(count_matrix, mean_covar, zi_covar, covar_to_test = "t2d_sim", adhoc = TRUE); lr.fit$LR_stat; lr.fit$LR_p.val
```

## Evaluation signals

- Output table contains exactly one row per simulated gene with a finite LR statistic (e.g. in twosigma_lr_results.csv).
- All Wald and LR p-values fall within [0, 1].
- LR p-value is consistent with the LR statistic under the reference with degrees of freedom equal to the number of constrained coefficients (2 for the joint H0: alpha1 = beta1 = 0; lr.df supplied for lr.twosigma_custom).
- For each gene, the conditional-mean estimates for t2d_sim extracted via summary(fit)$coefficients$cond['t2d_sim',] are finite whenever the fit returned a glmmTMB object rather than NA.
- Both mean_covar and zi_covar contain the tested covariate column before calling lr.twosigma, matching the function's stated assumption.

## Limitations

- If covar_to_test is given as an integer column position, there is no check that it corresponds to the same covariate in both the mean_covar and zi_covar matrices.
- Custom formulas must represent a valid likelihood ratio test and lr.df must be provided as a non-negative integer; the package places this responsibility on the user.
- The ad hoc random-effect screen is not recommended for gene set testing because geneset testing relies on a common gene-level null, which is broken when some genes have random effects and others do not.
- Ignoring random effect terms can lead to underestimated standard errors and thus inflated type-I errors (example given in the paper's table 3).
- LR testing over many genes is computationally heavy; multiple cores are recommended, particularly when using the likelihood ratio test.

## Evidence
