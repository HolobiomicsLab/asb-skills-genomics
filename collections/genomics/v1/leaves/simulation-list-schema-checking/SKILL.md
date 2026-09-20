---
name: simulation-list-schema-checking
description: Use when a simulation study consumes simmrd::generate_summary() output
  and you must verify, before analysis, that every documented named-list component
  is present and structurally correct (e.g. bx is m x p, IVtype labels are 'valid'/'UHP'/'CHP').
when_to_use_negative:
- You need individual-level genotype and phenotype data with directly set confounding
  — the generate_individual() workflow is slower but lets you set confounding directly,
  so schema validation should target that path instead
- You want to compare simulation scenarios against the paper's named settings — use
  load_preset()/list_presets() to reproduce named scenarios rather than relying on
  set_params() defaults
- The goal is visualization of simulated GWAS results rather than schema verification
  — plot_simdata() is the appropriate step
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: simmrd
  role: Provides set_params() to build the default parameter list and generate_summary()
    whose named-list output is the object being schema-checked
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Runtime environment in which simmrd is installed and names(data)/dimension
    checks are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:252b634c1c28f9e389292512ca70635b3dd903f589772bc26d6a56e8c3cedca5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - R
  - set_params
  - simmrd
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# simulation-list-schema-checking

## Summary

Validates that a simmrd simulation replicate returned by generate_summary() (run under set_params() defaults) contains the full named-list schema — the ten documented components bx, bxse, by, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, and the unstandardized bx/by variants — so downstream MR method evaluation consumes structurally complete data.

## When to use

Use when a simulation study consumes simmrd::generate_summary() output and you must verify, before analysis, that every documented named-list component is present and structurally correct (e.g. bx is m x p, IVtype labels are 'valid'/'UHP'/'CHP').

## When NOT to use

- You need individual-level genotype and phenotype data with directly set confounding — the generate_individual() workflow is slower but lets you set confounding directly, so schema validation should target that path instead
- You want to compare simulation scenarios against the paper's named settings — use load_preset()/list_presets() to reproduce named scenarios rather than relying on set_params() defaults
- The goal is visualization of simulated GWAS results rather than schema verification — plot_simdata() is the appropriate step

## Inputs

- Parameter list built by simmrd::set_params() (e.g. defaults: sample_size_Xs=1e5, sample_size_Y=1e5, number_of_causal_SNPs=100, IV_Pvalue_threshold=5e-8, simtype='winners')
- R environment with simmrd installed from https://github.com/noahlorinczcomi/simmrd

## Outputs

- Named list returned by generate_summary() containing bx, bxse, by, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, bx_unstd, by_unstd, bxse_unstd, byse_unstd
- component_check.txt recording per-component presence flags and structural checks (dimensions of bx/by, IVtype value set)

## How to apply

Install simmrd from its GitHub repository and build the parameter list with params <- set_params(), overriding nothing since every argument has a sensible default. Run the summary-level simulation with data <- generate_summary(params), which the paper describes as the faster path suitable for most simulation studies. Enumerate names(data) and check presence of the ten documented components: bx, bxse, by, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, and the unstandardized bx_unstd/bxse_unstd/by_unstd/byse_unstd. Then validate structure: bx must be an m x p matrix of IV-exposure associations, by an m x 1 vector of IV-outcome associations, RhoME a (p+1) x (p+1) measurement-error correlation matrix, and IVtype must classify each IV as 'valid', 'UHP', or 'CHP'. Write per-component presence flags and structural checks to component_check.txt so the schema validation is auditable per replicate.

## Related tools

- **simmrd** (Provides set_params() to build the default parameter list and generate_summary() whose named-list output is the object being schema-checked) — https://github.com/noahlorinczcomi/simmrd
- **R** (Runtime environment in which simmrd is installed and names(data)/dimension checks are executed)

## Examples

```
params <- set_params(); data <- generate_summary(params); names(data)
```

## Evaluation signals

- names(data) contains exactly the ten documented components: bx, bxse, by, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, and the unstandardized bx_unstd/bxse_unstd/by_unstd/byse_unstd
- bx has m x p dimensions (m IVs, p exposures) and by is m x 1
- RhoME is a (p+1) x (p+1) measurement-error correlation matrix
- Every element of IVtype is one of 'valid', 'UHP', or 'CHP'
- component_check.txt records TRUE presence flags for all components and passes the dimension/value-set checks

## Limitations

- Both generate_individual() and generate_summary() generate simulated data for a single simulation replicate, so schema checks must be re-run per replicate or looped for Monte Carlo studies
- The associated paper (simmr-5.pdf) is a working draft that has not been peer-checked or reviewed
- No version number is stated for simmrd or R in the methods section, so schema fields could shift across package versions without a pinned install

## Evidence

- [readme] Both `generate_summary()` and `generate_individual()` return a named list:: "Both `generate_summary()` and `generate_individual()` return a named list:"
- [readme] bx is m x p matrix of IV-exposure associations: [evidence span withheld — anchor status: external_doc]
- [readme] IVtype per-IV classification: [evidence span withheld — anchor status: external_doc]
- [methods] set_params builds parameter list with defaults: "Call `set_params()` to build a parameter list. Only specify the values you want to change — every argument has a sensible default."
- [methods] generate_summary is the faster path: "`generate_summary()` is the faster path and is suitable for most simulation studies."
