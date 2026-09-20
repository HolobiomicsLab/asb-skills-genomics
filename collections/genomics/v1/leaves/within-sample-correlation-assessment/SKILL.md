---
name: within-sample-correlation-assessment
description: Use when scRNA-seq count data contain multiple cells per individual/sample
  and you must decide per gene whether random effects are needed in the mean and/or
  zero-inflation components before final differential expression inference.
when_to_use_negative:
- Genes already determined (or assumed) to require random effects under a fixed specification
  — set adhoc=FALSE and specify mean_re/zi_re directly instead of screening.
- 'Gene set testing with twosigmag: the README states the ad hoc procedure is not
  recommended because a common gene-level null is broken when some genes have random
  effects and others do not.'
- Data with a single sample/individual per condition (no replicate individuals), where
  a sample-level ANOVA or variance component is not estimable.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: twosigma
  role: R package implementing the ad hoc ANOVA screening (adhoc.twosigma) and variance-component
    LRT (test.vc.twosigma) on ZINB model residuals/fits
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Backend used by twosigma to fit the zero-inflated negative binomial mixed
    models (full and reduced) underlying the LRT
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:839142daca9456b9174d443267eea7fd1cf5b395b013b6e75374cf04f2b1d37c
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
yes_rate: 0.9230769230769231
n_claims: 13
---

# within-sample-correlation-assessment

## Summary

Determines, gene by gene, whether within-sample (individual-level) correlation in scRNA-seq counts requires random intercepts in a TWO-SIGMA zero-inflated negative binomial model, via the ad hoc Pearson-residual ANOVA F-test screen and/or formal likelihood-ratio tests of variance components using test.vc.twosigma.

## When to use

Use when scRNA-seq count data contain multiple cells per individual/sample and you must decide per gene whether random effects are needed in the mean and/or zero-inflation components before final differential expression inference.

## When NOT to use

- Genes already determined (or assumed) to require random effects under a fixed specification — set adhoc=FALSE and specify mean_re/zi_re directly instead of screening.
- Gene set testing with twosigmag: the README states the ad hoc procedure is not recommended because a common gene-level null is broken when some genes have random effects and others do not.
- Data with a single sample/individual per condition (no replicate individuals), where a sample-level ANOVA or variance component is not estimable.

## Inputs

- Raw single-cell count matrix of non-negative integer counts (cell j from sample i), no normalization applied
- Vector of individual-level IDs, one entry per cell
- mean_covar matrix (from model.matrix, no intercept) of covariates for the conditional mean model
- zi_covar matrix (from model.matrix, no intercept) of covariates for the zero-inflation model

## Outputs

- Per-gene ANOVA F-test p-value from adhoc.twosigma as a screening statistic for genes needing random effects
- Variance-component LRT statistics and p-values for three configurations: random effects in mean only, zero-inflation only, or both components (e.g., variance_component_lrt_results.csv)

## How to apply

First fit, per gene, a zero-inflated negative binomial model WITHOUT random effects (glmmTMB backend, drop-out probability and conditional-mean components per equation (2) minus random intercepts) using adhoc.twosigma(count, mean_covar, zi_covar, id), which computes ZINB Pearson residuals — with estimated drop-out probability, conditional mean, and overdispersion in the standard-error denominator — and regresses them on the individual ID via one-way ANOVA; the ANOVA F-test p-value screens genes most in need of random effects and dramatically reduces computation over many genes. For genes flagged as significant, formally evaluate the need for random effects by testing whether the corresponding variance component(s) equal zero: fit the full mixed model and reduced models (sigma_b=0, sigma_a=0, or both) via glmmTMB and compute LRT statistics -2*(l_H0 - l_Ha); single-component tests use a 50:50 mixture of chi-square(1 df) and a point mass at zero, while the joint two-component test conservatively uses chi-square with 2 df. This is built into the twosigma function (adhoc=TRUE by default), with mean_re/zi_re controlling which components carry random effects under the alternative.

## Related tools

- **twosigma** (R package implementing the ad hoc ANOVA screening (adhoc.twosigma) and variance-component LRT (test.vc.twosigma) on ZINB model residuals/fits) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Backend used by twosigma to fit the zero-inflated negative binomial mixed models (full and reduced) underlying the LRT)

## Examples

```
R: pvals <- adhoc.twosigma(count, mean_covar, zi_covar, id)
```

## Evaluation signals

- The ad hoc output contains one ANOVA F-test p-value row per gene in the input count matrix, with all p-values in [0,1].
- Genes whose ANOVA p-value is significant show residuals that are not exchangeable across samples and should be forwarded to the formal LRT step.
- Variance-component LRT output contains a statistic and p-value for each of the three configurations (mean-only, ZI-only, both), all p-values in [0,1] and all LRT statistics non-negative.
- When adhoc=TRUE is used, mean_re and zi_re are ignored and a warning is printed — confirm the intended random-effect path was actually taken.
- Single-component LRT p-values come from the 50:50 chi-square(1)/point-mass-at-zero mixture, while the joint test uses chi-square(2); check the reference distribution matches the configuration.

## Limitations

- The id argument is currently required by twosigma even if random effect prediction and the ad hoc method are not being used.
- The ad hoc procedure should not be used for gene set testing since a common gene-level null is not maintained across genes with differing random-effect specifications.
- Failing to include random effect terms can lead to underestimated standard errors and inflated type-I errors, with dramatic impacts on scientific conclusions as seen in the pancreatic islet analysis.
- For custom formulas in LRTs, users must supply valid lr.df degrees of freedom and ensure the formulas represent a valid likelihood ratio test.

## Evidence
