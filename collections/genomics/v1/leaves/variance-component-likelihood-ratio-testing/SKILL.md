---
name: variance-component-likelihood-ratio-testing
description: Use when a fitted or candidate TWO-SIGMA zinb mixed model may require
  random intercepts in the mean model, the zero-inflation (dropout) model, or both,
  and you need a formal hypothesis test of whether the corresponding variance component(s)
  equal zero.
when_to_use_negative:
- The goal is fixed-effect association testing of a covariate such as disease status
  — use lr.twosigma or lr.twosigma_custom for the fixed-effect likelihood ratio test
  instead of a variance-component test.
- You only need a cheap screen for which genes may benefit from random effects before
  fitting — the ad hoc method (adhoc.twosigma, one-way ANOVA of Pearson residuals
  on sample indicator) is the intended lightweight alternative.
- Custom fixed/random-effect formulas are required — use twosigma_custom and construct
  the likelihood ratio test yourself, ensuring the formulas represent a valid LRT
  and supplying lr.df.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: twosigma
  role: R package providing test.vc.twosigma for variance-component LRTs, with mean_re
    and zi_re controlling which components contain random effects under the alternative;
    also twosigma/twosigma_custom for fitting the underlying glmmTMB models and simulate_zero_inflated_nb_random_effect_data
    for simulation studies
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Backend used by the twosigma package to fit the zero-inflated negative binomial
    mixed model (equation 2) whose log-likelihoods enter the LRT statistic
- name: R
  role: Runtime environment; the twosigma package requires R >= 3.6.3
  repo: https://cran.r-project.org/web/packages/twosigma/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:8139d43f379dfd2bd2770810eb2c0ee87ddf8014304ac08a31b6922a4b954a0d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edvanburen/twosigma
  tools:
  - R
  - glmmTMB
  - twosigma
  - twosigma (R package)
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.631578947368421
n_claims: 19
---

# variance-component-likelihood-ratio-testing

## Summary

Assesses whether random-intercept terms (variance components) are needed in the mean component, the zero-inflation component, or both components of the TWO-SIGMA zero-inflated negative binomial mixed model by fitting nested glmmTMB models and computing likelihood ratio tests, as implemented in the twosigma function test.vc.twosigma.

## When to use

Use when a fitted or candidate TWO-SIGMA zinb mixed model may require random intercepts in the mean model, the zero-inflation (dropout) model, or both, and you need a formal hypothesis test of whether the corresponding variance component(s) equal zero.

## When NOT to use

- The goal is fixed-effect association testing of a covariate such as disease status — use lr.twosigma or lr.twosigma_custom for the fixed-effect likelihood ratio test instead of a variance-component test.
- You only need a cheap screen for which genes may benefit from random effects before fitting — the ad hoc method (adhoc.twosigma, one-way ANOVA of Pearson residuals on sample indicator) is the intended lightweight alternative.
- Custom fixed/random-effect formulas are required — use twosigma_custom and construct the likelihood ratio test yourself, ensuring the formulas represent a valid LRT and supplying lr.df.

## Inputs

- Raw count matrix/vector of non-negative integer counts for a gene (no normalization, no missing values)
- mean_covar matrix of covariates for the conditional mean model without an intercept term
- zi_covar matrix of covariates for the zero-inflation model without an intercept term
- Vector of individual-level IDs (id), one entry per cell, used for random effect prediction
- Logical flags mean_re and zi_re indicating which components contain random intercepts under the alternative
- Log-likelihoods from glmmTMB fits of the full and reduced (null) models when constructing the test manually

## Outputs

- LRT statistic -2*(l_H0 - l_Ha) for each of three configurations: mean-component-only, zero-inflation-component-only, and joint both-component random effect retention
- p-value per configuration: 50:50 mixture of chi-square(1 df) and point mass at zero for single-component tests; chi-square(2 df) for the joint test
- Results table (e.g. variance_component_lrt_results.csv) with statistic and p-value rows per configuration

## How to apply

Fit the full TWO-SIGMA model (e.g. via glmmTMB through twosigma or twosigma_custom) with random intercepts in both components and record the log-likelihood, then fit reduced models dropping the mean-component intercept (sigma_b = 0), the zero-inflation-component intercept (sigma_a = 0), and both. Compute the LRT statistic as -2*(l_H0 - l_Ha) for each configuration. Assign p-values with the correct boundary-null reference: single-component tests use a 50:50 mixture of chi-square(1 df) and a point mass at zero, while the joint two-component test uses a conservative chi-square with 2 degrees of freedom. Alternatively, call test.vc.twosigma, whose mean_re and zi_re arguments control which components contain random effects under the alternative; note that if adhoc=TRUE, mean_re and zi_re are ignored and a warning is printed. Record the statistic and p-value for each of the three configurations in an output table (e.g. variance_component_lrt_results.csv).

## Related tools

- **twosigma** (R package providing test.vc.twosigma for variance-component LRTs, with mean_re and zi_re controlling which components contain random effects under the alternative; also twosigma/twosigma_custom for fitting the underlying glmmTMB models and simulate_zero_inflated_nb_random_effect_data for simulation studies) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Backend used by the twosigma package to fit the zero-inflated negative binomial mixed model (equation 2) whose log-likelihoods enter the LRT statistic)
- **R** (Runtime environment; the twosigma package requires R >= 3.6.3) — https://cran.r-project.org/web/packages/twosigma/

## Examples

```
fits<-twosigma(count_matrix, mean_covar=mean_covar_matrix, zi_covar=zi_covar_matrix, mean_re = TRUE, zi_re = TRUE, id=id,adhoc=F)
```

## Evaluation signals

- Output file exists and contains a row with statistic and p-value for each of the three configurations (mean-only, ZI-only, joint)
- All p-values lie in [0,1]
- All LRT statistics are non-negative, and the full-model log-likelihood is at least as large as every reduced-model log-likelihood
- Single-component p-values are computed from the 50:50 chi-square(1)/point-mass-at-zero mixture, not a plain chi-square(1), and the joint p-value uses the conservative chi-square(2 df)

## Limitations

- Variance components lie on the boundary of the parameter space under the null, so standard chi-square(1) p-values are invalid; the 50:50 mixture and conservative chi-square(2 df) reference distributions must be used.
- If adhoc=TRUE in twosigma, the mean_re and zi_re arguments are ignored and a warning is printed; explicit random-effect specification requires adhoc=FALSE.
- The paper's ad hoc ANOVA-based screening is only a heuristic for selecting genes needing random effects, and the README warns the ad hoc procedure is not recommended for gene set testing because a common gene-level null is broken when some genes have random effects and others do not.
- Ignoring needed random effects can lead to underestimated standard errors and inflated type-I errors, so incorrectly omitting this step can dramatically affect scientific conclusions (as shown in the pancreatic islet analysis).

## Evidence
