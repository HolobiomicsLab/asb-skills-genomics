---
name: boundary-null-hypothesis-pvalue-calibration
description: Use when testing whether random-intercept variance components (sigma_b
  in the mean component, sigma_a in the zero-inflation component) equal zero via LRT
  in a mixed-effects model, where standard chi-square reference distributions do not
  apply.
when_to_use_negative:
- Testing fixed-effect covariates (e.g. disease status) rather than random-effect
  variance components — use lr.twosigma or the standard LRT framework instead
- Genes where the ad hoc ANOVA screening on Pearson residuals indicates random effects
  are clearly unnecessary, where fitting the reduced model directly is more parsimonious
  and computationally cheaper
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: twosigma
  role: R package providing test.vc.twosigma for likelihood ratio tests of random
    effect variance components, with mean_re and zi_re arguments controlling which
    components contain random effects under the alternative
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Fits the full and reduced zero-inflated negative binomial mixed models whose
    log-likelihoods feed the LRT statistic
- name: R
  role: Runtime environment for fitting models and computing the mixture chi-square
    p-values
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:40f2e2cc817496f8cd6e86669fdbca4ed2a1e7ce2399ad98337023e6e0ec9495
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edvanburen/twosigma
  tools:
  - R
  - glmmTMB
  - twosigma
  - twosigma (R package)
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# boundary-null-hypothesis-pvalue-calibration

## Summary

Calibrates p-values for likelihood ratio tests of variance components (random-effect variances) in the TWO-SIGMA zero-inflated negative binomial mixed model, accounting for the non-standard boundary null (variance components constrained to the edge of the parameter space).

## When to use

Use when testing whether random-intercept variance components (sigma_b in the mean component, sigma_a in the zero-inflation component) equal zero via LRT in a mixed-effects model, where standard chi-square reference distributions do not apply.

## When NOT to use

- Testing fixed-effect covariates (e.g. disease status) rather than random-effect variance components — use lr.twosigma or the standard LRT framework instead
- Genes where the ad hoc ANOVA screening on Pearson residuals indicates random effects are clearly unnecessary, where fitting the reduced model directly is more parsimonious and computationally cheaper

## Inputs

- Count matrix of non-negative integer scRNA-seq counts for the gene(s) under test
- mean_covar matrix of covariates for the conditional mean model (no intercept term)
- zi_covar matrix of covariates for the zero-inflation model (no intercept term)
- id vector of individual-level sample IDs (length equal to number of cells)
- Fitted model log-likelihoods under the alternative and under each null configuration

## Outputs

- LRT statistics (-2*(l_H0 - l_Ha)) for each variance-component configuration (mean-only, zero-inflation-only, joint)
- Boundary-calibrated p-values: 50:50 chi-square(1)/point-mass mixture for single-component tests, chi-square(2) for the joint test
- A results table (e.g. variance_component_lrt_results.csv) with one row per configuration containing statistic and p-value

## How to apply

Fit the full TWO-SIGMA model (e.g. with random intercepts in both components via glmmTMB) and record its log-likelihood, then fit the reduced model(s) with the variance component(s) set to zero and record each log-likelihood. Compute the LRT statistic as -2*(l_H0 - l_Ha); this can be done directly with the test.vc.twosigma function from the twosigma R package, using the mean_re and zi_re arguments to control which component(s) contain random effects under the alternative. Assign p-values using boundary-null reference distributions: for a single-component test use a 50:50 mixture of chi-square(1 df) and a point mass at zero, and for the joint two-component test use a conservative chi-square with 2 degrees of freedom. Rationale: variance components are constrained to be non-negative, so the null lies on the boundary of the parameter space and the usual chi-square(df) approximation is invalid.

## Related tools

- **twosigma** (R package providing test.vc.twosigma for likelihood ratio tests of random effect variance components, with mean_re and zi_re arguments controlling which components contain random effects under the alternative) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Fits the full and reduced zero-inflated negative binomial mixed models whose log-likelihoods feed the LRT statistic)
- **R** (Runtime environment for fitting models and computing the mixture chi-square p-values)

## Examples

```
library(twosigma); vc <- test.vc.twosigma(count_matrix, mean_covar, zi_covar, id, mean_re = TRUE, zi_re = TRUE)
```

## Evaluation signals

- All LRT statistics are non-negative (consistent with l_H0 <= l_Ha for nested models)
- All p-values lie in [0, 1]
- The results table contains one row with a statistic and p-value for each of the three configurations: sigma_b = 0, sigma_a = 0, and both
- Single-component p-values reproduce a 50:50 mixture of chi-square(1 df) and a point mass at zero; the joint p-value uses chi-square with 2 df
- Statistic equals -2*(l_H0 - l_Ha) when recomputed from the recorded full and reduced log-likelihoods

## Limitations

- The joint two-component test uses a conservative chi-square(2 df) reference, which may sacrifice power relative to exact boundary-null mixtures
- The ad hoc procedure (ANOVA on Pearson residuals) is a screening heuristic and is explicitly not recommended for gene set testing, where a common gene-level null must be maintained
- The article's discussion suggests that excluding random effects from the zero-inflation component may be a pragmatic choice in some settings, so testing sigma_a may be of secondary practical interest

## Evidence

- [methods] Evaluating the need for a random effect term involves a hypothesis test of whether the corresponding variance component(s) equal zero.: "Evaluating the need for a random effect term involves a hypothesis test of whether the corresponding variance component(s) equal zero."
- [readme] one can use the likelihood ratio test to test either one or both components for random effect terms via the function test.vc.twosigma: "one can use the likelihood ratio test to test either one or both components for random effect terms via the function `test.vc.twosigma` Which components contain random effects under the alternative"
- [methods] Simulated covariates mirrored our pancreatic real data analysis and included a binary disease status, age and the cellular detection rate (CDR): "Simulated covariates mirrored our pancreatic real data analysis and included a binary disease status, age and the cellular detection rate (CDR"
