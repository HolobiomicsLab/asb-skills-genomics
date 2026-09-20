---
name: mendelian-randomization-output-structure-validation
description: Use when a simulation pipeline consumes the return value of simmrd::generate_summary()
  or generate_individual() and must verify that all documented output components exist
  with the expected dimensions and IV classifications before feeding MR estimators.
when_to_use_negative:
- The input is raw individual-level genotype/phenotype data from generate_individual()
  that has not yet been passed through GWAS — the ten-component contract applies to
  the summary-level return list
- The goal is parameter configuration rather than output checking — use ?set_params
  for the full parameter list and defaults instead
- The MR data come from real GWAS summary statistics rather than the simmrd simulation
  engine, whose output schema this skill validates
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: simmrd
  role: R package whose generate_summary()/generate_individual() output list structure
    is being validated; installed via remotes::install_github('noahlorinczcomi/simmrd')
  repo: https://github.com/noahlorinczcomi/simmrd
- name: set_params
  role: Builds and validates the parameter list used to generate the output being
    checked; every argument has a sensible default
  repo: https://github.com/noahlorinczcomi/simmrd
- name: generate_summary
  role: Produces the summary-level simulated GWAS data list whose components (bx,
    by, RhoME, LDMatrix, theta, IVtype, etc.) must be verified
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Runtime environment in which names(), dim(), and label-vocabulary checks on
    the returned list are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:4f968f60477d4f16e66126d8b377f52322970db96ce38878f7f9921f1941bf9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - R
  - generate_summary
  - set_params
  - simmrd
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# mendelian-randomization-output-structure-validation

## Summary

Validates that simmrd simulation output objects carry the documented ten-component named-list structure (bx, bxse, by, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, and unstandardized variants) so downstream MR analyses consume correctly shaped data.

## When to use

Use when a simulation pipeline consumes the return value of simmrd::generate_summary() or generate_individual() and must verify that all documented output components exist with the expected dimensions and IV classifications before feeding MR estimators.

## When NOT to use

- The input is raw individual-level genotype/phenotype data from generate_individual() that has not yet been passed through GWAS — the ten-component contract applies to the summary-level return list
- The goal is parameter configuration rather than output checking — use ?set_params for the full parameter list and defaults instead
- The MR data come from real GWAS summary statistics rather than the simmrd simulation engine, whose output schema this skill validates

## Inputs

- R named list returned by simmrd::generate_summary() (or generate_individual())
- Parameter list produced by simmrd::set_params() or load_preset()

## Outputs

- Per-component presence flags for the ten documented list elements
- Structural validation report (e.g., component_check.txt) recording dimension checks for bx (m x p), by (m x 1), RhoME ((p+1) x (p+1)), and IVtype label vocabulary

## How to apply

Build the parameter list with set_params() (defaults are sensible, so call it with no arguments for a baseline check), then run generate_summary(params) which is the faster path suitable for most simulation studies. Enumerate names() on the returned list and confirm presence of the ten documented components: bx, bxse, by, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, and the unstandardized variants bx_unstd, bxse_unstd, by_unstd, byse_unstd. Validate structural invariants: bx is an m x p matrix of IV-exposure associations, by is an m x 1 vector of IV-outcome associations, RhoME is a (p+1) x (p+1) measurement-error correlation matrix, and each IV in IVtype is labeled 'valid', 'UHP', or 'CHP'. Rationale: catching misconfigured parameters early matters because set_params() validates inputs immediately and returns a clear error message rather than failing deep inside the simulation; structural checks at the output boundary are the complement to that input validation.

## Related tools

- **simmrd** (R package whose generate_summary()/generate_individual() output list structure is being validated; installed via remotes::install_github('noahlorinczcomi/simmrd')) — https://github.com/noahlorinczcomi/simmrd
- **set_params** (Builds and validates the parameter list used to generate the output being checked; every argument has a sensible default) — https://github.com/noahlorinczcomi/simmrd
- **generate_summary** (Produces the summary-level simulated GWAS data list whose components (bx, by, RhoME, LDMatrix, theta, IVtype, etc.) must be verified) — https://github.com/noahlorinczcomi/simmrd
- **R** (Runtime environment in which names(), dim(), and label-vocabulary checks on the returned list are executed)

## Examples

```
params <- set_params(); data <- generate_summary(params); stopifnot(all(c('bx','bxse','by','byse','RhoME','LDMatrix','LDhatMatrix','theta','IVtype') %in% names(data)))
```

## Evaluation signals

- names(data) contains all of bx, bxse, by, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype plus bx_unstd/bxse_unstd/by_unstd/byse_unstd
- dim(data$bx) is m x p and length(data$by) equals m (an m x 1 vector of IV-outcome associations)
- dim(data$RhoME) equals (p+1) x (p+1) for the measurement-error correlation matrix
- unique(data$IVtype) is a subset of {'valid', 'UHP', 'CHP'}
- set_params(params) raises a clear, immediate error rather than the simulation failing deep inside generate_summary() when inputs are misconfigured

## Limitations

- Both generate_summary() and generate_individual() generate data for a single simulation replicate, so structural validation covers one replicate at a time; multi-iteration Monte Carlo runs need the cli/ YAML-driven path
- No version number is stated for simmrd or R in the methods, so the output contract may drift across package versions
- The paper describes the ten-item enumeration (i)-(x) but some component definitions rely on the README table; verify against ?set_params documentation

## Evidence

- [readme] Both `generate_summary()` and `generate_individual()` return a named list: "Both `generate_summary()` and `generate_individual()` return a named list:"
- [readme] Output component table: [evidence span withheld — anchor status: not_found]
- [readme] IVtype classification: [evidence span withheld — anchor status: external_doc]
- [methods] generate_summary is the faster path: "`generate_summary()` is the faster path and is suitable for most simulation studies."
- [readme] Input validation rationale: [evidence span withheld — anchor status: external_doc]
- [readme] Single replicate caveat: "Both `simmrd::generate_individual()` and `simmrd::generate_summary()` generate simulated data for a **single** simulation replicate."
