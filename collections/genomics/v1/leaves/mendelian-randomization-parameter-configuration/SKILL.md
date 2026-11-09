---
name: mendelian-randomization-parameter-configuration
description: Use when setting up MR simulation scenarios in simmrd that require explicit
  parameter choices such as GWAS sample sizes, SNP counts, sample overlap, weak-instrument
  regimes, or UHP/CHP horizontal pleiotropy before data generation.
when_to_use_negative:
- The goal is to fit or evaluate MR estimators on real GWAS summary statistics rather
  than to generate simulated data — simmrd is a simulation tool for evaluating MR
  method performance.
- You need multiple simulation replicates driven from a YAML parameter file at scale
  — the CLI in the cli/ repo with pixi is designed for multi-iteration Monte Carlo
  runs, whereas set_params()/generate_*() produce a single replicate per call.
- The simulation scenario matches a paper preset exactly and needs no customization
  beyond set_params()/load_preset() defaults with no explicit configuration required.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0622
tools:
- name: simmrd
  role: R package whose set_params(), load_preset(), and list_presets() functions
    build and validate the simulation parameter configuration before generate_summary()
    or generate_individual() is called
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Runtime environment in which simmrd is installed (via remotes::install_github)
    and the parameter configuration is executed
- name: pixi
  role: Manages the R/Python environment for the command-line CLI in the cli/ repo,
    which runs simulations from YAML parameter files as an alternative configuration
    path
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:9d17bb64533a7a16fe782a5cd6d77c4527d6c66d4466c4bbfda02d12f49b94ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - R
  - pixi
  - simmrd
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mendelian-randomization-parameter-configuration

## Summary

Configure simulation parameters for Mendelian randomization data generation using simmrd's set_params(), which builds a validated parameter list where only non-default values (e.g., type, sample_size_Xs, number_of_causal_SNPs, prop_gwas_overlap_Xs_and_Y, pleiotropy settings) need to be specified before passing it to generate_summary() or generate_individual().

## When to use

Use when setting up MR simulation scenarios in simmrd that require explicit parameter choices such as GWAS sample sizes, SNP counts, sample overlap, weak-instrument regimes, or UHP/CHP horizontal pleiotropy before data generation.

## When NOT to use

- The goal is to fit or evaluate MR estimators on real GWAS summary statistics rather than to generate simulated data — simmrd is a simulation tool for evaluating MR method performance.
- You need multiple simulation replicates driven from a YAML parameter file at scale — the CLI in the cli/ repo with pixi is designed for multi-iteration Monte Carlo runs, whereas set_params()/generate_*() produce a single replicate per call.
- The simulation scenario matches a paper preset exactly and needs no customization beyond set_params()/load_preset() defaults with no explicit configuration required.

## Inputs

- R parameter values for set_params(): type ("summary" or "individual"), sample_size_Xs (default 1e5), sample_size_Y (default 1e5), number_of_exposures (default 1), number_of_causal_SNPs (default 100), prop_gwas_overlap_Xs_and_Y (default 1), true_causal_effects (default 0.3), number_of_UHP_causal_SNPs, number_of_CHP_causal_SNPs, simtype ("winners" or "weak"), fix_Fstatistic_at, IV_Pvalue_threshold (default 5e-8)
- Preset name and dimension overrides for load_preset() (bias, n, snps, exposures, overlap)

## Outputs

- Validated parameter list (R list) suitable for passing to generate_summary() or generate_individual()
- Simulated data named list from generate_summary()/generate_individual() containing bx, by, bxse, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, and unstandardized variants

## How to apply

Load the simmrd R package with library(simmrd) and call set_params(), specifying only the values to change from defaults — e.g., type = "individual", sample_size_Xs = 1e6, sample_size_Y = 1e6, prop_gwas_overlap_Xs_and_Y = 0, and number_of_causal_SNPs = 500 for a large non-overlapping individual-level scenario. Alternatively, use load_preset() (with list_presets() to enumerate options) to reproduce named scenarios from the paper, choosing from bias types (none, UHP, CHP, UHP_CHP, UHP_CHP_WEAK, WEAK), n (3e4, 1e5), snps (100, 500), exposures (1, 3), and overlap (full, none); preset pleiotropy parameters scale automatically with snps (20% pleiotropic SNPs for single-bias scenarios, 10% each for combined). set_params() validates inputs immediately and returns a clear error message on misconfiguration rather than failing deep inside the simulation, so treat validation errors as a signal to correct arguments at the configuration stage. Pass the resulting parameter list to generate_summary() for the faster summary-level path or generate_individual() for individual-level genotype/phenotype data where confounding is set directly; note generate_individual() is slower, taking up to ~2 minutes at the largest reported setting (1M nonoverlapping samples, 500 causal SNPs).

## Related tools

- **simmrd** (R package whose set_params(), load_preset(), and list_presets() functions build and validate the simulation parameter configuration before generate_summary() or generate_individual() is called) — https://github.com/noahlorinczcomi/simmrd
- **R** (Runtime environment in which simmrd is installed (via remotes::install_github) and the parameter configuration is executed)
- **pixi** (Manages the R/Python environment for the command-line CLI in the cli/ repo, which runs simulations from YAML parameter files as an alternative configuration path)

## Examples

```
params <- set_params(type = "individual", sample_size_Xs = 1e6, sample_size_Y = 1e6, prop_gwas_overlap_Xs_and_Y = 0, number_of_causal_SNPs = 500); data <- generate_individual(params)
```

## Evaluation signals

- set_params() returns without error — it validates inputs immediately and returns a clear error message if something is misconfigured, so a clean return indicates a coherent configuration.
- The configured sample_size_Xs/sample_size_Y, number_of_causal_SNPs, overlap, and bias settings match the intended scenario (e.g., 1e6 nonoverlapping samples with 500 causal SNPs for the largest reported setting).
- Runtime falls within the reported envelope: small settings complete in seconds or less, and generate_individual() takes up to ~2 minutes at 1M nonoverlapping samples with 500 causal SNPs; flag the run as out of envelope if elapsed time exceeds ~2 minutes.
- The returned data list contains the expected elements (bx, by, bxse, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype) with IVtype entries restricted to "valid", "UHP", or "CHP".
- For preset-based configuration, list_presets() confirms the chosen bias/n/snps/exposures/overlap combination is among the named paper scenarios.

## Limitations

- Both generate_individual() and generate_summary() generate simulated data for a single simulation replicate only; multi-iteration Monte Carlo runs require the CLI in the cli/ repo.
- generate_individual() is slower than generate_summary() because it builds individual-level genotype and phenotype data before running GWAS; summary-level simulation is described as the faster path suitable for most simulation studies.
- Preset pleiotropy parameters scale automatically with snps (20% for UHP or CHP alone, 10% each for combined UHP_CHP/UHP_CHP_WEAK), so preset-based configurations constrain pleiotropy proportions to these rules.
- No version number for simmrd or R is stated in the article's methods section.

## Evidence

- [methods] Call `set_params()` to build a parameter list. Only specify the values you want to change — every argument has a sensible default.: "Call `set_params()` to build a parameter list. Only specify the values you want to change — every argument has a sensible default."
- [methods] `load_preset()` reproduces the named scenarios from the simmrd paper. Use `list_presets()` to see all options.: "`load_preset()` reproduces the named scenarios from the simmrd paper. Use `list_presets()` to see all options."
- [readme] set_params() validates your inputs immediately and returns a clear error message if something is misconfigured, rather than failing deep inside the simulation.: [evidence span withheld — anchor status: not_found]
- [readme] Both `simmrd::generate_individual()` and `simmrd::generate_summary()` generate simulated data for a **single** simulation replicate.: "Both `simmrd::generate_individual()` and `simmrd::generate_summary()` generate simulated data for a **single** simulation replicate."
- [methods] `generate_individual()` builds individual-level genotype and phenotype data before running GWAS, which is slower but lets you set confounding directly.: "`generate_individual()` builds individual-level genotype and phenotype data before running GWAS, which is slower but lets you set confounding directly."
- [readme] Preset pleiotropy parameters scale automatically with `snps` — 20% of SNPs are pleiotropic for single-bias scenarios (UHP or CHP), and 10% each for combined scenarios (UHP_CHP, UHP_CHP_WEAK).: [evidence span withheld — anchor status: external_doc]
