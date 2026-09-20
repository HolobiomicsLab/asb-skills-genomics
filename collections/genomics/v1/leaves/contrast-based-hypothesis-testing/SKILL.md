---
name: contrast-based-hypothesis-testing
description: 'Use when you need to test custom contrasts of regression coefficients
  in a TWO-SIGMA zero-inflated NB model — e.g., joint testing of H0: alpha1 = beta1
  = 0 — or to verify that the contrast route reproduces the Z-statistic gene-level
  set-level p-values.'
when_to_use_negative:
- When testing a covariate present in both mean and zero-inflation components of the
  model — use lr.twosigma or lr.twosigma_custom likelihood-ratio testing instead
- When genes in the test set would have heterogeneous random-effect specifications
  (adhoc screening applied to some genes only) — the common gene-level null required
  by twosigmag is violated
- When the covariate of interest is only in the mean component and a simple gene-level
  Wald/Z test suffices — the contrast route via twosigmag is unnecessary machinery
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0203
tools:
- name: twosigma (R package)
  role: Fits the two-component zero-inflated negative binomial mixed model whose fixed-effect
    coefficients the contrast matrix tests; its twosigmag function executes contrast-based
    gene/set testing
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Backend used by the twosigma package to fit the model specified in equation
    (2), supplying the coefficient estimates and standard errors consumed by the contrast
    test
- name: R
  role: Implementation environment; the twosigmag contrast workflow and simulated-data
    generation run in R (>= 3.6.3, package version 1.0.2)
  repo: https://cran.r-project.org/web/packages/twosigma/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:ae88c6a5e1c18c0809e9242074c8eb9e25a568e7a63bd2ac012cbc65b6921f32
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edvanburen/twosigma
  tools:
  - R
  - glmmTMB
  - twosigma (R package)
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# contrast-based-hypothesis-testing

## Summary

Test fixed-effect contrasts (e.g., disease-status regression coefficients) in the TWO-SIGMA zero-inflated negative binomial mixed model by routing a custom contrast matrix through the twosigmag gene-set testing function, yielding set-level p-values for each contrast row that agree with the Z-statistic gene-level route.

## When to use

Use when you need to test custom contrasts of regression coefficients in a TWO-SIGMA zero-inflated NB model — e.g., joint testing of H0: alpha1 = beta1 = 0 — or to verify that the contrast route reproduces the Z-statistic gene-level set-level p-values.

## When NOT to use

- When testing a covariate present in both mean and zero-inflation components of the model — use lr.twosigma or lr.twosigma_custom likelihood-ratio testing instead
- When genes in the test set would have heterogeneous random-effect specifications (adhoc screening applied to some genes only) — the common gene-level null required by twosigmag is violated
- When the covariate of interest is only in the mean component and a simple gene-level Wald/Z test suffices — the contrast route via twosigmag is unnecessary machinery

## Inputs

- Fitted TWO-SIGMA model object (fits$fit from twosigma or twosigma_custom, glmmTMB fits)
- Non-negative integer count matrix (raw scRNA-seq counts, no normalization)
- mean_covar and zi_covar matrices (no intercept term), plus id vector of individual-level IDs
- Custom contrast matrix over fixed-effect regression coefficients (rows = contrasts to test)
- index_test list of numeric gene indices (e.g., {1:5} DE set, {6:10} non-DE set from simulate_zero_inflated_nb_random_effect_data)

## Outputs

- Set-level p-values, one per row of the contrast matrix, for each test/reference set combination
- Gene-level contrast output (usable when index_test=list(c(1,2)) and all_as_ref=TRUE)
- Combined p-value table (e.g., pvalues_table.csv with rows for gene sets and columns for 'Z' and 'contrast' routes)

## How to apply

Fit the TWO-SIGMA model (via twosigma/twosigma_custom, which fits a mixed-effects logistic regression for dropout plus a mixed-effects log-linear negative binomial model for the conditional mean using glmmTMB), then pass the fitted object to twosigmag with statistic='contrast' and a contrast matrix over regression coefficients; set index_test to the gene indices of the test set and set all_as_ref=TRUE (default) so all other genes form the reference set. twosigmag returns set-level results for each row of the contrast matrix. To validate the route, rerun the same test and reference sets with statistic='Z' (the Z-statistic from the mean model) and confirm the set-level p-values agree to floating-point tolerance, as demonstrated on the 10-gene simulated matrix where sets {6:10} (non-DE) and {1:5} (DE) give identical p-values under both routes. Even when not doing gene set testing, contrast testing can be obtained by setting index_test=list(c(1,2)) and all_as_ref=TRUE and reading the gene-level output. Do not enable the adhoc procedure here: geneset testing requires a common gene-level null, and mixing genes with and without random effects violates it.

## Related tools

- **twosigma (R package)** (Fits the two-component zero-inflated negative binomial mixed model whose fixed-effect coefficients the contrast matrix tests; its twosigmag function executes contrast-based gene/set testing) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Backend used by the twosigma package to fit the model specified in equation (2), supplying the coefficient estimates and standard errors consumed by the contrast test)
- **R** (Implementation environment; the twosigmag contrast workflow and simulated-data generation run in R (>= 3.6.3, package version 1.0.2)) — https://cran.r-project.org/web/packages/twosigma/

## Examples

```
res <- twosigmag(fits, index_test=list(6:10), all_as_ref=TRUE, statistic='contrast')
```

## Evaluation signals

- Set-level p-values from statistic='contrast' match those from statistic='Z' for the same test and reference sets to within floating-point tolerance (identical for both sets {6:10} and {1:5} in the 10-gene simulation)
- Directional sanity check: the non-DE control set {6:10} is non-significant while the DE set {1:5} is significant, mirroring the paper's joint null testing of H0: alpha1 = beta1 = 0
- twosigmag returns one set-level result per row of the supplied contrast matrix
- The adhoc procedure is disabled during gene set testing (a warning is printed if adhoc=TRUE is attempted with twosigmag-style testing)

## Limitations

- The adhoc procedure is not recommended for gene set testing because it gives some genes random effects and not others, so the common gene-level null hypothesis is no longer clearly met
- Direct contrast-matrix testing at the gene level in twosigma itself is 'ongoing' functionality; the workaround is to route contrasts through twosigmag
- lr.twosigma assumes the tested covariate is in both model components; integer covar_to_test positions are not checked to correspond to the same covariate in mean_covar and zi_covar
- Negative correlations are set to zero by default in twosigmag (allow_neg_corr=FALSE) for conservatism; most users should not modify this

## Evidence

- [readme] Gene-level statistics currently implemented include likelihood ratio, Z-statistic from the mean model, Stouffer's combination of the Z-statistics from the mean and ZI model, or a test of a custom contrast matrix.: "Gene-level statistics currently implemented include likelihood ratio, Z-statistic from the mean model, Stouffer's combination of the Z-statistics from the mean and ZI model, or a test of a custom"
- [readme] If a contrast matrix is input, set-level results are returned for each row of the contrast.: "If a contrast matrix is input, set-level results are returned for each row of the contrast."
- [readme] individuals can use the `twosigmag` function for TWO-SIGMA based Gene-Set Testing to test custom contrast matrices: "individuals can use the `twosigmag` function for TWO-SIGMA based Gene-Set Testing to test custom contrast matrices (even if not interested in gene set testing, simply set `index_test=list(c(1,2))`"
