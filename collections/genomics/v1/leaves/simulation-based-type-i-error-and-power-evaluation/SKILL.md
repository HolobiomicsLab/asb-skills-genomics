---
name: simulation-based-type-i-error-and-power-evaluation
description: Use when assessing whether a gene-level DE test (likelihood ratio, mean-model
  Z, or Stouffer Z_combined) preserves nominal type-I error and yields interpretable
  power on simulated scRNA-seq counts with within-sample correlation.
when_to_use_negative:
- When the goal is single-gene real-data inference rather than calibrated error-rate/power
  assessment across many simulated replicates
- When mixing genes with and without random effects under a common null — the ad hoc
  screening procedure is not recommended for gene set testing because a common gene-level
  null requirement is violated
- When the data are not integer counts with dropout/overdispersion (the model assumes
  zero-inflated negative binomial observed counts, no normalization, no log transformation)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_3308
tools:
- name: twosigma
  role: R package providing simulate_zero_inflated_nb_random_effect_data for simulation
    and twosigma/twosigma_custom/lr.twosigma for fitting and joint LRT of the zero-inflated
    NB mixed model; Stouffer combination is a built-in gene-level statistic for twosigmag
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Fitting engine for the TWO-SIGMA model; component-wise fixed-effect estimates,
    SEs, and z values are extracted from its fitted objects to compute Z_mean, Z_ZI,
    and the Stouffer statistic
- name: R
  role: Environment in which the simulation, per-gene model fitting, Z-statistic extraction
    (lapply over fits$fit), and error-rate tabulation are performed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:63ad5fc176b14729075b9c91cafcff99c7ed7e6f5d068b10b64f58accc01ddd1
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
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# simulation-based-type-i-error-and-power-evaluation

## Summary

A skill for evaluating the type-I error control and power of gene-level association tests (e.g., TWO-SIGMA's LRT, mean-model Z-statistic, or Stouffer combination of mean- and zero-inflation Z-statistics) on simulated zero-inflated negative binomial scRNA-seq data with random effects.

## When to use

Use when assessing whether a gene-level DE test (likelihood ratio, mean-model Z, or Stouffer Z_combined) preserves nominal type-I error and yields interpretable power on simulated scRNA-seq counts with within-sample correlation.

## When NOT to use

- When the goal is single-gene real-data inference rather than calibrated error-rate/power assessment across many simulated replicates
- When mixing genes with and without random effects under a common null — the ad hoc screening procedure is not recommended for gene set testing because a common gene-level null requirement is violated
- When the data are not integer counts with dropout/overdispersion (the model assumes zero-inflated negative binomial observed counts, no normalization, no log transformation)

## Inputs

- Simulated count matrix from simulate_zero_inflated_nb_random_effect_data (~10,000 independent genes, zero-inflated NB with random effects)
- Covariate design: binary disease status, age, cellular detection rate (CDR)
- Per-cell individual-level ID vector for random intercepts
- Fitted glmmTMB objects from twosigma()/twosigma_custom() per gene

## Outputs

- Per-gene table (stouffer_z_table.csv) with columns gene, Z_mean, Z_ZI, Z_combined, and p-value
- Empirical type-I error rates at the nominal 0.05 significance level (Bonferroni 1/10000 as check)
- Empirical significance thresholds and power estimates per method and simulation setting
- LRT results for the joint null H0: alpha1 = beta1 = 0

## How to apply

Simulate ~10,000 independent genes per scenario with simulate_zero_inflated_nb_random_effect_data from the twosigma R package, using a binary disease status, age, and the cellular detection rate (CDR) as covariates to mirror the pancreatic islet analysis. Fit the TWO-SIGMA zero-inflated negative binomial mixed model per gene with glmmTMB, with sample-specific random intercepts in both the zero-inflation (logit link) and conditional mean (log link) components. Extract the disease-status fixed-effect estimate and SE from each component, compute Z_mean = beta1_hat/SE and Z_ZI = alpha1_hat/SE, and combine via Stouffer: Z_combined = (Z_mean + Z_ZI)/sqrt(2) with a two-sided standard-normal p-value. Evaluate type-I error and power using the likelihood ratio test on the joint null H0: alpha1 = beta1 = 0 at the nominal 0.05 rate; for true power estimation, compare against the empirical significance threshold defined as the test statistic value at the quantile associated with 1 minus alpha under the null. Assemble a per-gene table (e.g., stouffer_z_table.csv) with columns gene, Z_mean, Z_ZI, Z_combined, p-value; the Bonferroni threshold of 1/10000 is available as a stricter check.

## Related tools

- **twosigma** (R package providing simulate_zero_inflated_nb_random_effect_data for simulation and twosigma/twosigma_custom/lr.twosigma for fitting and joint LRT of the zero-inflated NB mixed model; Stouffer combination is a built-in gene-level statistic for twosigmag) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Fitting engine for the TWO-SIGMA model; component-wise fixed-effect estimates, SEs, and z values are extracted from its fitted objects to compute Z_mean, Z_ZI, and the Stouffer statistic)
- **R** (Environment in which the simulation, per-gene model fitting, Z-statistic extraction (lapply over fits$fit), and error-rate tabulation are performed)

## Examples

```
lr.twosigma(count_matrix, mean_covar, zi_covar, covar_to_test, mean_re = TRUE, zi_re = TRUE, adhoc=TRUE)
```

## Evaluation signals

- Empirical type-I error for the joint H0: alpha1 = beta1 = 0 must be close to the nominal 0.05 rate (with the stricter Bonferroni threshold 1/10000 available as a check)
- Z_combined must equal (Z_mean + Z_ZI)/sqrt(2) exactly for every gene in the output table
- Each per-gene Z statistic must match the ratio of the fixed-effect estimate to its SE from the corresponding glmmTMB component summary
- Empirical significance thresholds for true power must be derived from the null-scenario test statistic quantile at 1 - alpha, not from the theoretical null alone
- Output table must contain all four required columns (gene, Z_mean, Z_ZI, Z_combined, p-value) with no missing entries for genes whose fits returned non-glmmTMB objects (which the extraction guards convert to NA)

## Limitations

- Ignoring random effects when within-sample correlation exists leads to underestimated standard errors and inflated type-I errors; random-effect inclusion must match the correlation structure
- The empirical threshold must be computed per method and per simulation setting under the null; using a single theoretical threshold can misstate true power
- Fits may fail or return character summaries for some genes; the extraction functions return NA in those cases, so downstream power estimates must account for missing values

## Evidence
