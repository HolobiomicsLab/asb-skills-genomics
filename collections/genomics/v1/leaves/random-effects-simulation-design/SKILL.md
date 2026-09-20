---
name: random-effects-simulation-design
description: Use when you need simulated zero-inflated negative-binomial count data
  with known random-effect variance components (sigma.a, sigma.b) to benchmark mixed-effects
  differential expression methods like TWO-SIGMA or MAST.
when_to_use_negative:
- Input is real scRNA-seq count data rather than a simulation to be generated — use
  the twosigma fitting functions instead
- The goal is gene set testing (TWO-SIGMA-G) rather than generating benchmark data
  with controlled variance components
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: twosigma
  role: R package providing simulate_zero_inflated_nb_random_effect_data to generate
    the zero-inflated negative-binomial random-effect simulation
  repo: https://github.com/edvanburen/twosigma
- name: R
  role: Runtime environment in which the twosigma package (R >= 3.6.3) is used to
    run the simulation and save the .rds object
  repo: https://github.com/edvanburen/twosigma
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:4dbb94c97dd47aade034bb25f2a516586a42a7d6c927b9a92a47b2ee679bd995
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edvanburen/twosigma
  tools:
  - R
  - twosigma
  - twosigma (R package)
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# random-effects-simulation-design

## Summary

Replicates the TWO-SIGMA paper's simulation of zero-inflated negative-binomial scRNA-seq data with within-sample (random-effect) correlation, using the twosigma R package's simulate_zero_inflated_nb_random_effect_data function to generate benchmark data for type-I error and power evaluation.

## When to use

Use when you need simulated zero-inflated negative-binomial count data with known random-effect variance components (sigma.a, sigma.b) to benchmark mixed-effects differential expression methods like TWO-SIGMA or MAST.

## When NOT to use

- Input is real scRNA-seq count data rather than a simulation to be generated — use the twosigma fitting functions instead
- The goal is gene set testing (TWO-SIGMA-G) rather than generating benchmark data with controlled variance components

## Inputs

- Simulation parameter settings: nind = 10, ncellsper = 1000, sigma.a = sigma.b = 0.1, alpha = c(-1, 0, -0.5, -2), beta = c(2, 0, -0.1, 0.6), phi = 0.1
- Covariate vectors mirroring the pancreatic real-data analysis: binary disease status (t2d_sim), age (age_sim), and cellular detection rate (cdr_sim)

## Outputs

- simulated_zinb_data.rds: an R data object holding simulated counts for 10,000 genes x 10,000 cells (10 individuals x 1000 cells each)
- Per-gene zero-inflated negative-binomial counts with known drop-out status and random-effect structure

## How to apply

Load the twosigma R package (version >= 1.0.2, R >= 3.6.3) and set the paper's exact parameters: nind = 10, ncellsper = 1000, sigma.a = sigma.b = 0.1, alpha = c(-1, 0, -0.5, -2) for the zero-inflation (drop-out) component, beta = c(2, 0, -0.1, 0.6) for the conditional mean component, and overdispersion phi = 0.1. Construct covariates mirroring the pancreatic islet real-data analysis: a binary disease status indicator (t2d_sim), age (age_sim), and the cellular detection rate (cdr_sim). Call simulate_zero_inflated_nb_random_effect_data for each of 10,000 genes; the function simulates the drop-out probability from the fixed effects and covariates plus random effects, draws drop-out status from a Bernoulli distribution, and draws the conditional mean count from a negative binomial with phi = 0.1. Assemble the per-gene counts and covariates into one data object and save it as simulated_zinb_data.rds. The known variance components allow later evaluation of whether including random effects is necessary to preserve type-I error under the likelihood ratio test of H0: alpha1 = beta1 = 0.

## Related tools

- **twosigma** (R package providing simulate_zero_inflated_nb_random_effect_data to generate the zero-inflated negative-binomial random-effect simulation) — https://github.com/edvanburen/twosigma
- **R** (Runtime environment in which the twosigma package (R >= 3.6.3) is used to run the simulation and save the .rds object) — https://github.com/edvanburen/twosigma

## Examples

```
sim <- simulate_zero_inflated_nb_random_effect_data(nind=10, ncellsper=1000, sigma.a=0.1, sigma.b=0.1, alpha=c(-1,0,-.5,-2), beta=c(2,0,-.1,.6), phi=.1)
```

## Evaluation signals

- Saved object has exactly 10,000 genes x 10,000 cells (10 individuals x 1000 cells each)
- All simulated counts are non-negative integers
- Simulated data exhibit zero inflation consistent with the drop-out probability implied by alpha = c(-1, 0, -0.5, -2) and covariates t2d_sim, age_sim, cdr_sim
- Empirical overdispersion reflects phi = 0.1 and within-sample correlation reflects sigma.a = sigma.b = 0.1
- Under the null (H0: alpha1 = beta1 = 0), the simulated data reproduce the paper's type-I error behavior for the likelihood ratio test

## Limitations

- Simulation scale is fixed by the paper's example (10 individuals x 1000 cells, 10,000 genes); other designs require re-specifying parameters
- The ad hoc ANOVA procedure for random-effect selection is a screening heuristic and is not recommended for gene set testing, since a common gene-level null may not hold when only some genes carry random effects

## Evidence

- [other] The example simulation sets sigma.a = sigma.b = 0.1, alpha = (-1, 0, -.5, -2), beta = (2, 0, -.1, .6), and phi = .1.: [evidence span withheld — anchor status: not_found]
