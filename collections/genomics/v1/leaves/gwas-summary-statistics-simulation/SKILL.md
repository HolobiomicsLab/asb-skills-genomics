---
name: gwas-summary-statistics-simulation
description: Use when evaluating or comparing MR estimators (e.g., IVW, dIVW, MR-RAPS)
  on simulated data, or when you need replicates of GWAS summary-level data with known
  true causal effects, pleiotropy labels, and measurement-error structure.
when_to_use_negative:
- Input is already real GWAS summary statistics — simmrd generates simulated data
  with known ground truth (beta_true/alpha_true), not an analysis tool for observed
  GWAS results
- You need individual-level genotypes/phenotypes with directly set confounding and
  speed matters — generate_summary() is the faster path; generate_individual() is
  slower but required when Y_variance_explained_by_U must be set directly
- The scenario requires features MR_predictor-era tools or simmrd cannot accommodate;
  check that horizontal pleiotropy, sample overlap, weak IVs, correlated IVs, or winner's
  curse are among simmrd's supported conditions before assuming coverage
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2269
tools:
- name: simmrd
  role: R package providing set_params(), generate_summary(), generate_individual(),
    plot_simdata(), load_preset(), and list_presets() for generating simulated GWAS
    data for MR
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Runtime environment in which the simmrd package is installed and executed
- name: pixi
  role: Command-line runner for the cli/ repo enabling multi-iteration Monte Carlo
    simulation from a YAML parameter file
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_005
  - task_002
  - task_006
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:c34ad6fd7fb2be6844575ce5c26225e8ed92ff1170cf8726e20356e3b10c92b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - R
  - generate_summary
  - pixi
  - set_params
  - simmrd
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# gwas-summary-statistics-simulation

## Summary

Generates simulated GWAS summary statistics for univariable or multivariable Mendelian randomization using the simmrd R package, under weak-instrument bias, GWAS sample overlap, UHP, and CHP conditions. It provides a standard simulation environment for evaluating the statistical properties of MR estimators.

## When to use

Use when evaluating or comparing MR estimators (e.g., IVW, dIVW, MR-RAPS) on simulated data, or when you need replicates of GWAS summary-level data with known true causal effects, pleiotropy labels, and measurement-error structure.

## When NOT to use

- Input is already real GWAS summary statistics — simmrd generates simulated data with known ground truth (beta_true/alpha_true), not an analysis tool for observed GWAS results
- You need individual-level genotypes/phenotypes with directly set confounding and speed matters — generate_summary() is the faster path; generate_individual() is slower but required when Y_variance_explained_by_U must be set directly
- The scenario requires features MR_predictor-era tools or simmrd cannot accommodate; check that horizontal pleiotropy, sample overlap, weak IVs, correlated IVs, or winner's curse are among simmrd's supported conditions before assuming coverage

## Inputs

- Parameter list from simmrd::set_params() (e.g., number_of_exposures, sample_size_Xs, sample_size_Y, prop_gwas_overlap_Xs_and_Y, number_of_CHP_causal_SNPs, simtype, fix_Fstatistic_at)
- Preset name and overrides for simmrd::load_preset() (e.g., 'CHP', n = 1e5, snps = 100, exposures = 1, overlap = 'none')
- YAML parameter file for the simmrd CLI (cli/params/example_summary.yaml) with iteration count and seed

## Outputs

- Named list per replicate containing bx (m x p), bxse, by (m x 1), byse, RhoME ((p+1) x (p+1)), LDMatrix, LDhatMatrix, theta, IVtype vector, and bx_unstd/bxse_unstd/by_unstd/byse_unstd
- Optional plot_simdata() figure with SNP-exposure, SNP-outcome, and B̂θ̂ vs α̂ scatterplots
- RDS file of replicates for downstream MR-method evaluation (e.g., results.rds from the CLI)

## How to apply

Install simmrd from GitHub (remotes::install_github('noahlorinczcomi/simmrd')) and build a parameter list with set_params(), specifying only non-default values such as number_of_exposures, sample_size_Xs/sample_size_Y (default 1e5), prop_gwas_overlap_Xs_and_Y, number_of_UHP/CHP_causal_SNPs, ratio_of_CHP_variance, CHP_correlation, and simtype ('winners' with IV_Pvalue_threshold=5e-8 or 'weak' with fix_Fstatistic_at=10). Pass the list to generate_summary() for fast summary-level simulation (the faster path, ≤0.1 s per replicate, generating GWAS statistics plus IV selection) or generate_individual() when confounding must be set directly via Y_variance_explained_by_U and Xs_variance_explained_by_U. To reproduce paper scenarios, use list_presets() and load_preset('CHP'|'UHP'|'UHP_CHP'|'UHP_CHP_WEAK'|'WEAK', n=, snps=, exposures=, overlap=) — pleiotropy parameters scale automatically with snps (20% for single-bias, 10% each for combined scenarios). Each replicate returns a named list including bx (m x p IV-exposure matrix), by, bxse, byse, the (p+1) x (p+1) measurement-error correlation matrix RhoME, LDMatrix/LDhatMatrix, true causal effects theta, and per-IV IVtype labels ('valid', 'UHP', 'CHP'); optionally visualise with plot_simdata().

## Related tools

- **simmrd** (R package providing set_params(), generate_summary(), generate_individual(), plot_simdata(), load_preset(), and list_presets() for generating simulated GWAS data for MR) — https://github.com/noahlorinczcomi/simmrd
- **R** (Runtime environment in which the simmrd package is installed and executed)
- **pixi** (Command-line runner for the cli/ repo enabling multi-iteration Monte Carlo simulation from a YAML parameter file)

## Examples

```
library(simmrd); params <- set_params(number_of_CHP_causal_SNPs = 20, ratio_of_CHP_variance = 0.25, CHP_correlation = -0.5); data <- generate_summary(params)
```

## Evaluation signals

- names(data) contains all ten documented components: bx, bxse, by, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, and the unstandardized variants bx_unstd/bxse_unstd/by_unstd/byse_unstd
- Structural checks: bx is m x p, by is m x 1, and IVtype labels each IV as 'valid', 'UHP', or 'CHP' with class counts matching the preset configuration (e.g., number_of_CHP_causal_SNPs = 20)
- For simtype = 'weak' with fix_Fstatistic_at = 10, the mean first-stage F-statistic computed from bx and bxse is centred on the requested value
- Empirical covariance of estimation errors (bx - beta_true, by - alpha_true) across replicates matches the returned RhoME structure within Monte Carlo tolerance
- system.time() on generate_summary() shows elapsed wall-clock time ≤ 0.1 s for the five default scenarios

## Limitations

- Both generate_summary() and generate_individual() generate data for a single simulation replicate; multi-replicate Monte Carlo runs must loop externally or use the pixi CLI
- No version number is stated for simmrd or R, and the associated working paper draft has not been peer-checked or reviewed
- MR simulations built on unrealistic settings (few IVs, very small or very large GWAS sample sizes, few IVs in MVMR) may unrealistically reflect optimal estimator performance; choose parameters to reflect realistic conditions

## Evidence

- [readme] TL;DR invocation with defaults and a documented CHP configuration: "`simmrd` is an R package for generating simulated GWAS data for univariable or multivariable Mendelian Randomization (MR) under various conditions of weak instrument bias, GWAS sample overlap,"
- [methods] Workflow ordering for generating data: "1. Call `set_params()` to build a parameter list. Only specify the values you want to change — every argument has a sensible default."
- [readme] Presets reproduce the paper's named scenarios: [evidence span withheld — anchor status: external_doc]
- [readme] Output schema of the returned named list: [evidence span withheld — anchor status: external_doc]
- [results] Runtime bound for the summary-level generator: [evidence span withheld — anchor status: not_found]
- [intro] Rationale for using simmrd as a standard framework: "the purpose of the open-source simmrd software is to begin the construction of a standard simulation environment in which fair comparisons can be made between competing MR estimators"
