---
name: r-statistical-programming
description: Use when reproducing or extending TWO-SIGMA / TWO-SIGMA-G analyses in
  R — e.g., fitting zero-inflated NB mixed models to a count matrix and comparing
  set-level p-values from the 'Z' and statistic='contrast' gene-level tests.
when_to_use_negative:
- When the task requires normalized expression (twosigma takes raw integer counts
  and performs no normalization itself)
- When using twosigmag with the ad hoc procedure — the README states 'The adhoc procedure
  is not recommended for use in gene set testing' because a common gene-level null
  is required
- When the goal is two-group-only DE analysis better served by dedicated tools like
  MAST, SCDE, scDD, or DESingle rather than covariate-flexible mixed modeling
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
tools:
- name: twosigma
  role: R package providing the twosigma, twosigma_custom, lr.twosigma, adhoc.twosigma,
    test.vc.twosigma, and twosigmag functions for model fitting, fixed-effect testing,
    and gene set testing
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Backend used by twosigma to fit the zero-inflated negative binomial mixed-effects
    model specified in the methods
- name: R
  role: Runtime environment (>= 3.6.3) in which the package is installed and the simulated
    10-gene example is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:5586d5d24e4adc06017b14fb4dac07c8d07da9a82f71a0d02b8a9230f85442c9
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# r-statistical-programming

## Summary

Use R (>= 3.6.3) with the twosigma package (built on glmmTMB) to fit zero-inflated negative binomial mixed models for single-cell RNA-seq differential expression and gene set testing, including the Z-statistic vs. contrast-matrix routes in twosigmag.

## When to use

Use when reproducing or extending TWO-SIGMA / TWO-SIGMA-G analyses in R — e.g., fitting zero-inflated NB mixed models to a count matrix and comparing set-level p-values from the 'Z' and statistic='contrast' gene-level tests.

## When NOT to use

- When the task requires normalized expression (twosigma takes raw integer counts and performs no normalization itself)
- When using twosigmag with the ad hoc procedure — the README states 'The adhoc procedure is not recommended for use in gene set testing' because a common gene-level null is required
- When the goal is two-group-only DE analysis better served by dedicated tools like MAST, SCDE, scDD, or DESingle rather than covariate-flexible mixed modeling

## Inputs

- R count_matrix of non-negative integer single-cell counts (genes x cells), no normalization applied
- mean_covar and zi_covar numeric design matrices (cells x covariates, no intercept), e.g. from model.matrix
- Vector of individual-level sample IDs, one per cell
- Gene set index lists (e.g. index_test=list(1:5, 6:10)) and optional custom contrast matrix

## Outputs

- Per-gene model fits (glmmTMB objects) with coefficients, z-values, and p-values for the mean and zero-inflation components
- Set-level p-values from twosigmag for each test/reference set pair under each gene-level statistic route
- Combined p-value table (e.g. pvalues_table.csv) with rows for gene sets {1:5} and {6:10} and columns for 'Z' and 'contrast' routes

## How to apply

Install the twosigma package from CRAN (version 1.0.2, R >= 3.6.3) or GitHub, noting minimum imported versions (multcomp >= 1.4-13, pscl >= 1.5.5, pbapply >= 1.4.0, doParallel >= 1.0.15). Fit gene-level models with twosigma(count_matrix, mean_covar, zi_covar, id, ncores), supplying design matrices WITHOUT intercept terms and a per-cell ID vector; the ad hoc one-way ANOVA on Pearson residuals decides random-effect inclusion unless adhoc=FALSE. For gene set testing, call twosigmag with index_test, statistic='Z' or statistic='contrast', and all_as_ref=TRUE, testing sets such as {1:5} (DE genes with non-zero beta2) vs {6:10} (non-DE controls) on the 10-gene simulated matrix from simulate_zero_inflated_nb_random_effect_data. Use ncores > 1 for parallelization, especially with the LR statistic. Verify that the 'Z' and 'contrast' routes give identical set-level p-values within floating-point tolerance and that the DE set is significant while the non-DE set is not.

## Related tools

- **twosigma** (R package providing the twosigma, twosigma_custom, lr.twosigma, adhoc.twosigma, test.vc.twosigma, and twosigmag functions for model fitting, fixed-effect testing, and gene set testing) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Backend used by twosigma to fit the zero-inflated negative binomial mixed-effects model specified in the methods)
- **R** (Runtime environment (>= 3.6.3) in which the package is installed and the simulated 10-gene example is executed)

## Examples

```
library(twosigma); fits <- twosigma(count_matrix, mean_covar, zi_covar, id, ncores=4); res <- twosigmag(fits, index_test=list(1:5, 6:10), statistic='contrast')
```

## Evaluation signals

- pvalues_table.csv exists and contains four p-values (two gene sets x two statistic routes)
- statistic='Z' and statistic='contrast' set-level p-values agree for both gene sets within floating-point tolerance
- The non-DE set {6:10} is non-significant while the DE set {1:5} is significant (smaller p-value)
- Gene-level fits return non-NA coefficients for the 't2d_sim' covariate rather than the character NA placeholder used when fitting fails
- Random-effect screening via adhoc.twosigma returns an ANOVA F-test p-value per gene, and adhoc=TRUE suppresses explicit mean_re/zi_re with a warning

## Limitations

- The ad hoc procedure should not be used for gene set testing because genes must share a common gene-level null hypothesis
- twosigma performs no normalization; the user supplies raw non-negative integer counts with no missing values
- Negative set-level correlations are zeroed by default (allow_neg_corr=FALSE) to be conservative, which may miss true depletion signals
- Custom contrast-matrix testing at the gene level is still under development ('Functionality to do this is ongoing') and routed through twosigmag

## Evidence
