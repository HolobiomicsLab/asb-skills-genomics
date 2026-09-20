---
name: weak-instrument-f-statistic-evaluation
description: Use when simulating GWAS summary or individual-level data for MR method
  evaluation where instrument strength must be controlled or held at a specific first-stage
  F-statistic rather than arising from winner's-curse P-value selection.
when_to_use_negative:
- The goal is to study winner's-curse / P-value-selected instruments — simmrd's default
  simtype = 'winners' with IV_Pvalue_threshold = 5e-8 is the appropriate regime instead
- The task is applying MR to real observed GWAS data rather than generating simulated
  replicates for method evaluation
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: simmrd
  role: R package whose set_params(simtype='weak', fix_Fstatistic_at=...) and generate_summary()/generate_individual()
    functions produce weak-instrument simulation replicates
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Runtime environment for installing and running the simmrd package
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:8d60021bf9f2e2d348d135af5f39b6a7868937d84df5a899fb076a094af8e544
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - R
  - simmrd
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# weak-instrument-f-statistic-evaluation

## Summary

Generates MR simulation replicates with instruments whose strength is pinned to a target first-stage F-statistic (e.g. F = 10) using simmrd's weak-instrument mode, enabling evaluation of MR estimators under controlled weak-IV bias.

## When to use

Use when simulating GWAS summary or individual-level data for MR method evaluation where instrument strength must be controlled or held at a specific first-stage F-statistic rather than arising from winner's-curse P-value selection.

## When NOT to use

- The goal is to study winner's-curse / P-value-selected instruments — simmrd's default simtype = 'winners' with IV_Pvalue_threshold = 5e-8 is the appropriate regime instead
- The task is applying MR to real observed GWAS data rather than generating simulated replicates for method evaluation

## Inputs

- simmrd parameter list from set_params() with simtype = 'weak' and a fix_Fstatistic_at target (e.g. 10)
- Named preset parameters from load_preset('WEAK') or load_preset('UHP_CHP_WEAK') with n, snps, exposures, overlap overrides

## Outputs

- Named simulation replicate list with bx (m x p), by (m x 1), bxse, byse, LDMatrix, theta, and per-IV IVtype labels ('valid', 'UHP', 'CHP')
- Empirical mean first-stage F-statistic computed from bx and bxse for validation

## How to apply

Build a parameter list with simmrd's set_params(), setting simtype = 'weak' and fix_Fstatistic_at = 10 (or another target) instead of the default simtype = 'winners' P-value-selection regime; weak-IV bias generally becomes more severe as more IVs are used, so include a realistic number of instruments. For named paper scenarios, load_preset('UHP_CHP_WEAK') or load_preset('WEAK') reproduces the weak-instrument configurations, which can be combined with n, snps, exposures, and overlap overrides. Pass the parameter list to generate_summary() (fast, summary-level) or generate_individual() (individual-level, allows direct confounding specification) to obtain one replicate. Verify the simulation by computing the mean first-stage F-statistic from bx and bxse and confirming it is centred on the requested fix_Fstatistic_at value.

## Related tools

- **simmrd** (R package whose set_params(simtype='weak', fix_Fstatistic_at=...) and generate_summary()/generate_individual() functions produce weak-instrument simulation replicates) — https://github.com/noahlorinczcomi/simmrd
- **R** (Runtime environment for installing and running the simmrd package)

## Examples

```
params <- set_params(simtype = 'weak', fix_Fstatistic_at = 10); data <- generate_summary(params)
```

## Evaluation signals

- Mean first-stage F-statistic computed from bx and bxse is centred on the fix_Fstatistic_at target (e.g. 10)
- Output replicate is a named list containing correctly dimensioned bx (m x p matrix), by (m x 1), bxse, and byse
- IVtype vector class counts ('valid', 'UHP', 'CHP') match the preset's pleiotropy configuration (e.g. 20% of SNPs pleiotropic for single-bias scenarios, 10% each for UHP_CHP_WEAK)
- set_params() returned no validation error before generate_summary()/generate_individual() was invoked

## Limitations

- MVMR simulations using very few IVs may unrealistically reflect optimal performance because weak IV bias generally becomes more severe as more IVs are used
- Each generate_summary()/generate_individual() call produces a single simulation replicate, so multi-replicate Monte Carlo studies require looping or the CLI in the cli/ repo
- The associated paper draft is not peer-checked or reviewed, and no version number for simmrd is stated in the methods

## Evidence

- [other] For weak-instrument configurations (simtype = 'weak' with fix_Fstatistic_at = 10), compute the mean first-stage F-statistic from bx and bxse and confirm it is centred on the requested value.: [evidence span withheld — anchor status: not_found]
- [readme] simtype parameter doc: [evidence span withheld — anchor status: external_doc]
- [intro] MVMR weak IV count caveat: [evidence span withheld — anchor status: not_found]
- [readme] generate_single_replicate_doc: "Both `simmrd::generate_individual()` and `simmrd::generate_summary()` generate simulated data for a **single** simulation replicate."
- [readme] preset_weak_scenarios: [evidence span withheld — anchor status: external_doc]
