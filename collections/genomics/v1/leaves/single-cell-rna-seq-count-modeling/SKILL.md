---
name: single-cell-rna-seq-count-modeling
description: Use when you have a raw single-cell count matrix (cells nested within
  individuals) and need gene-level covariate association inference that controls for
  drop-out (zero inflation), overdispersion, and within-sample correlation.
when_to_use_negative:
- 'The ad hoc procedure is not recommended for use in gene set testing: when some
  genes have random effects and others do not, the common gene-level null required
  by twosigmag is not clearly met.'
- Input is already normalized or log-transformed data — twosigma models observed counts
  directly (like a ZINB, unlike MAST's log-scale hurdle model) and performs no normalization.
- Counts are non-integer or contain missing values, or you need custom random-effect
  specifications without verifying the lme4-style formulas yourself — twosigma_custom
  uses formulas directly and it is the user's responsibility to ensure they yield
  a valid likelihood ratio test with a supplied lr.df.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: twosigma (R package)
  role: 'Implements the TWO-SIGMA ZINB mixed model: twosigma/twosigma_custom for fitting,
    adhoc.twosigma for residual-based screening, lr.twosigma and test.vc.twosigma
    for LRT-based testing'
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Backend used by the twosigma package to fit the zero-inflated negative binomial
    model with random intercepts in both components
- name: R
  role: Runtime environment; package requires R >= 3.6.3 (version 1.0.2), with minimum
    versions for multcomp, pscl, pbapply, and doParallel
  repo: https://github.com/edvanburen/twosigma
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_006
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:39a665cb859b2120aafcb3c9989169c002ab3e2ef1baa01c6c2a4f9d1411678e
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# single-cell-rna-seq-count-modeling

## Summary

Fits the TWO-SIGMA zero-inflated negative binomial mixed model (via the twosigma R package with glmmTMB backend) directly on raw scRNA-seq counts, with random intercepts in both the drop-out (logit) and conditional-mean (log) components, to support fixed-effect association testing such as disease status while accounting for within-sample correlation.

## When to use

Use when you have a raw single-cell count matrix (cells nested within individuals) and need gene-level covariate association inference that controls for drop-out (zero inflation), overdispersion, and within-sample correlation.

## When NOT to use

- The ad hoc procedure is not recommended for use in gene set testing: when some genes have random effects and others do not, the common gene-level null required by twosigmag is not clearly met.
- Input is already normalized or log-transformed data — twosigma models observed counts directly (like a ZINB, unlike MAST's log-scale hurdle model) and performs no normalization.
- Counts are non-integer or contain missing values, or you need custom random-effect specifications without verifying the lme4-style formulas yourself — twosigma_custom uses formulas directly and it is the user's responsibility to ensure they yield a valid likelihood ratio test with a supplied lr.df.

## Inputs

- count_matrix: matrix of non-negative integer scRNA-seq counts (rows = genes, columns = cells), no missing values, no normalization required
- mean_covar: covariate matrix for the conditional mean model (e.g. from model.matrix), without an intercept column
- zi_covar: covariate matrix for the zero-inflation (drop-out) model (e.g. binary disease status, age, CDR), without an intercept column
- id: vector of individual/sample IDs, one entry per cell

## Outputs

- Per-gene glmmTMB fit objects (fits$fit) for the TWO-SIGMA ZINB mixed model
- Per-gene fixed-effect estimates, standard errors, and Z-statistics from the conditional mean and zero-inflation components
- Per-gene ANOVA F-test p-values from the ad hoc screening on Pearson residuals
- Likelihood-ratio p-values from lr.twosigma / test.vc.twosigma for fixed effects and variance components
- Stouffer-combined gene-level statistics Z_combined = (Z_mean + Z_ZI)/sqrt(2) usable as twosigmag gene-level statistics

## How to apply

Call the twosigma workhorse function on a matrix of non-negative integer counts (no normalization or log transformation is performed), passing mean_covar and zi_covar design matrices built with model.matrix WITHOUT intercept terms and the individual-level id vector (required even when no random effects are used). By default adhoc=TRUE: for each gene a zero-inflated negative binomial model without random effects is fit, Pearson residuals (with the estimated drop-out probability, conditional mean, and overdispersion in the standard-error denominator) are regressed on individual ID via one-way ANOVA, and the ANOVA F-test p-value screens genes that most need random effects; genes with significant p-values are then fit with random intercepts or forwarded to the likelihood-ratio variance-component test (test.vc.twosigma). Alternatively set adhoc=FALSE and specify mean_re/zi_re, or use twosigma_custom with lme4-style formulas such as mean_form=count~covars+(1|id), zi_form=~covars+(1|id). For fixed-effect inference, use lr.twosigma (testing a covariate present in both components) or extract component-wise Z-statistics and combine them via Stouffer: Z_combined = (Z_mean + Z_ZI)/sqrt(2), as used for set-level enrichment in twosigmag. In the paper's simulations this specification was fit with binary disease status, age, and cellular detection rate (CDR) covariates, and joint association was tested via the likelihood ratio test of H0: alpha1 = beta1 = 0.

## Related tools

- **twosigma (R package)** (Implements the TWO-SIGMA ZINB mixed model: twosigma/twosigma_custom for fitting, adhoc.twosigma for residual-based screening, lr.twosigma and test.vc.twosigma for LRT-based testing) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Backend used by the twosigma package to fit the zero-inflated negative binomial model with random intercepts in both components)
- **R** (Runtime environment; package requires R >= 3.6.3 (version 1.0.2), with minimum versions for multcomp, pscl, pbapply, and doParallel) — https://github.com/edvanburen/twosigma

## Examples

```
library(twosigma); fits <- twosigma(count_matrix, mean_covar=mean_covar_matrix, zi_covar=zi_covar_matrix, mean_re=TRUE, zi_re=TRUE, id=id, adhoc=FALSE)
```

## Evaluation signals

- All covariate matrices (mean_covar, zi_covar) must lack an intercept column and have rows matching the number of cells; the id vector must have one entry per cell.
- Under the default adhoc=TRUE setting, every gene should receive an ANOVA F-test p-value from regressing Pearson residuals on individual ID; significant p-values flag genes for the full random-effects fit or the likelihood-ratio variance-component test.
- Failing to include random effects when within-sample correlation is present should reproduce the paper's pathology: underestimated standard errors and inflated type-I error (type-I error and power were evaluated via the LRT on H0: alpha1 = beta1 = 0 at nominal 0.05, with a Bonferroni threshold of 1/10000 for 10,000 genes).
- The Stouffer statistic for each gene must equal (Z_mean + Z_ZI)/sqrt(2), combining the disease-status Z from the conditional mean component with the Z from the zero-inflation component.
- If custom formulas are passed to twosigma_custom, verify the response is named 'count' (e.g. mean_form = count ~ 1) and that lr.df is a non-negative integer matching the LRT degrees of freedom.

## Limitations

- By default the ad hoc procedure is used to decide random effects; if adhoc=TRUE, user-specified mean_re and zi_re are ignored and a warning is printed.
- The ad hoc ANOVA screening is a screening statistic only, and it is explicitly not recommended for gene set testing because a common gene-level null is not guaranteed when random-effect specifications differ across genes.
- The README's calc_Stouffer example sums the same mean-model Z twice rather than combining mean-model and zero-inflation Z-statistics, so users should not copy that snippet verbatim when constructing the Stouffer combination.
- No normalization is performed and counts are modeled directly; the paper also discusses the pragmatic choice of excluding random effects from the zero-inflation component (e.g. as in the real pancreatic islet data analysis), so component-wise random-effect inclusion is a modeling decision.

## Evidence
