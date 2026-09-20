---
name: scrna-seq-count-simulation
description: Use when a pipeline needs a ground-truth simulated scRNA-seq count matrix
  (known DE vs non-DE genes, zero inflation, within-sample correlation) to validate
  gene-level or gene-set testing methods.
when_to_use_negative:
- The goal is analyzing real scRNA-seq data — twosigma's count_matrix input expects
  observed non-negative integer counts and performs no normalization, so simulated
  matrices are only for method validation
- The analysis requires the ad hoc random-effect screening (adhoc.twosigma) to be
  evaluated on empirical residual structure — simulated data with imposed variance
  components would not be an independent test
- A different count model is required (e.g. hurdle/log-scale models like MAST or SCDE-style
  simulations) — this function generates only zero-inflated negative binomial data
  with random effects
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: twosigma
  role: R package providing simulate_zero_inflated_nb_random_effect_data to generate
    the zero-inflated NB random-effect count matrix
  repo: https://github.com/edvanburen/twosigma
- name: R
  role: Runtime environment in which the twosigma package simulation function is called
    and the simulated object is assembled and saved
- name: glmmTMB
  role: Fitting engine for the zero-inflated negative binomial mixed models that the
    simulated data are subsequently evaluated against
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:1e8645570bcd6b8073e89b7cca3bed578497a9fa179d04014134c494af6ffd50
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# scRNA-seq-count-simulation

## Summary

Simulates zero-inflated negative binomial scRNA-seq count matrices with individual-level random effects using the simulate_zero_inflated_nb_random_effect_data function from the twosigma R package, so that TWO-SIGMA/TWO-SIGMA-G workflows can be exercised on data with known DE and non-DE gene labels and controlled drop-out and overdispersion.

## When to use

Use when a pipeline needs a ground-truth simulated scRNA-seq count matrix (known DE vs non-DE genes, zero inflation, within-sample correlation) to validate gene-level or gene-set testing methods.

## When NOT to use

- The goal is analyzing real scRNA-seq data — twosigma's count_matrix input expects observed non-negative integer counts and performs no normalization, so simulated matrices are only for method validation
- The analysis requires the ad hoc random-effect screening (adhoc.twosigma) to be evaluated on empirical residual structure — simulated data with imposed variance components would not be an independent test
- A different count model is required (e.g. hurdle/log-scale models like MAST or SCDE-style simulations) — this function generates only zero-inflated negative binomial data with random effects

## Inputs

- Simulation parameter settings (nind, ncellsper, sigma.a, sigma.b, alpha vector, beta vector, phi)
- Covariate vectors mirroring the pancreatic analysis: binary disease status (t2d_sim), age (age_sim), cellular detection rate (cdr_sim)
- twosigma R package (imports glmmTMB, pscl, multcomp, pbapply, parallel, doParallel)

## Outputs

- Per-gene simulated non-negative integer count matrix (e.g. 10 genes x 10,000 cells in the TWO-SIGMA-G example)
- Single data object with counts plus covariates, saved as simulated_zinb_data.rds
- Known DE (genes 1-5) vs non-DE (genes 6-10) gene labels for downstream validation

## How to apply

Load the twosigma R package and call simulate_zero_inflated_nb_random_effect_data per gene with the paper's example parameters: nind = 10, ncellsper = 1000 (10 individuals x 1000 cells each), sigma.a = sigma.b = 0.1, alpha = c(-1, 0, -0.5, -2) for the drop-out (zero-inflation) component, beta = c(2, 0, -0.1, 0.6) for the conditional mean component, and phi = 0.1 negative-binomial overdispersion. Construct covariates mirroring the pancreatic real-data analysis: a binary disease status indicator (t2d_sim), age (age_sim), and the cellular detection rate (cdr_sim). The function simulates drop-out probability from the fixed effects, covariates, and random effects, draws drop-out status from a Bernoulli distribution, and draws the conditional mean count from a negative binomial. Set the beta2 covariate coefficient non-zero for genes 1-5 (DE) and zero for genes 6-10 (non-DE) so downstream set-level testing has a positive and a negative control set. Assemble the per-gene counts and covariates into one object and save it (e.g. as .rds) for repeated use.

## Related tools

- **twosigma** (R package providing simulate_zero_inflated_nb_random_effect_data to generate the zero-inflated NB random-effect count matrix) — https://github.com/edvanburen/twosigma
- **R** (Runtime environment in which the twosigma package simulation function is called and the simulated object is assembled and saved)
- **glmmTMB** (Fitting engine for the zero-inflated negative binomial mixed models that the simulated data are subsequently evaluated against)

## Examples

```
library(twosigma); sim <- simulate_zero_inflated_nb_random_effect_data(nind=10, ncellsper=1000, sigma.a=0.1, sigma.b=0.1, alpha=c(-1,0,-0.5,-2), beta=c(2,0,-0.1,0.6), phi=0.1)
```

## Evaluation signals

- Saved object has the expected dimensions (10,000 genes x 10,000 cells = 10 individuals x 1000 cells each in the full simulation; a 10-gene matrix in the TWO-SIGMA-G example)
- All simulated values are non-negative integers
- Simulated data exhibit zero inflation consistent with the drop-out probability component specified by alpha and sigma.a/sigma.b
- Downstream TWO-SIGMA-G testing on genes 6-10 (non-DE control set) yields a non-significant set-level p-value while genes 1-5 (DE set) yields a significant p-value
- Parameter reproduction is exact: sigma.a = sigma.b = 0.1, alpha = (-1, 0, -.5, -2), beta = (2, 0, -.1, .6), phi = .1 as in the paper's example

## Limitations

- The function draws counts from a zero-inflated negative binomial with random effects only; it cannot simulate the log-scale hurdle behavior that competing methods like MAST assume
- The paper provides no software version or package installation details in the article body itself; minimum R version is >= 3.6.3 per the README, and imported-package minimums (multcomp >= 1.4-13, pscl >= 1.5.5, pbapply >= 1.4.0) must be met
- Simulated zero inflation depends on the specified drop-out probability component plus random effects, so mis-specified alpha/sigma values will silently change the zero-inflation rate

## Evidence
