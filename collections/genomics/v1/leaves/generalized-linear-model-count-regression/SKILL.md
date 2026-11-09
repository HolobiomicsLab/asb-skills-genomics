---
name: generalized-linear-model-count-regression
description: Use when the input is a matrix of non-negative integer count data from
  scRNA-seq and inference is needed on covariates (e.g. disease status, age, cellular
  detection rate) in a conditional mean or drop-out component, with the choice of
  zero-inflation or random-effect specification under study.
when_to_use_negative:
- When cells from the same individual are known to be uncorrelated and the goal is
  a simple two-group comparison already served by dedicated tools (SCDE, scDD, DESingle,
  MAST) rather than covariate-controlled GLM inference on raw counts
- When the count matrix contains missing values or non-integer normalized values —
  twosigma requires non-negative integer counts with no missing values and performs
  no normalization itself
- When testing a covariate jointly across components via lr.twosigma but the zero-inflation
  component does not exist or contains only an intercept — use lr.twosigma_custom
  with explicit lr.df instead
edam_operation: http://edamontology.org/operation_3663
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: twosigma (R package)
  role: Fitting function wrapping glmmTMB to fit zero-inflated negative binomial GLMs/GLMMs
    per gene under user-specified mean and ZI covariate matrices; twosigma_custom
    accepts custom mean/zi formulas
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Underlying engine fitting the zero-inflated negative binomial (mixed) regression;
    ziformula ~0 / ~1 / covariate-inclusive selects the ZI specification
- name: R
  role: Runtime environment; package installable from CRAN or GitHub for R >= 3.6.3
  repo: https://cran.r-project.org/web/packages/twosigma/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:ee34638a1f00f9277d8e26bd05bf5c794a5df345fc6f0e8793773a2a3efd95b6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edvanburen/twosigma
  tools:
  - R
  - glmmTMB
  - twosigma (R package)
  edam_operation: http://edamontology.org/operation_3663
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Generalized linear model count regression

## Summary

Fit GLM-family regressions directly on observed scRNA-seq integer counts using glmmTMB via the twosigma R package, comparing zero-inflation specifications (no ZI ~0, intercept-only ~1, covariate-inclusive) and optionally adding random intercepts to preserve type-I error under within-sample correlation.

## When to use

Use when the input is a matrix of non-negative integer count data from scRNA-seq and inference is needed on covariates (e.g. disease status, age, cellular detection rate) in a conditional mean or drop-out component, with the choice of zero-inflation or random-effect specification under study.

## When NOT to use

- When cells from the same individual are known to be uncorrelated and the goal is a simple two-group comparison already served by dedicated tools (SCDE, scDD, DESingle, MAST) rather than covariate-controlled GLM inference on raw counts
- When the count matrix contains missing values or non-integer normalized values — twosigma requires non-negative integer counts with no missing values and performs no normalization itself
- When testing a covariate jointly across components via lr.twosigma but the zero-inflation component does not exist or contains only an intercept — use lr.twosigma_custom with explicit lr.df instead

## Inputs

- count_matrix: matrix/vector of non-negative integer counts per gene (one gene's count vector per fit), no missing values, no normalization
- mean_covar: model.matrix-style covariate matrix for the conditional mean model without an intercept term (e.g. binary disease status, age, cellular detection rate/CDR)
- zi_covar: covariate matrix for the zero-inflation (drop-out) model without an intercept term (0 = no ZI component, 1 = intercept-only ZI)
- id: vector of individual-level IDs, length equal to number of cells, required even when random effects are not used

## Outputs

- Per-gene fitted glmmTMB model objects with conditional-mean fixed-effect coefficient estimates, standard errors, z-values, and p-values
- A combined comparison table (e.g. zinb_specification_comparison.csv) with gene, model component, ZI specification, coefficient, estimate, and standard error
- ANOVA F-test p-values from the ad hoc residual screen flagging genes needing random effects

## How to apply

Pass a count matrix with no missing values, a mean_covar matrix built with model.matrix WITHOUT an intercept column, a zi_covar matrix (also without an intercept; use zi_covar=0 for no-ZI and zi_covar=1 for intercept-only drop-out), and a cell-to-individual id vector to twosigma, optionally with ncores for parallelization. The package uses glmmTMB to fit the zero-inflated negative binomial model, so covariate-inclusive ZI specifications can be fitted via twosigma_custom with formulas like count ~ mean_covar + (1|id) and ~ zi_covar + (1|id); extract coefficients from x$coefficients$cond of the returned glmmTMB summary. By default the ad hoc procedure runs a one-way ANOVA of Pearson's residuals on the sample indicator and uses the ANOVA F-test p-value to decide whether random intercepts are needed; to disable, set adhoc=FALSE and specify mean_re/zi_re explicitly, or test variance components with the likelihood-ratio function test.vc.twosigma. Assemble per-gene estimates and standard errors across all ZI specifications into one table (e.g. zinb_specification_comparison.csv) with columns for gene, model component, specification, coefficient, estimate, and standard error.

## Related tools

- **twosigma (R package)** (Fitting function wrapping glmmTMB to fit zero-inflated negative binomial GLMs/GLMMs per gene under user-specified mean and ZI covariate matrices; twosigma_custom accepts custom mean/zi formulas) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Underlying engine fitting the zero-inflated negative binomial (mixed) regression; ziformula ~0 / ~1 / covariate-inclusive selects the ZI specification)
- **R** (Runtime environment; package installable from CRAN or GitHub for R >= 3.6.3) — https://cran.r-project.org/web/packages/twosigma/

## Examples

```
fits <- twosigma(count_matrix, mean_covar = mean_covar_matrix, zi_covar = zi_covar_matrix, id = id, ncores = 4)
```

## Evaluation signals

- Output CSV contains rows for all three ZI specifications (~0, ~1, covariate-inclusive) per gene with finite coefficient estimates and standard errors
- No random-effect (variance component) terms appear in any fitted model when mean_re=FALSE/zi_re=FALSE or when fitting plain GLMs
- mean_covar and zi_covar matrices were built without intercept columns, as the interface expects intercept-free covariate matrices
- Extracted coefficients come from x$coefficients$cond of the glmmTMB summary (conditional-mean component), not the zi component, when mean-model inference is intended
- Ad hoc ANOVA F-test p-values are returned per gene and match genes where random effects were retained if the default adhoc=TRUE path was used

## Limitations

- The formula inputs of twosigma_custom are used directly, so it is the user's responsibility to ensure custom formulas are valid (e.g. response named 'count', valid likelihood-ratio df in lr.df)
- lr.twosigma assumes the tested variable is in both model components and that the ZI component exists with more than an intercept; integer covar_to_test values are not checked to match the same covariate in both matrices
- The ad hoc procedure is not recommended for gene set testing because a common gene-level null is required, and ignoring random effects can lead to underestimated standard errors and inflated type-I errors
- No normalization is performed on counts; batch must be controlled for by inclusion in the design matrices

## Evidence
