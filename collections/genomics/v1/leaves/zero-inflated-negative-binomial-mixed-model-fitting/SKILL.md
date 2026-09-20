---
name: zero-inflated-negative-binomial-mixed-model-fitting
description: Use when raw non-negative integer scRNA-seq counts (gene x cell matrix)
  must be modeled with excess zeros and overdispersion, while controlling cell-level/subject-level
  covariates and accommodating within-sample correlation across cells from the same
  individual.
when_to_use_negative:
- Input counts are already normalized or transformed — the package expects raw non-negative
  integer counts and performs no normalization itself
- The goal is a set-level or contrast-matrix test rather than per-gene model fitting
  — use twosigmag instead (contrast-matrix support in twosigma_custom was noted as
  ongoing in the README)
- Genes are being pooled for competitive gene set testing where the ad hoc procedure
  is explicitly not recommended, since a common gene-level null is unclear when some
  genes have random effects and others do not
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: twosigma (R package)
  role: 'Primary fitting interface: twosigma() and twosigma_custom() fit the two-component
    ZINB mixed model per gene, with the ad hoc random-effect selection procedure built
    in'
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Backend optimizer that fits the mixed-effects logistic and negative-binomial
    components specified by equation (2)
- name: R
  role: Runtime environment; package requires R >= 3.6.3 and imports multcomp, pscl,
    pbapply, parallel, doParallel
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_003
  - task_006
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:ac16404f777ef2e84919603a21122d1bfa6bd454645a3722c6b128a05324a1bd
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# zero-inflated-negative-binomial-mixed-model-fitting

## Summary

Fits the TWO-SIGMA two-component zero-inflated negative binomial mixed model (logistic drop-out component + log-linear negative binomial conditional-mean component, each with optional sample-level random intercepts) to per-gene scRNA-seq counts using the twosigma R package's glmmTMB backend.

## When to use

Use when raw non-negative integer scRNA-seq counts (gene x cell matrix) must be modeled with excess zeros and overdispersion, while controlling cell-level/subject-level covariates and accommodating within-sample correlation across cells from the same individual.

## When NOT to use

- Input counts are already normalized or transformed — the package expects raw non-negative integer counts and performs no normalization itself
- The goal is a set-level or contrast-matrix test rather than per-gene model fitting — use twosigmag instead (contrast-matrix support in twosigma_custom was noted as ongoing in the README)
- Genes are being pooled for competitive gene set testing where the ad hoc procedure is explicitly not recommended, since a common gene-level null is unclear when some genes have random effects and others do not

## Inputs

- Raw non-negative integer count matrix (genes x cells), no missing values, no normalization applied
- mean_covar: cell-level covariate matrix for the conditional-mean model WITHOUT an intercept term
- zi_covar: cell-level covariate matrix for the zero-inflation model WITHOUT an intercept term
- id: vector of individual-level IDs, one entry per cell (length = total number of cells)
- Optional: custom lme4-style mean_form (two-sided, response named 'count') and zi_form (one-sided) for twosigma_custom

## Outputs

- Per-gene fitted glmmTMB model objects (accessible via fits$fit) containing coefficient estimates, standard errors, Z values, and Wald p-values
- Conditional-mean component fixed-effect estimates (e.g. for covariates like t2d_sim, age, CDR) extractable per gene
- Random-effect variance component estimates and BLUPs for the sample-level random intercepts
- Ad hoc ANOVA F-test p-values screening genes that need random effects (when adhoc=TRUE)

## How to apply

Call twosigma(count_matrix, mean_covar, zi_covar, id) passing raw (non-normalized) count rows, covariate matrices WITHOUT intercept terms built via model.matrix, and the individual-level ID vector; by default the ad hoc procedure fits a ZINB model without random effects, computes Pearson residuals, and runs a one-way ANOVA of residuals on individual ID to decide whether random intercepts enter the mean and/or zero-inflation components. To bypass the ad hoc screen set adhoc=FALSE and specify mean_re and zi_re directly, or use twosigma_custom(count_matrix, mean_form, zi_form, id) with lme4-style formulas such as count~mean_covar+(1|id) and ~zi_covar+(1|id), ensuring the response is named 'count'. Fitting goes through glmmTMB, so mixed-effects logistic and negative-binomial components are optimized jointly per gene; disp_covar can carry covariates for the dispersion. Predictions of random effects require the id argument, which is mandatory even if no random effects are used. Extract conditional-mean fixed-effect coefficients, Z values, and Wald p-values from the returned glmmTMB fits (e.g. summary(fit)$coefficients$cond['t2d_sim',]) for downstream testing.

## Related tools

- **twosigma (R package)** (Primary fitting interface: twosigma() and twosigma_custom() fit the two-component ZINB mixed model per gene, with the ad hoc random-effect selection procedure built in) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Backend optimizer that fits the mixed-effects logistic and negative-binomial components specified by equation (2))
- **R** (Runtime environment; package requires R >= 3.6.3 and imports multcomp, pscl, pbapply, parallel, doParallel)

## Examples

```
fits <- twosigma(count_matrix, mean_covar=mean_covar_matrix, zi_covar=zi_covar_matrix, mean_re = TRUE, zi_re = TRUE, id=id, adhoc=F)
```

## Evaluation signals

- Each returned fit is a valid glmmTMB object (or, per README convention, NA/character if fitting failed — check no unexpected NAs among genes that should fit)
- Covariate design matrices passed to mean_covar/zi_covar contain no intercept column, as the package requires
- Extracted conditional-mean coefficients, Z values, and Wald p-values via $coefficients$cond are finite, with p-values in [0,1]
- With mean_re=TRUE and zi_re=TRUE the specification matches the default TWO-SIGMA model with random intercepts in both components, reproducible via twosigma_custom(count, mean_form=count~mean_covar+(1|id), zi_form=~zi_covar+(1|id), id=id)
- One fitted model per gene in the input count matrix, with the individual-level id vector length equal to the total number of cells

## Limitations

- twosigma assumes counts are raw integers; no normalization is performed, so batch or sequencing-depth effects must be controlled by inclusion in the design matrices
- Formula inputs to twosigma_custom are used directly and the user is responsible for ensuring they operate as expected and (for LRTs) represent a valid likelihood ratio test
- Ignoring random effects when cells from the same sample are even moderately correlated leads to underestimated standard errors and inflated type-I errors
- Failing to include random effect terms can have dramatic impacts on scientific conclusions, as shown in the pancreas islet single-cell analysis of nine individuals

## Evidence
