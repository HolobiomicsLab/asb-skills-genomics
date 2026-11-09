---
name: monte-carlo-replicate-simulation
description: Use when evaluating the statistical properties of MR methods (e.g., dIVW,
  MR-RAPS) via Monte Carlo replicates of simulated GWAS summary data under scenarios
  like UHP, CHP, weak instruments, or GWAS sample overlap.
when_to_use_negative:
- You need a single exploratory visualization of simulated data rather than repeated
  replicates — use plot_simdata() on one generate_summary()/generate_individual()
  output instead
- The goal is empirical calibration against the true measurement-error covariance
  structure (Algorithm 2 / RhoME verification) rather than MR estimator performance
  evaluation across replicates
- Simulation settings use very few IVs in MVMR — the paper warns this may unrealistically
  reflect optimal performance because weak-IV bias generally worsens with more IVs
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: simmrd
  role: R package providing set_params(), load_preset(), generate_summary(), and generate_individual()
    to produce one simulated GWAS summary replicate per call for MR Monte Carlo studies
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Runtime environment in which simmrd is installed via remotes::install_github()
    and replication loops over generate_summary() calls are executed
- name: pixi
  role: Command-line runner for multi-iteration Monte Carlo simulations from a YAML
    parameter file with reproducible seeding (pixi run simulate --params ... --output
    results.rds -n 500 --seed 42)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:b754e70bfdc6b12e89d4620926e2eacb38a859b713b8311ffe1b651914acd341
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - R
  - pixi
  - simmrd
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# monte-carlo-replicate-simulation

## Summary

Run repeated simulation replicates of GWAS summary-level Mendelian randomization data with simmrd to empirically evaluate MR estimator performance (bias, Type I error, power) under pleiotropy, weak instruments, and sample overlap. Each replicate is generated independently and aggregated across iterations for Monte Carlo assessment.

## When to use

Use when evaluating the statistical properties of MR methods (e.g., dIVW, MR-RAPS) via Monte Carlo replicates of simulated GWAS summary data under scenarios like UHP, CHP, weak instruments, or GWAS sample overlap.

## When NOT to use

- You need a single exploratory visualization of simulated data rather than repeated replicates — use plot_simdata() on one generate_summary()/generate_individual() output instead
- The goal is empirical calibration against the true measurement-error covariance structure (Algorithm 2 / RhoME verification) rather than MR estimator performance evaluation across replicates
- Simulation settings use very few IVs in MVMR — the paper warns this may unrealistically reflect optimal performance because weak-IV bias generally worsens with more IVs

## Inputs

- simmrd parameter list from set_params() (e.g., number_of_exposures, number_of_causal_SNPs, prop_gwas_overlap_Xs_and_Y, number_of_UHP_causal_SNPs, number_of_CHP_causal_SNPs, ratio_of_CHP_variance, CHP_correlation, simtype, IV_Pvalue_threshold)
- Named preset scenario from load_preset() (bias: none/UHP/CHP/UHP_CHP/UHP_CHP_WEAK/WEAK; n: 3e4/1e5; snps: 100/500; exposures: 1/3; overlap: full/none)
- YAML parameter file for the pixi-based CLI (e.g., params/example_summary.yaml) with iteration count and seed

## Outputs

- Named list per replicate with bx (m x p IV–exposure matrix), by (m x 1 IV–outcome vector), bxse, byse, RhoME ((p+1) x (p+1) measurement-error correlation matrix), LDMatrix, LDhatMatrix, theta (true causal effects), IVtype (per-IV 'valid'/'UHP'/'CHP' labels), and unstandardized variants bx_unstd/by_unstd with their standard errors
- Aggregated Monte Carlo results across replicates (e.g., empirical error covariance, bias, Type I error, power of MR estimators), saved e.g. as results.rds from the CLI

## How to apply

Build a parameter list with simmrd::set_params(), overriding only the settings of interest (e.g., number_of_exposures = 3, number_of_causal_SNPs = 100, number_of_CHP_causal_SNPs = 20, ratio_of_CHP_variance = 0.25, CHP_correlation = -0.5, prop_gwas_overlap_Xs_and_Y, simtype = 'weak' with fix_Fstatistic_at = 10); set_params() validates inputs immediately and errors clearly on misconfiguration. Alternatively load a named paper scenario with load_preset() after listing options via list_presets(). For each Monte Carlo replicate, call generate_summary(params) (faster, summary-level) or generate_individual(params) (slower, individual-level genotypes/phenotypes with direct confounding control) — each call produces a single replicate. Collect the returned bx, by, bxse, byse, RhoME, theta, and IVtype elements across replicates, then aggregate estimator performance (e.g., empirical covariance of bx - beta_true and by - alpha_true against RhoME) over iterations. For multi-iteration runs from the command line, use the pixi-based CLI in the cli/ directory with a YAML parameter file and a seed.

## Related tools

- **simmrd** (R package providing set_params(), load_preset(), generate_summary(), and generate_individual() to produce one simulated GWAS summary replicate per call for MR Monte Carlo studies) — https://github.com/noahlorinczcomi/simmrd
- **R** (Runtime environment in which simmrd is installed via remotes::install_github() and replication loops over generate_summary() calls are executed)
- **pixi** (Command-line runner for multi-iteration Monte Carlo simulations from a YAML parameter file with reproducible seeding (pixi run simulate --params ... --output results.rds -n 500 --seed 42))

## Examples

```
params <- set_params(number_of_exposures = 3, number_of_causal_SNPs = 100, number_of_CHP_causal_SNPs = 20, ratio_of_CHP_variance = 0.25, CHP_correlation = -0.5, LD_causal_SNPs = 'ar1(0.5)'); data <- generate_summary(params)
```

## Evaluation signals

- Each replicate's returned list contains all expected elements (bx, by, bxse, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype) with correct dimensions: bx is m x p, RhoME is (p+1) x (p+1)
- Across replicates, the empirical covariance of (bx - beta_true, by - alpha_true) converges to the returned RhoME structure, with element-wise absolute differences shrinking as replicate count increases
- IVtype labels partition the IVs consistently with the requested pleiotropy settings (e.g., ~20% pleiotropic SNPs for single-bias presets, 10% each for combined UHP_CHP scenarios)
- Parameter lists from set_params() with invalid configurations raise clear immediate validation errors rather than failing deep inside the simulation
- Multi-iteration CLI runs with the same --seed reproduce identical replicate sequences

## Limitations

- Both generate_summary() and generate_individual() generate data for a SINGLE simulation replicate — the caller must loop over replicates or use the CLI for Monte Carlo runs
- generate_individual() is slower than generate_summary(), trading speed for direct control of confounding at the individual level
- There is no standard simulation framework for MR, so replicate results may not be directly comparable across independent studies with similar reported settings
- No version number is stated for simmrd or R in the methods; install from GitHub may track a moving API
- Default simtype is 'winners' (P-value-based IV selection at IV_Pvalue_threshold = 5e-8), which induces winner's curse unless deliberately changed to 'weak' with fix_Fstatistic_at

## Evidence

- [readme] Both `simmrd::generate_individual()` and `simmrd::generate_summary()` generate simulated data for a **single** simulation replicate.: "Both `simmrd::generate_individual()` and `simmrd::generate_summary()` generate simulated data for a **single** simulation replicate."
- [methods] Call set_params() to build a parameter list, pass it to a generator, optionally plot: "1. Call `set_params()` to build a parameter list. Only specify the values you want to change — every argument has a sensible default."
- [methods] generate_summary() is the faster path for simulation studies: "`generate_summary()` is the faster path and is suitable for most simulation studies."
- [readme] set_params() validates inputs immediately: [evidence span withheld — anchor status: external_doc]
- [readme] CLI multi-iteration Monte Carlo command with seed: "pixi run simulate --params params/example_summary.yaml --output results.rds -n 500 --seed 42"
