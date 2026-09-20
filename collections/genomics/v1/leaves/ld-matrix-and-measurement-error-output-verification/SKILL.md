---
name: ld-matrix-and-measurement-error-output-verification
description: Use when a simmrd simulation has been run and you must confirm the returned
  list contains all documented components (bx, bxse, by, byse, RhoME, LDMatrix, LDhatMatrix,
  theta, IVtype, unstandardized variants) with correct dimensions before feeding them
  to MR methods.
when_to_use_negative:
- The goal is to evaluate the statistical performance of MR estimators (bias, power,
  Type I error) rather than to verify the structural completeness of the simulation
  output object.
- Input data are real GWAS summary statistics rather than simulated lists produced
  by simmrd's generate_summary() or generate_individual().
- You need to modify simulation parameters — use set_params() validation directly
  instead of post-hoc output checking.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: simmrd
  role: R package whose generate_summary()/generate_individual() output list (bx,
    bxse, by, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, unstandardized variants)
    is verified by this skill
  repo: https://github.com/noahlorinczcomi/simmrd
- name: set_params
  role: Builds and validates the parameter list whose values (e.g. true_causal_effects,
    number_of_exposures) are cross-checked against theta and IVtype in the output
  repo: https://github.com/noahlorinczcomi/simmrd
- name: generate_summary
  role: Faster summary-level GWAS simulation function that produces the output list
    being structurally verified
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Environment in which names(data), dimension checks, and per-component flags
    are scripted
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:af45e01910888014600c401969906ad4b361440d32b838a347bbe09a760e4059
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
yes_rate: 0.5
n_claims: 8
---

# ld-matrix-and-measurement-error-output-verification

## Summary

A verification skill for simmrd simulation output that checks the structural and semantic integrity of the named list returned by generate_summary() or generate_individual(), including the true and estimated LD correlation matrices (LDMatrix, LDhatMatrix), the (p+1)x(p+1) measurement-error correlation matrix RhoME, theta, and IVtype classifications.

## When to use

Use when a simmrd simulation has been run and you must confirm the returned list contains all documented components (bx, bxse, by, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, unstandardized variants) with correct dimensions before feeding them to MR methods.

## When NOT to use

- The goal is to evaluate the statistical performance of MR estimators (bias, power, Type I error) rather than to verify the structural completeness of the simulation output object.
- Input data are real GWAS summary statistics rather than simulated lists produced by simmrd's generate_summary() or generate_individual().
- You need to modify simulation parameters — use set_params() validation directly instead of post-hoc output checking.

## Inputs

- Named list returned by simmrd::generate_summary() or generate_individual()
- Parameter list from set_params() or load_preset() used to generate the data

## Outputs

- Per-component presence flags for the ten documented output elements
- Dimension checks for bx (m x p), by (m x 1), RhoME ((p+1) x (p+1)), LDMatrix and LDhatMatrix (m x m)
- IVtype classification counts ('valid', 'UHP', 'CHP')
- component_check.txt audit file

## How to apply

Run set_params() with defaults (or overrides) and generate data via generate_summary() — the faster summary-level path — or generate_individual() when confounding must be set directly. Enumerate names(data) and confirm the ten documented elements are present: bx, bxse, by, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, and the unstandardized variants bx_unstd/bxse_unstd/by_unstd/byse_unstd. Validate dimensions: bx is an m x p matrix of IV–exposure associations, by is an m x 1 vector of IV–outcome associations, RhoME is (p+1) x (p+1), and LDMatrix/LDhatMatrix are m x m correlation matrices among IVs (LDhatMatrix reflects imprecise LD estimation). Check IVtype labels each instrument as 'valid', 'UHP', or 'CHP', and confirm theta matches the true_causal_effects passed to set_params(). Write per-component presence flags and structural checks to an output file (e.g. component_check.txt) for auditability.

## Related tools

- **simmrd** (R package whose generate_summary()/generate_individual() output list (bx, bxse, by, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, unstandardized variants) is verified by this skill) — https://github.com/noahlorinczcomi/simmrd
- **set_params** (Builds and validates the parameter list whose values (e.g. true_causal_effects, number_of_exposures) are cross-checked against theta and IVtype in the output) — https://github.com/noahlorinczcomi/simmrd
- **generate_summary** (Faster summary-level GWAS simulation function that produces the output list being structurally verified) — https://github.com/noahlorinczcomi/simmrd
- **R** (Environment in which names(data), dimension checks, and per-component flags are scripted)

## Examples

```
params <- set_params(); data <- generate_summary(params); stopifnot(all(c('bx','bxse','by','byse','RhoME','LDMatrix','LDhatMatrix','theta','IVtype') %in% names(data)))
```

## Evaluation signals

- names(data) contains exactly the documented elements: bx, bxse, by, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, bx_unstd, bxse_unstd, by_unstd, byse_unstd
- dim(data$bx) == c(m, p) and length(data$by) == m, where m is the number of IVs and p the number of exposures
- dim(data$RhoME) == c(p+1, p+1) and data$LDMatrix / data$LDhatMatrix are m x m matrices with unit diagonal (correlation matrices)
- data$theta equals the true_causal_effects vector supplied to set_params()
- unique(data$IVtype) is a subset of {'valid', 'UHP', 'CHP'}

## Limitations

- Both generate_summary() and generate_individual() generate data for a single simulation replicate, so verification applies per replicate only.
- LDhatMatrix is an estimated LD correlation matrix and may be imprecise relative to the true LDMatrix when LD estimation error is enabled in the simulation scenario.
- No version number is stated for simmrd or R in the paper's methods, so reproducibility of the exact output schema across versions is not guaranteed.

## Evidence

- [readme] Both generate_summary() and generate_individual() return a named list: "Both `generate_summary()` and `generate_individual()` return a named list:"
- [readme] LD matrices and measurement error definitions: [evidence span withheld — anchor status: not_found]
- [readme] IVtype classification: [evidence span withheld — anchor status: external_doc]
- [methods] generate_summary is the faster path: "`generate_summary()` is the faster path and is suitable for most simulation studies."
- [readme] single replicate caveat: "Both `simmrd::generate_individual()` and `simmrd::generate_summary()` generate simulated data for a **single** simulation replicate."
