---
name: simulation-based-scrna-seq-data-generation
description: Use when a null or power study needs synthetic single-cell count data
  with excess zeros, overdispersion, and within-sample correlation, e.g. to benchmark
  ZI-model specifications or type-I error of regression-based DE methods.
when_to_use_negative:
- The goal is differential expression on real, observed scRNA-seq data rather than
  a controlled simulation with known generating parameters
- The analysis requires normalized or log-transformed expression rather than raw non-negative
  integer counts (twosigma applies no normalization)
- Gene-set-level (TWO-SIGMA-G) enrichment testing is the target rather than gene-level
  model fitting or method benchmarking
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: twosigma
  role: R package whose simulate_zero_inflated_nb_random_effect_data function generates
    zero-inflated NB count data with random effects for the simulation study
  repo: https://github.com/edvanburen/twosigma
- name: R
  role: Runtime environment for the twosigma package (R >= 3.6.3) used to run the
    simulation and downstream glmmTMB fits
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:2ececef4de37131cf6d333d07b3effb65a02821e5f2f4845572eaa772954128d
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# simulation-based-scrna-seq-data-generation

## Summary

Generate synthetic scRNA-seq count matrices under a zero-inflated negative binomial model with random effects using the simulate_zero_inflated_nb_random_effect_data function in the twosigma R package, with covariates mirroring the pancreatic islet analysis (binary disease status, age, cellular detection rate).

## When to use

Use when a null or power study needs synthetic single-cell count data with excess zeros, overdispersion, and within-sample correlation, e.g. to benchmark ZI-model specifications or type-I error of regression-based DE methods.

## When NOT to use

- The goal is differential expression on real, observed scRNA-seq data rather than a controlled simulation with known generating parameters
- The analysis requires normalized or log-transformed expression rather than raw non-negative integer counts (twosigma applies no normalization)
- Gene-set-level (TWO-SIGMA-G) enrichment testing is the target rather than gene-level model fitting or method benchmarking

## Inputs

- Simulation settings for the zero-inflated negative binomial model (gene number, e.g. 10,000 independent genes; fixed-effect coefficients; dispersion; zero-inflation parameters; random-effect variance components)
- Covariate design mirroring the pancreatic islet analysis: binary disease status, age, and cellular detection rate (CDR)
- Vector of individual-level IDs defining within-sample correlation structure

## Outputs

- Simulated count matrix of non-negative integer counts (genes x cells), e.g. 10,000 independent genes
- Simulated covariate matrix (disease status, age, CDR) aligned to cells
- Per-cell sample/individual ID vector usable as the id argument in twosigma

## How to apply

Call simulate_zero_inflated_nb_random_effect_data from the twosigma R package (installed from CRAN or edvanburen/twosigma) to generate a count matrix of non-negative integer counts — e.g. on the order of 10,000 independent genes as used in the paper's simulations — together with simulated covariates. The simulated covariates should mirror the pancreatic real data analysis and include a binary disease status, age, and the cellular detection rate (CDR). Because the generative model is a zero-inflated negative binomial with a point mass at zero for dropout and random-effect structure, the output supports fitting both conditional-mean and drop-out components downstream (e.g. with glmmTMB via twosigma or twosigma_custom). Retain the covariate matrix and per-cell sample IDs so that fitted models can be specified with mean_covar and zi_covar matrices lacking intercept terms, as required by the twosigma interface. Use the resulting simulated truth (known generating coefficients) to assess estimation or type-I error/power of downstream models.

## Related tools

- **twosigma** (R package whose simulate_zero_inflated_nb_random_effect_data function generates zero-inflated NB count data with random effects for the simulation study) — https://github.com/edvanburen/twosigma
- **R** (Runtime environment for the twosigma package (R >= 3.6.3) used to run the simulation and downstream glmmTMB fits)

## Examples

```
library(twosigma); sim <- simulate_zero_inflated_nb_random_effect_data()  # generates simulated ZINB counts with disease status, age, CDR covariates
```

## Evaluation signals

- Output count matrix contains only non-negative integer counts with no missing values
- Simulated covariates contain the expected three columns (binary disease status, age, CDR) with rows matching the number of cells
- Marginal zero fraction exceeds a standard NB without dropout would produce, consistent with the zero-inflation (point-mass-at-zero) component
- Per-gene fits (e.g. via glmmTMB) converge with finite coefficient estimates, indicating the simulated data are compatible with the ZINB model class
- Simulated covariates reproduce the structure of the pancreatic real data analysis (binary disease status, age, cellular detection rate)

## Limitations

- No normalization is performed by the twosigma interface; counts must be used as generated
- Simulated covariates mirror one specific design (disease status, age, CDR); other study designs require adapted simulation settings not documented in the provided material
- The paper notes the robustness details of the zero-inflated negative binomial mixed-effects model 'are not discussed in as much detail as we can here' and defers ZI-component discussion to the supplement

## Evidence
