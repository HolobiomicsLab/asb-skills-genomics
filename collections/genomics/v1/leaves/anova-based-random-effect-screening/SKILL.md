---
name: anova-based-random-effect-screening
description: Use when scRNA-seq counts contain multiple cells per individual/sample
  and you must decide, gene by gene, which genes warrant random effects (in the mean
  and/or zero-inflation component) before a full mixed-model TWO-SIGMA fit.
when_to_use_negative:
- 'Gene set testing with twosigmag: the README explicitly states the ad hoc procedure
  is not recommended there because set-level testing requires a common gene-level
  null, which is unclear when some genes get random effects and others do not'
- 'When you already know the random-effect specification: set adhoc=FALSE and pass
  mean_re/zi_re (or use twosigma_custom with (1|id) terms) instead of screening'
- When a formal test of whether variance components equal zero is required — the ad
  hoc p-value is a screening statistic, not the likelihood-ratio test of the variance
  component(s)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: twosigma
  role: R package providing the adhoc.twosigma function (and adhoc=TRUE default inside
    twosigma) that fits the ZINB model without random effects and returns the ANOVA
    F-test p-value
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Backend used by twosigma to fit the zero-inflated negative binomial regression
    model
- name: R
  role: Runtime environment; package requires R >= 3.6.3 (version 1.0.2 from CRAN
    or GitHub)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:f6254872a300570a7487763ce5cefa3dc0135ee8fded7004550a23893a45adb5
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# anova-based-random-effect-screening

## Summary

The TWO-SIGMA ad hoc procedure: per gene, fit a zero-inflated negative binomial (ZINB) model without random effects via glmmTMB, compute Pearson residuals, run a one-way ANOVA of residuals on individual ID, and use the F-test p-value to screen which genes most need random intercept terms.

## When to use

Use when scRNA-seq counts contain multiple cells per individual/sample and you must decide, gene by gene, which genes warrant random effects (in the mean and/or zero-inflation component) before a full mixed-model TWO-SIGMA fit.

## When NOT to use

- Gene set testing with twosigmag: the README explicitly states the ad hoc procedure is not recommended there because set-level testing requires a common gene-level null, which is unclear when some genes get random effects and others do not
- When you already know the random-effect specification: set adhoc=FALSE and pass mean_re/zi_re (or use twosigma_custom with (1|id) terms) instead of screening
- When a formal test of whether variance components equal zero is required — the ad hoc p-value is a screening statistic, not the likelihood-ratio test of the variance component(s)

## Inputs

- Raw single-cell count matrix (non-negative integer counts, cells x genes, no normalization)
- Vector of individual-level IDs (one per cell), e.g. sample indicator from nine pancreatic islet individuals
- mean_covar matrix of fixed covariates for the conditional mean model (no intercept, e.g. disease status, age, cellular detection rate)
- zi_covar matrix of fixed covariates for the zero-inflation model (no intercept)

## Outputs

- Per-gene ANOVA F-test p-value (one row per gene in the input count matrix, e.g. adhoc_anova_pvalues.csv)
- Flagged list of candidate genes to forward to the full TWO-SIGMA fit with random effects and the LRT-based variance-component test

## How to apply

For each gene in the raw count matrix (no normalization), fit the zero-inflated negative binomial model without random intercepts using the twosigma package's glmmTMB backend, including the drop-out probability and conditional mean components and cell-level covariates. Compute Pearson residuals per cell, with estimated drop-out probability, conditional mean, and overdispersion entering the standard-error denominator. Regress the Pearson residuals on the indicator of sample/individual ID via one-way ANOVA and extract the overall ANOVA F-test p-value. Genes with significant F-test p-values are candidates for the formal TWO-SIGMA fit with random effects and the likelihood-ratio variance-component test (test.vc.twosigma), while non-significant genes can be fit with fixed effects only, which dramatically reduces overall computation time over many genes and increases model parsimony where appropriate.

## Related tools

- **twosigma** (R package providing the adhoc.twosigma function (and adhoc=TRUE default inside twosigma) that fits the ZINB model without random effects and returns the ANOVA F-test p-value) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Backend used by twosigma to fit the zero-inflated negative binomial regression model)
- **R** (Runtime environment; package requires R >= 3.6.3 (version 1.0.2 from CRAN or GitHub))

## Examples

```
library(twosigma); pvals <- adhoc.twosigma(count, mean_covar, zi_covar, id)
```

## Evaluation signals

- Output contains exactly one ANOVA F-test p-value per gene in the input count matrix (rows match count_matrix rows)
- Residuals are Pearson residuals of a ZINB model without random effects — no glmmTMB random-intercept terms in either the mean or zero-inflation component
- The ANOVA is one-way with the individual/sample indicator as the grouping factor (id vector length equals total number of cells)
- Genes flagged with significant p-values, when forwarded to test.vc.twosigma, show nonzero variance components, providing evidence residuals are not exchangeable across samples
- Running with adhoc=FALSE and mean_re/zi_re set manually reproduces/ignores the ad hoc result as documented (mean_re and zi_re ignored and warning printed if adhoc=TRUE)

## Limitations

- Ignoring random effects when they exist leads to underestimated standard errors and inflated type-I errors, so screening significance must be acted upon
- The ad hoc procedure is not recommended for gene set testing because a common gene-level null is unclear when some genes have random effects and others do not
- The ANOVA p-value is an ad hoc screening statistic; the formal assessment of needing random effects is the likelihood-ratio test of whether variance components equal zero

## Evidence

- [methods] Ad hoc ANOVA mechanics: "using a one-way ANOVA, we regress the Pearson’s residuals on an indicator of sample and take the p-value from the overall ANOVA F test."
- [methods] Rationale for ad hoc screening: [evidence span withheld — anchor status: not_found]
- [readme] README description of adhoc.twosigma: "This method fits a zero-inflated negative binomial model without random effects and uses a one-way ANOVA regressing the Pearson residuals on the individual ID to look for differences between"
- [readme] p-value usage: "The p-value from the ANOVA F test is returned, and can be used as a screening for genes that are most in need of random effects."
- [readme] Gene-set testing caveat: "The adhoc procedure is not recommended for use in gene set testing"
