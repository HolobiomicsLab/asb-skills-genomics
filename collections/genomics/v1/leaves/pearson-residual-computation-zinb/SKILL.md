---
name: pearson-residual-computation-zinb
description: Use when analyzing single-cell scRNA-seq count data (e.g. the pancreatic
  islet dataset with individual/sample IDs) to decide which genes require random intercepts
  before a full TWO-SIGMA mixed-model fit.
when_to_use_negative:
- 'When performing TWO-SIGMA-G competitive gene set testing: the ad hoc procedure
  is not recommended there because a common gene-level null is required and mixed
  random-effect specifications break comparability across genes'
- When you already know the random effect specification and can supply mean_re/zi_re
  directly with adhoc=FALSE instead of screening
- When counts are normalized/log-transformed — twosigma models observed non-negative
  integer counts directly with no normalization
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: twosigma
  role: R package providing adhoc.twosigma, which fits the zero-inflated negative
    binomial model without random effects and returns the ANOVA F test p-value on
    its Pearson residuals
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Backend used by twosigma to fit the ZINB regression models from which the
    Pearson residuals are computed
- name: R
  role: Runtime environment (R >= 3.6.3) for the twosigma package and the one-way
    ANOVA of residuals on individual ID
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:725a790c68f487a3a3dcc0844c7bd147bfe5d24b7c9d1e83a188b114ab5f6926
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pearson-residual-computation-zinb

## Summary

Computes Pearson residuals from a zero-inflated negative binomial regression fitted without random effects, using the twosigma R package (glmmTMB backend), as the basis of the ad hoc screening for genes needing random effect terms.

## When to use

Use when analyzing single-cell scRNA-seq count data (e.g. the pancreatic islet dataset with individual/sample IDs) to decide which genes require random intercepts before a full TWO-SIGMA mixed-model fit.

## When NOT to use

- When performing TWO-SIGMA-G competitive gene set testing: the ad hoc procedure is not recommended there because a common gene-level null is required and mixed random-effect specifications break comparability across genes
- When you already know the random effect specification and can supply mean_re/zi_re directly with adhoc=FALSE instead of screening
- When counts are normalized/log-transformed — twosigma models observed non-negative integer counts directly with no normalization

## Inputs

- Raw single-cell count matrix (non-negative integer counts, one row per gene, one column per cell; no normalization applied)
- mean_covar matrix of covariates for the conditional mean model (no intercept, rows = cells)
- zi_covar matrix of covariates for the zero-inflation model (no intercept, rows = cells)
- Vector of individual-level IDs (length = number of cells), e.g. from the pancreatic islet dataset with nine individuals

## Outputs

- Per-gene Pearson residuals over all cells
- Per-gene ANOVA F test p-value (one p-value per gene, e.g. written to adhoc_anova_pvalues.csv) used to flag genes most in need of random effects

## How to apply

For each gene, fit the zero-inflated negative binomial regression with drop-out probability and conditional mean components but omitting random intercepts, via the twosigma R package's glmmTMB backend. Compute Pearson residuals per cell, with the estimated drop-out probability, conditional mean, and overdispersion entering the standard-error denominator of the ZINB residual definition. Regress the Pearson residuals on the individual/sample ID via one-way ANOVA and extract the overall ANOVA F test p-value; this p-value (returned by adhoc.twosigma) is the screening statistic forwarded so that significant genes receive the full TWO-SIGMA fit with random effects. This ad hoc procedure is employed by default inside the twosigma function to decide random effect inclusion and can dramatically reduce computation time and increase model parsimony.

## Related tools

- **twosigma** (R package providing adhoc.twosigma, which fits the zero-inflated negative binomial model without random effects and returns the ANOVA F test p-value on its Pearson residuals) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Backend used by twosigma to fit the ZINB regression models from which the Pearson residuals are computed)
- **R** (Runtime environment (R >= 3.6.3) for the twosigma package and the one-way ANOVA of residuals on individual ID)

## Examples

```
pvals <- adhoc.twosigma(count, mean_covar, zi_covar, id)
```

## Evaluation signals

- Output contains exactly one ANOVA F-test p-value row per gene in the input count matrix
- Residuals must be computed from the ZINB fit WITHOUT random effect terms, with drop-out probability, conditional mean, and overdispersion in the residual standard-error denominator
- The fitted ZINB model is identical to the TWO-SIGMA specification of equation (2) minus the random intercepts in each component
- Genes with significant F-test p-values should overlap with genes that would fail a likelihood-ratio test of zero variance components
- twosigma's internal adhoc output (with adhoc=TRUE, default) should match a direct call to adhoc.twosigma for the same gene, covariates, and id vector

## Limitations

- The ad hoc method is a screening heuristic; formal evaluation of the need for random effects is via a likelihood ratio test of whether variance components equal zero
- The ad hoc procedure is not recommended for use in TWO-SIGMA-G gene set testing, since a common gene-level null hypothesis is not met when only some genes include random effects
- The id argument is required even when neither random effect prediction nor the ad hoc method is used
- Ignoring random effects when warranted leads to underestimated standard errors and inflated type-I errors

## Evidence
