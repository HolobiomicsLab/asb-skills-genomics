---
name: simulation-parameter-configuration
description: Use when you need an exact replicate of the TWO-SIGMA paper's simulated
  ZINB random-effect dataset for benchmarking, type-I error/power studies, or validating
  the twosigma package against the published simulation design.
when_to_use_negative:
- When the goal is to fit TWO-SIGMA to real scRNA-seq counts rather than generate
  a synthetic benchmark dataset — use the twosigma()/lr.twosigma() fitting functions
  instead
- When you need a simulation design with different covariates, sample sizes, or effect
  sizes than the paper's example — those require newly chosen parameters, not this
  fixed configuration
- When you need normalized or transformed expression values — the simulator produces
  non-negative integer counts only (no normalization is done)
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: twosigma (R package)
  role: Provides simulate_zero_inflated_nb_random_effect_data, the function that generates
    the zero-inflated negative binomial random-effect data under the configured parameters
  repo: https://github.com/edvanburen/twosigma
- name: R
  role: Runtime environment in which the twosigma package is installed and the simulation
    is executed (R >= 3.6.3)
  repo: https://github.com/edvanburen/twosigma
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:d3dd10e4daec0309e59b47c2859c910bd888f3339cfef7cfd6be9faad423bce8
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# simulation-parameter-configuration

## Summary

Reproduces the TWO-SIGMA paper's example zero-inflated negative binomial random-effect simulation by calling simulate_zero_inflated_nb_random_effect_data with the paper's exact parameters (nind=10, ncellsper=1000, sigma.a=sigma.b=0.1, alpha=(-1,0,-0.5,-2), beta=(2,0,-0.1,0.6), phi=0.1) over 10,000 genes.

## When to use

Use when you need an exact replicate of the TWO-SIGMA paper's simulated ZINB random-effect dataset for benchmarking, type-I error/power studies, or validating the twosigma package against the published simulation design.

## When NOT to use

- When the goal is to fit TWO-SIGMA to real scRNA-seq counts rather than generate a synthetic benchmark dataset — use the twosigma()/lr.twosigma() fitting functions instead
- When you need a simulation design with different covariates, sample sizes, or effect sizes than the paper's example — those require newly chosen parameters, not this fixed configuration
- When you need normalized or transformed expression values — the simulator produces non-negative integer counts only (no normalization is done)

## Inputs

- Simulation parameter set: nind=10, ncellsper=1000, sigma.a=sigma.b=0.1, alpha=(-1,0,-0.5,-2), beta=(2,0,-0.1,0.6), phi=0.1
- Covariate vectors mirroring the pancreatic islet real data analysis: t2d_sim (binary disease status), age_sim (age), cdr_sim (cellular detection rate)

## Outputs

- simulated_zinb_data.rds: an R object of per-gene simulated counts for 10,000 genes x 10,000 cells (10 individuals x 1000 cells each) with non-negative integer counts and the associated covariates

## How to apply

Load the twosigma R package (>= 3.6.3 R, installable from CRAN or GitHub edvanburen/twosigma) and set the paper's example parameters: nind=10 individuals with ncellsper=1000 cells each (10,000 cells total), sigma.a=sigma.b=0.1, alpha=c(-1,0,-0.5,-2), beta=c(2,0,-0.1,0.6), and phi=0.1 overdispersion. Construct covariates mirroring the pancreatic real-data analysis — a binary disease status indicator (t2d_sim), age (age_sim), and the cellular detection rate (cdr_sim). For each of the 10,000 independent genes, call simulate_zero_inflated_nb_random_effect_data, which simulates a drop-out probability from the fixed effects, covariates, and random effects, draws drop-out status from a Bernoulli distribution, and draws the conditional mean count from a negative binomial with phi=0.1. Assemble the per-gene counts and covariates into a single object and save it as simulated_zinb_data.rds.

## Related tools

- **twosigma (R package)** (Provides simulate_zero_inflated_nb_random_effect_data, the function that generates the zero-inflated negative binomial random-effect data under the configured parameters) — https://github.com/edvanburen/twosigma
- **R** (Runtime environment in which the twosigma package is installed and the simulation is executed (R >= 3.6.3)) — https://github.com/edvanburen/twosigma

## Examples

```
sim <- simulate_zero_inflated_nb_random_effect_data(nind=10, ncellsper=1000, sigma.a=0.1, sigma.b=0.1, alpha=c(-1,0,-0.5,-2), beta=c(2,0,-0.1,0.6), phi=0.1); saveRDS(sim, 'simulated_zinb_data.rds')
```

## Evaluation signals

- The saved simulated_zinb_data.rds object has dimensions 10,000 genes x 10,000 cells (10 individuals x 1000 cells each)
- All simulated values are non-negative integers (counts), consistent with twosigma's expected input
- The simulated data exhibits zero inflation consistent with the drop-out probability component built from alpha=(-1,0,-0.5,-2), beta=(2,0,-0.1,0.6) and covariates
- Count overdispersion matches the specified phi = 0.1 negative-binomial component
- Covariates present in the object match the pancreatic real-data design: binary disease status, age, and cellular detection rate (CDR)

## Limitations

- The configuration reproduces one fixed example; the paper notes no software version details are given in the discussion section, so package version tracking must come from the README (version 1.0.2, R >= 3.6.3)
- Simulation validity depends on the two-component model structure (drop-out logistic regression + log-linear negative binomial mean) matching the paper's parameterization; deviations require re-deriving parameters
- Zero-inflation component behavior beyond the example is discussed only in supplementary material (section 6 and supplementary figure S10), which is not available in this context

## Evidence

- [methods] The example simulation sets sigma.a = sigma.b = 0.1, alpha = (-1, 0, -.5, -2), beta = (2, 0, -.1, .6), and phi = .1.: [evidence span withheld — anchor status: not_found]
- [methods] simulates a drop-out probability from the fixed effects and covariates plus random effects, draws drop-out status from a Bernoulli distribution: [evidence span withheld — anchor status: not_found]
