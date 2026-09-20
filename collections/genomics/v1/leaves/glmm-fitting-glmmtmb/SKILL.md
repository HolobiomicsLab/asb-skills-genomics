---
name: glmm-fitting-glmmtmb
description: Use when per-gene differential expression or covariate association testing
  on raw single-cell UMI counts must account for zero inflation, overdispersion, and
  within-sample correlation (cells nested in individuals).
when_to_use_negative:
- Input is already normalized/transformed (e.g., log-scale counts) — TWO-SIGMA models
  observed raw counts directly and performs no normalization
- The goal is gene set testing with the ad hoc procedure active — README warns the
  adhoc procedure is not recommended for twosigmag because a common gene-level null
  is required when some genes have random effects and others do not
- The zero-inflation component contains only an intercept and joint fixed-effect testing
  is intended — lr.twosigma assumes the tested covariate is in both components; use
  lr.twosigma_custom with custom formulas and an explicit lr.df instead
edam_operation: http://edamontology.org/operation_3657
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: twosigma
  role: R package wrapping glmmTMB to fit the TWO-SIGMA zero-inflated negative binomial
    mixed model per gene; workhorse functions twosigma, twosigma_custom, lr.twosigma,
    adhoc.twosigma, test.vc.twosigma
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Underlying fitting engine for the two-component mixed model (mixed logistic
    zero-inflation and log-link negative binomial mean components)
- name: R
  role: Runtime environment; twosigma version 1.0.2 requires R >= 3.6.3 with minimum
    imported versions multcomp >= 1.4-13, pscl >= 1.5.5, pbapply >= 1.4.0, doParallel
    >= 1.0.15
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:0c3021d39d8a54975b3416dd433f4c7756974abbef5e991eb492b0032013147e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edvanburen/twosigma
  tools:
  - R
  - glmmTMB
  - twosigma
  - twosigma (R package)
  edam_operation: http://edamontology.org/operation_3657
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# glmm-fitting-glmmtmb

## Summary

Fits the TWO-SIGMA zero-inflated negative binomial mixed model per gene via glmmTMB, with optional sample-specific random intercepts in both the zero-inflation (logit link) and conditional mean (log link) components, to enable fixed-effect association testing in scRNA-seq data.

## When to use

Use when per-gene differential expression or covariate association testing on raw single-cell UMI counts must account for zero inflation, overdispersion, and within-sample correlation (cells nested in individuals).

## When NOT to use

- Input is already normalized/transformed (e.g., log-scale counts) — TWO-SIGMA models observed raw counts directly and performs no normalization
- The goal is gene set testing with the ad hoc procedure active — README warns the adhoc procedure is not recommended for twosigmag because a common gene-level null is required when some genes have random effects and others do not
- The zero-inflation component contains only an intercept and joint fixed-effect testing is intended — lr.twosigma assumes the tested covariate is in both components; use lr.twosigma_custom with custom formulas and an explicit lr.df instead

## Inputs

- Per-gene count matrix of non-negative integer UMI counts (no normalization, no missing values)
- mean_covar: covariate matrix for the conditional mean model without intercept (e.g., model.matrix output with disease status, age, CDR)
- zi_covar: covariate matrix for the zero-inflation model without intercept
- id: vector of individual-level IDs, one entry per cell (length equal to number of cells)

## Outputs

- A glmmTMB fitted object per gene (returned in fits$fit), containing conditional-mean and zero-inflation coefficient estimates, standard errors, z-values, and p-values
- Fixed-effect Z-statistics and p-values extractable via summary()$coefficients$cond for downstream Stouffer combination or gene-set testing
- ANOVA F-test p-values from adhoc.twosigma screening for genes that need random effects
- Likelihood ratio test p-values and degrees of freedom when using lr.twosigma / test.vc.twosigma

## How to apply

For each gene, fit the TWO-SIGMA specification (equation 2) with glmmTMB as employed by the twosigma R package: a mixed-effects logistic regression for the dropout probability and a mixed-effects log-linear negative binomial regression for the conditional mean, using raw non-negative integer counts without normalization. Supply mean and zero-inflation covariate matrices (from model.matrix, without intercept terms) and a vector of individual-level IDs used for random effect prediction. By default the package's ad hoc procedure decides whether random intercepts are needed: it fits a zero-inflated negative binomial model without random effects, regresses Pearson's residuals on the sample indicator via one-way ANOVA, and uses the ANOVA F-test p-value to screen genes needing random effects; set adhoc=FALSE to force mean_re/zi_re manually (e.g., both TRUE as in the paper's simulations with binary disease status, age, and CDR covariates). Fixed effects of interest (e.g., disease status) can then be tested via Wald Z-statistics extracted from summary(fit)$coefficients$cond/$zi or via the likelihood ratio test of the joint null H0: alpha1 = beta1 = 0 using lr.twosigma or test.vc.twosigma for variance components.

## Related tools

- **twosigma** (R package wrapping glmmTMB to fit the TWO-SIGMA zero-inflated negative binomial mixed model per gene; workhorse functions twosigma, twosigma_custom, lr.twosigma, adhoc.twosigma, test.vc.twosigma) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Underlying fitting engine for the two-component mixed model (mixed logistic zero-inflation and log-link negative binomial mean components))
- **R** (Runtime environment; twosigma version 1.0.2 requires R >= 3.6.3 with minimum imported versions multcomp >= 1.4-13, pscl >= 1.5.5, pbapply >= 1.4.0, doParallel >= 1.0.15)

## Examples

```
fits <- twosigma(count_matrix, mean_covar = mean_covar_matrix, zi_covar = zi_covar_matrix, mean_re = TRUE, zi_re = TRUE, id = id, adhoc = FALSE)
```

## Evaluation signals

- Each fitted object is of class glmmTMB (or a character 'try-error' marker if fitting failed) — check class(fits$fit[[i]]) before extracting coefficients
- Ignoring random effects when within-sample correlation is present should inflate type-I error; the fitted model with random intercepts should preserve nominal 0.05 type-I error on the joint null H0: alpha1 = beta1 = 0
- Empirical significance thresholds can be computed under the null as the test statistic quantile at 1 minus alpha for true power estimation
- Coefficients extracted must match the tested covariate name in BOTH components (e.g., 't2d_sim' in $coefficients$cond and $coefficients$zi) when forming component-wise Z statistics
- Bonferroni threshold of 1/10000 for 10,000 simulated genes can serve as a conservative significance check against the nominal rate

## Limitations

- Failing to include random effect terms when cells are even moderately correlated leads to underestimated standard errors and inflated type-I errors
- Users must ensure custom formulas passed to twosigma_custom operate as expected and represent a valid likelihood ratio test with a user-supplied lr.df (non-negative integer); there is no check that an integer covar_to_test index refers to the same covariate in both mean_covar and zi_covar
- Ad hoc random-effect selection is not recommended for gene set testing since mixed random-effect specifications may break the common gene-level null
- No normalization is performed on the count matrix; missing values are not allowed

## Evidence
