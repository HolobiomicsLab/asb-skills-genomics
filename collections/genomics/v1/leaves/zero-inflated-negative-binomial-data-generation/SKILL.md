---
name: zero-inflated-negative-binomial-data-generation
description: Use when building simulated scRNA-seq benchmarks that require excess
  zeros plus overdispersed counts with within-sample correlation, mirroring the TWO-SIGMA
  simulation design (10 individuals x 1000 cells, ZINB with random effects).
when_to_use_negative:
- Input is already real scRNA-seq count data that needs model fitting rather than
  simulated data generation
- The analysis requires hurdle/log-scale models (e.g. MAST-style) rather than zero-inflated
  negative binomial counts on the observed scale
- Only bulk RNA-seq data without drop-out or within-sample correlation is being simulated
  — a simpler NB model suffices
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: twosigma
  role: R package providing simulate_zero_inflated_nb_random_effect_data to generate
    ZINB random-effect count data; fits via glmmTMB downstream
  repo: https://github.com/edvanburen/twosigma
- name: R
  role: Language environment (>= 3.6.3) in which the twosigma package is installed
    and the simulation loop is executed
  repo: https://github.com/edvanburen/twosigma
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:caffcc865adf54930ea7ea3a69c35ef3a6fb5ee74dff8af8abd43868b1f15ad6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edvanburen/twosigma
  tools:
  - R
  - twosigma
  - twosigma (R package)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# zero-inflated-negative-binomial-data-generation

## Summary

Reproduces the TWO-SIGMA paper's simulation of zero-inflated negative binomial scRNA-seq count data with random effects, using the simulate_zero_inflated_nb_random_effect_data function from the twosigma R package to generate per-gene counts under specified drop-out and overdispersion parameters.

## When to use

Use when building simulated scRNA-seq benchmarks that require excess zeros plus overdispersed counts with within-sample correlation, mirroring the TWO-SIGMA simulation design (10 individuals x 1000 cells, ZINB with random effects).

## When NOT to use

- Input is already real scRNA-seq count data that needs model fitting rather than simulated data generation
- The analysis requires hurdle/log-scale models (e.g. MAST-style) rather than zero-inflated negative binomial counts on the observed scale
- Only bulk RNA-seq data without drop-out or within-sample correlation is being simulated — a simpler NB model suffices

## Inputs

- Simulation parameter set: nind = 10, ncellsper = 1000, sigma.a = sigma.b = 0.1, alpha = c(-1, 0, -0.5, -2), beta = c(2, 0, -0.1, 0.6), phi = 0.1
- Covariate vectors mirroring the pancreatic real data analysis: binary disease status (t2d_sim), age (age_sim), cellular detection rate (cdr_sim)
- Number of independent genes to simulate (10,000 in the paper's example)

## Outputs

- simulated_zinb_data.rds: RDS object holding a 10,000 genes x 10,000 cells count matrix (10 individuals x 1000 cells each) of non-negative integer counts with zero inflation, plus the covariates

## How to apply

Install the twosigma R package and set the paper's exact simulation parameters: nind = 10, ncellsper = 1000, sigma.a = sigma.b = 0.1, alpha = c(-1, 0, -0.5, -2), beta = c(2, 0, -0.1, 0.6), phi = 0.1. Construct covariates mirroring the pancreatic real data analysis: a binary disease status indicator (t2d_sim), age (age_sim), and the cellular detection rate (cdr_sim). For each of 10,000 independent genes, call simulate_zero_inflated_nb_random_effect_data, which simulates a drop-out probability from the fixed effects plus covariates and random effects, draws drop-out status from a Bernoulli distribution, and draws the conditional mean count from the negative binomial with overdispersion phi = 0.1. Assemble the per-gene simulated counts and covariates into a single data object and save as simulated_zinb_data.rds. The random-effect component is essential because the paper shows failing to include random effect terms can have dramatic impacts on scientific conclusions.

## Related tools

- **twosigma** (R package providing simulate_zero_inflated_nb_random_effect_data to generate ZINB random-effect count data; fits via glmmTMB downstream) — https://github.com/edvanburen/twosigma
- **R** (Language environment (>= 3.6.3) in which the twosigma package is installed and the simulation loop is executed) — https://github.com/edvanburen/twosigma

## Examples

```
library(twosigma); sim <- simulate_zero_inflated_nb_random_effect_data(nind=10, ncellsper=1000, sigma.a=0.1, sigma.b=0.1, alpha=c(-1,0,-0.5,-2), beta=c(2,0,-0.1,0.6), phi=0.1); saveRDS(sim, 'simulated_zinb_data.rds')
```

## Evaluation signals

- Saved object has exactly 10,000 genes x 10,000 cells (10 individuals x 1000 cells each)
- Count matrix contains only non-negative integer values with no missing values
- Simulated data exhibits zero inflation consistent with the drop-out probability component driven by alpha = c(-1, 0, -0.5, -2)
- Overdispersion of counts is consistent with the negative binomial component at phi = 0.1
- Covariates match the pancreatic-real-data design: binary disease status, age, and cellular detection rate (CDR)

## Limitations

- Random-effect variance components are fixed at sigma.a = sigma.b = 0.1; other settings are not covered by this reproduction
- The simulation parameterizes counts directly on the observed scale (no normalization is done), so downstream methods must not log-transform before the TWO-SIGMA fit
- Simulation draws independent genes; inter-gene correlation is not modeled

## Evidence
