---
name: simulation-based-count-data-analysis
description: Use when validating a differential expression method on scRNA-seq count
  data with known ground truth — e.g. testing whether a covariate such as t2d_sim
  is associated with expression while controlling type-I error and estimating power.
when_to_use_negative:
- The goal is differential expression on real (not simulated) data where a two-group
  DE tool like MAST, SCDE, scDD, or DESingle suffices without ground-truth evaluation.
- Analysis requires log-scale hurdle modeling of log-transformed counts rather than
  direct modeling of observed counts via zero-inflation.
- Gene set (competitive) testing is the goal and the ad hoc procedure is being considered
  — TWO-SIGMA-G gene set testing requires a common gene-level null, so random-effect
  inclusion must be uniform across genes.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: twosigma
  role: R package implementing the two-component ZINB mixed model fit (twosigma, twosigma_custom)
    and the likelihood ratio test (lr.twosigma) used on simulated counts
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Underlying GLMM fitter employed by twosigma to fit the zero-inflated negative
    binomial model in equation (2)
- name: R
  role: Runtime environment for the simulation, model fitting, and likelihood ratio
    testing workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:9e3dd135fd3fa8ea3b5ee5c0b5853c093459cc6065d27bdd8519caa4db4c2211
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edvanburen/twosigma
  tools:
  - R
  - glmmTMB
  - twosigma
  - twosigma (R package)
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# simulation-based-count-data-analysis

## Summary

Generates and analyzes simulated single-cell count data with covariates like binary disease status, age, and cellular detection rate (CDR) to evaluate a two-component zero-inflated negative binomial mixed model (TWO-SIGMA), including type-I error and power via the likelihood ratio test on the joint null H0: alpha1 = beta1 = 0.

## When to use

Use when validating a differential expression method on scRNA-seq count data with known ground truth — e.g. testing whether a covariate such as t2d_sim is associated with expression while controlling type-I error and estimating power.

## When NOT to use

- The goal is differential expression on real (not simulated) data where a two-group DE tool like MAST, SCDE, scDD, or DESingle suffices without ground-truth evaluation.
- Analysis requires log-scale hurdle modeling of log-transformed counts rather than direct modeling of observed counts via zero-inflation.
- Gene set (competitive) testing is the goal and the ad hoc procedure is being considered — TWO-SIGMA-G gene set testing requires a common gene-level null, so random-effect inclusion must be uniform across genes.

## Inputs

- simulated gene-by-cell count matrix of non-negative integer counts (no normalization)
- covariate design: binary disease status (e.g. t2d_sim), age, and cellular detection rate (CDR)
- individual-level ID vector (length equal to number of cells) for random effects
- mean_covar and zi_covar matrices (from model.matrix, without intercept)

## Outputs

- per-gene table with mean-component coefficient, Z value, Wald p-value, LR statistic, and LR p-value (e.g. twosigma_lr_results.csv)
- type-I error and power estimates under the joint null H0: alpha1 = beta1 = 0
- empirical significance thresholds at the quantile associated with 1 minus the nominal level

## How to apply

Simulate covariates mirroring the real pancreatic islet analysis — a binary disease status, age, and the cellular detection rate (CDR) — and generate a count matrix per gene under a zero-inflated negative binomial mixed model with sample-specific random intercepts ai and bi (logit link for drop-out, log link for the conditional mean). Fit the model per gene with twosigma()/twosigma_custom(), which rely on glmmTMB for GLMM fitting, and by default apply the ad hoc procedure: a one-way ANOVA of Pearson's residuals (from a ZINB fit without random effects) on the sample indicator to decide whether random intercepts are needed. Evaluate type-I error and power using the likelihood ratio test on the joint null H0: alpha1 = beta1 = 0 via lr.twosigma with covar_to_test set to the disease covariate, and extract mean-component coefficient estimates, Z values, and Wald p-values from the glmmTMB summaries. Assemble a per-gene table (gene, mean coefficient, Z value, Wald p-value, LR statistic, LR p-value) and, for true power estimation, compute empirical significance thresholds from null simulations defined as the test statistic value at the quantile associated with 1 minus the nominal level.

## Related tools

- **twosigma** (R package implementing the two-component ZINB mixed model fit (twosigma, twosigma_custom) and the likelihood ratio test (lr.twosigma) used on simulated counts) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Underlying GLMM fitter employed by twosigma to fit the zero-inflated negative binomial model in equation (2))
- **R** (Runtime environment for the simulation, model fitting, and likelihood ratio testing workflow)

## Examples

```
lr.fit <- lr.twosigma(count_matrix, mean_covar, zi_covar, covar_to_test = "t2d_sim", mean_re = TRUE, zi_re = TRUE, id = id)
```

## Evaluation signals

- Output table contains exactly one row per simulated gene with a finite LR statistic.
- Wald p-values and LR p-values all fall within [0, 1].
- Under null simulations, per-gene rejection rates match the nominal type-I error level after computing the empirical significance threshold.
- The ad hoc ANOVA F-test p-value from Pearson's residuals identifies genes for which random effects are needed, consistent with the likelihood ratio test of variance components (test.vc.twosigma).
- Fitted objects are glmmTMB class models and summary(fit)$coefficients$cond contains rows for the tested covariate (e.g. 't2d_sim').

## Limitations

- Ignoring random effects when within-sample correlation is present can lead to underestimated standard errors and inflated type-I errors.
- lr.twosigma assumes the tested variable appears in both model components and that the zero-inflation component contains more than an intercept; other cases require lr.twosigma_custom with a user-specified lr.df.
- The ad hoc procedure should not be used for gene set testing because genes with and without random effects do not share a common gene-level null.
- Custom formulas passed to twosigma_custom / lr.twosigma_custom are used directly; the user must ensure they represent a valid likelihood ratio test.

## Evidence
