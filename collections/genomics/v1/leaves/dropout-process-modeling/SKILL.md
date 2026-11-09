---
name: dropout-process-modeling
description: Use when scRNA-seq count data exhibit excess zeros beyond sampling zero
  (drop-out/dropout) and you need to model or test covariate effects on the zero-inflation
  probability itself, or to simulate zero-inflated count data.
when_to_use_negative:
- When the analysis goal is gene set testing with a common gene-level null — the ad
  hoc procedure is explicitly 'not recommended for use in gene set testing' because
  mixed random-effect specifications break comparability across genes
- When zeros are better explained as hurdle/censoring on the log scale rather than
  a generative drop-out process — TWO-SIGMA contrasts with MAST's hurdle model fit
  on the log scale
- When counts are log-transformed or normalized features rather than raw non-negative
  integer counts, since the model works on observed counts directly and no normalization
  is done
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: twosigma
  role: R package implementing the zero-inflated negative binomial mixed model; its
    zero-inflation component models drop-out probability, with functions twosigma,
    adhoc.twosigma, test.vc.twosigma, and simulate_zero_inflated_nb_random_effect_data
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Fitting engine employed by the twosigma package for the ZINB mixed-effects
    model specified in equation (2)
- name: R
  role: Runtime environment; package requires R >= 3.6.3
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:85e37496f9df1632eb22581b1eb566eba068f70ed383a2c6620421a367af5464
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# dropout-process-modeling

## Summary

Explicitly models the drop-out (excess-zero) process in scRNA-seq data as a mixed-effects logistic regression, as in the TWO-SIGMA zero-inflated negative binomial framework, allowing covariates and random effects to drive the probability of an observed zero distinct from the mean count process.

## When to use

Use when scRNA-seq count data exhibit excess zeros beyond sampling zero (drop-out/dropout) and you need to model or test covariate effects on the zero-inflation probability itself, or to simulate zero-inflated count data.

## When NOT to use

- When the analysis goal is gene set testing with a common gene-level null — the ad hoc procedure is explicitly 'not recommended for use in gene set testing' because mixed random-effect specifications break comparability across genes
- When zeros are better explained as hurdle/censoring on the log scale rather than a generative drop-out process — TWO-SIGMA contrasts with MAST's hurdle model fit on the log scale
- When counts are log-transformed or normalized features rather than raw non-negative integer counts, since the model works on observed counts directly and no normalization is done

## Inputs

- Gene x cell non-negative integer count matrix (no missing values, no normalization applied by the tool)
- zi_covar: cell-level covariate matrix (e.g. model.matrix output) without intercept for the drop-out component
- mean_covar: cell-level covariate matrix without intercept for the mean component
- id: individual-level ID vector, one entry per cell
- Simulation parameters (nind, ncellsper, sigma.a, sigma.b, alpha, beta, phi) for generative use

## Outputs

- Per-gene glmmTMB model fits containing zero-inflation component coefficients (e.g. t2d_sim effect) and random-effect estimates
- ANOVA F-test p-values from adhoc.twosigma screening for genes needing random effects
- Likelihood ratio test results for variance components (test.vc.twosigma) or joint H0: alpha1 = beta1 = 0
- Simulated zero-inflated NB count matrix (e.g. 10,000 genes x 10,000 cells) with covariates, saved as simulated_zinb_data.rds

## How to apply

Model observed counts directly with a two-component zero-inflated negative binomial: a mixed-effects logistic regression for the drop-out probability (zi component) and a mixed-effects log-linear negative binomial regression for the conditional mean. Supply separate design matrices without intercepts (e.g. zi_covar with t2d_sim, age_sim, cdr_sim) and per-cell individual IDs, then either fit via glmmTMB (twosigma or twosigma_custom) or simulate from the same generative model using simulate_zero_inflated_nb_random_effect_data with parameters such as nind=10, ncellsper=1000, sigma.a=sigma.b=0.1, alpha=c(-1,0,-0.5,-2), beta=c(2,0,-0.1,0.6), phi=0.1. Decide whether random effects are needed in the zero-inflation component using the ad hoc procedure (one-way ANOVA of Pearson's residuals from a random-effect-free ZINB fit on the sample indicator, F-test p-value) or a likelihood ratio test of variance components (test.vc.twosigma). Rationale: ignoring within-sample dependency in the drop-out process yields underestimated standard errors and inflated type-I error.

## Related tools

- **twosigma** (R package implementing the zero-inflated negative binomial mixed model; its zero-inflation component models drop-out probability, with functions twosigma, adhoc.twosigma, test.vc.twosigma, and simulate_zero_inflated_nb_random_effect_data) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Fitting engine employed by the twosigma package for the ZINB mixed-effects model specified in equation (2))
- **R** (Runtime environment; package requires R >= 3.6.3)

## Examples

```
sim <- simulate_zero_inflated_nb_random_effect_data(nind=10, ncellsper=1000, sigma.a=0.1, sigma.b=0.1, alpha=c(-1,0,-0.5,-2), beta=c(2,0,-0.1,0.6), phi=0.1)
```

## Evaluation signals

- Saved simulation object has the expected dimensions: 10,000 genes x 10,000 cells (10 individuals x 1000 cells each)
- All simulated values are non-negative integers
- Zero inflation in the simulated data is consistent with the drop-out probability component built from alpha, the covariates, and random effects
- Ad hoc screening: significant ANOVA F-test p-values on Pearson residuals indicate genes where random effects in the zero-inflation/mean components are warranted
- Ignoring the random-effect structure (when present) should manifest as underestimated standard errors and inflated type-I error in downstream tests

## Limitations

- The ad hoc ANOVA screening is not suitable for gene set testing because a common gene-level null is not met when only some genes carry random effects
- The discussion notes it 'might be a pragmatic choice to exclude random effects from the zero-inflation component' in practice
- lr.twosigma assumes the tested covariate appears in both model components and that the zero-inflation component exists beyond an intercept; other cases require lr.twosigma_custom with user-supplied lr.df
- Custom formulas passed to twosigma_custom are used verbatim, so invalid likelihood ratio specifications are the user's responsibility

## Evidence
