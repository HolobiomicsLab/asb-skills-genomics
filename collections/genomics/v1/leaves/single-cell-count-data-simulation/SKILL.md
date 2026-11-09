---
name: single-cell-count-data-simulation
description: Use when you need synthetic scRNA-seq count matrices with known variance-component
  structure (random intercepts in mean and/or zero-inflation components) to evaluate
  type-I error or power of mixed-effects DE tests.
when_to_use_negative:
- The goal is analysis of a real scRNA-seq dataset — simulation is only for benchmarking
  type-I error/power, not for producing biological findings
- You need normalized expression values or log-transformed data — the twosigma count_matrix
  input must be non-negative integer counts with no normalization performed
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: twosigma
  role: R package providing simulate_zero_inflated_nb_random_effect_data for generating
    synthetic scRNA-seq counts with random effects
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Fits the zero-inflated negative binomial mixed model (equation 2) to the simulated
    counts, supplying log-likelihoods for LRTs
- name: R
  role: Runtime environment for the twosigma package (R >= 3.6.3)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:7bbe750429585a056bd8089be7dbea35bf2f0841804243f6afd937a9495436c1
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
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# single-cell-count-data-simulation

## Summary

Simulates zero-inflated negative binomial single-cell count data with random effects using the simulate_zero_inflated_nb_random_effect_data function of the twosigma R package, to benchmark model-based association tests such as TWO-SIGMA variance-component likelihood ratio tests.

## When to use

Use when you need synthetic scRNA-seq count matrices with known variance-component structure (random intercepts in mean and/or zero-inflation components) to evaluate type-I error or power of mixed-effects DE tests.

## When NOT to use

- The goal is analysis of a real scRNA-seq dataset — simulation is only for benchmarking type-I error/power, not for producing biological findings
- You need normalized expression values or log-transformed data — the twosigma count_matrix input must be non-negative integer counts with no normalization performed

## Inputs

- Simulation design parameters (number of genes, number of individuals/cells per individual)
- Sample-level and cell-level covariate specifications: binary disease status, age, cellular detection rate (CDR)
- Random-effect variance settings (sigma_a for the zero-inflation component, sigma_b for the mean component)
- R with the twosigma package (>= 3.6.3, version 1.0.2) and glmmTMB

## Outputs

- Simulated gene-by-cell integer count matrix of non-negative counts with dropout zeros
- Per-gene simulated model fits (glmmTMB objects) under full and reduced specifications
- Derived statistics such as variance-component LRT statistics and p-values (e.g. variance_component_lrt_results.csv)

## How to apply

Call twosigma's simulate_zero_inflated_nb_random_effect_data to generate counts for a large panel of independent genes (e.g. 10,000) under a zero-inflated negative binomial model with random intercepts. Set simulated covariates to mirror the real analysis being mimicked — for the pancreatic islet study this means a binary disease status, age, and the cellular detection rate (CDR). Generate both null and alternative configurations (e.g. random effects present/absent in mean_re and zi_re components, disease effect present/absent) so downstream LRT statistics and empirical significance thresholds can be computed. Fit candidate models with glmmTMB via twosigma/twosigma_custom on the simulated counts and compare log-likelihoods between full and reduced models. Rationale: simulating on the observed-count scale preserves zero inflation and overdispersion, letting the simulation isolate whether random-effect terms are needed to control type-I error when cells within a sample are even moderately correlated.

## Related tools

- **twosigma** (R package providing simulate_zero_inflated_nb_random_effect_data for generating synthetic scRNA-seq counts with random effects) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Fits the zero-inflated negative binomial mixed model (equation 2) to the simulated counts, supplying log-likelihoods for LRTs)
- **R** (Runtime environment for the twosigma package (R >= 3.6.3))

## Examples

```
library(twosigma); sim <- simulate_zero_inflated_nb_random_effect_data(); fits <- twosigma(sim$count_matrix, mean_covar, zi_covar, id=sim$id, mean_re=TRUE, zi_re=TRUE, adhoc=FALSE)
```

## Evaluation signals

- The output results file exists and contains a row with a statistic and p-value for each of the three random-effect configurations (mean-only, zero-inflation-only, both)
- All simulated p-values lie within [0,1] and all LRT statistics (-2*(l_H0 - l_Ha)) are non-negative
- Simulated counts are non-negative integers on the raw count scale (no normalization applied), matching the twosigma count_matrix contract
- Under null simulations, empirical type-I error at the nominal level can be estimated against an empirical significance threshold defined at the 1-alpha quantile of the null statistic distribution

## Limitations

- Simulated covariates only mirror the pancreatic analysis design (binary disease status, age, CDR); other biological contexts require redesigning the covariate structure
- The ad hoc procedure is not recommended for gene set testing because genes with and without random effects do not share a common gene-level null; simulation-based evaluations must account for this
- No software version or package installation details beyond the twosigma GitHub/CRAN source are given, so reproducibility depends on pinning package versions

## Evidence
