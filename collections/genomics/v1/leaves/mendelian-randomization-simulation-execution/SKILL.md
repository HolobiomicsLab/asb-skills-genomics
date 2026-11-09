---
name: mendelian-randomization-simulation-execution
description: Use when a task requires simulating GWAS summary or individual-level
  data for UVMR/MVMR under weak instruments, sample overlap, UHP/CHP, correlated IVs,
  or winner's curse, or when reproducing named scenarios from the simmrd paper.
when_to_use_negative:
- The task needs real, observed GWAS data — simmrd generates only simulated data intended
  to evaluate statistical properties of MR methods
- Individual-level confounding must be set directly and generate_summary()'s faster
  summary-level path is used instead of generate_individual()
- The prior tool MR_predictor is assumed sufficient — it cannot accommodate horizontal
  pleiotropy, sample overlap, multiple exposures, weak instruments, correlated IVs,
  imprecise LD estimation, or winner's curse
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: simmrd
  role: Core R package providing set_params(), generate_summary(), generate_individual(),
    plot_simdata(), load_preset(), and list_presets() for MR simulation
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Runtime environment in which simmrd is installed (remotes::install_github('noahlorinczcomi/simmrd'))
    and scripts are executed
- name: pixi
  role: Runs simmrd simulations from the command line via the cli/ repo with YAML
    parameter files for multi-iteration Monte Carlo runs
- name: MendelianRandomization R package
  role: Downstream package with which simmrd integrates for evaluating MR estimators
    on simulated data
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:0b6d222f93f06ecc0b14084f3dd60956194097c3435984efbb9ee43e934dde8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - MendelianRandomization R package
  - R
  - pixi
  - simmrd
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# mendelian-randomization-simulation-execution

## Summary

Execute Mendelian randomization simulation replicates with the simmrd R package: build a validated parameter list via set_params() (or load_preset()), generate summary-level or individual-level GWAS data with generate_summary()/generate_individual(), and optionally visualise with plot_simdata(). This replaces ad-hoc, unreliable hand-rolled MR simulation scripts with a standard, open-source framework.

## When to use

Use when a task requires simulating GWAS summary or individual-level data for UVMR/MVMR under weak instruments, sample overlap, UHP/CHP, correlated IVs, or winner's curse, or when reproducing named scenarios from the simmrd paper.

## When NOT to use

- The task needs real, observed GWAS data — simmrd generates only simulated data intended to evaluate statistical properties of MR methods
- Individual-level confounding must be set directly and generate_summary()'s faster summary-level path is used instead of generate_individual()
- The prior tool MR_predictor is assumed sufficient — it cannot accommodate horizontal pleiotropy, sample overlap, multiple exposures, weak instruments, correlated IVs, imprecise LD estimation, or winner's curse

## Inputs

- Parameter list from set_params() or load_preset() (e.g. number_of_exposures, number_of_causal_SNPs, prop_gwas_overlap_Xs_and_Y, number_of_CHP_causal_SNPs, ratio_of_CHP_variance, CHP_correlation, LD_causal_SNPs, IV_Pvalue_threshold)
- YAML parameter file for the CLI (e.g. params/example_summary.yaml)
- Scenario parameter files from the simmrd GitHub default_setups folder (e.g. CHP, UHP, UHP+CHP, weak IVs, UHP+CHP+weak IVs)

## Outputs

- Named list from generate_summary()/generate_individual() containing bx, by, bxse, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype ('valid'/'UHP'/'CHP'), and unstandardized variants (bx_unstd, by_unstd, bxse_unstd, byse_unstd)
- plot_simdata() figure with SNP-exposure and SNP-outcome association scatterplots plus the B̂θ̂ vs α̂ association plot
- Benchmark table of wall-clock times (e.g. generate_summary_benchmark_times.csv)

## How to apply

First call set_params() to build a parameter list, specifying only the values to change since every argument has a sensible default (e.g. sample_size_Xs=1e5, number_of_causal_SNPs=100, IV_Pvalue_threshold=5e-8, simtype='winners' or 'weak'); set_params() validates inputs immediately and returns clear error messages rather than failing deep inside the simulation. Then pass the list to generate_summary() for the fast summary-level path (suitable for most simulation studies; benchmarks show it completes in <=0.1 s per replicate) or generate_individual() when individual-level genotypes/phenotypes with directly set confounding are needed, noting each call generates data for a single simulation replicate, so loop or use the cli/ pixi pipeline (pixi run simulate --params ... -n 500 --seed 42) for Monte Carlo runs. Optionally visualise the simulated data with plot_simdata(gwas_data, params). To reproduce named paper scenarios, use load_preset() with options like 'CHP', 'UHP_CHP', 'UHP_CHP_WEAK', or 'WEAK' and dimensions n (3e4/1e5), snps (100/500), exposures (1/3), and overlap ('full'/'none'); preset pleiotropy parameters scale automatically with snps (20% pleiotropic for single-bias, 10% each for combined scenarios). Results integrate with the MendelianRandomization R package for downstream estimator evaluation.

## Related tools

- **simmrd** (Core R package providing set_params(), generate_summary(), generate_individual(), plot_simdata(), load_preset(), and list_presets() for MR simulation) — https://github.com/noahlorinczcomi/simmrd
- **R** (Runtime environment in which simmrd is installed (remotes::install_github('noahlorinczcomi/simmrd')) and scripts are executed)
- **pixi** (Runs simmrd simulations from the command line via the cli/ repo with YAML parameter files for multi-iteration Monte Carlo runs)
- **MendelianRandomization R package** (Downstream package with which simmrd integrates for evaluating MR estimators on simulated data)

## Examples

```
params <- set_params(number_of_exposures=2, number_of_CHP_causal_SNPs=20, ratio_of_CHP_variance=0.25, CHP_correlation=-0.5); data <- generate_summary(params)
```

## Evaluation signals

- The returned object is a named list containing all documented elements: bx, by, bxse, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, and the unstandardized variants
- IVtype entries are classified as one of 'valid', 'UHP', or 'CHP' per IV
- Each generate_summary() call completes within the paper's <=0.1 s wall-clock benchmark for the evaluated scenarios
- bx has dimension m x p and by dimension m x 1, where m is the number of selected IVs (filtered at IV_Pvalue_threshold=5e-8 by default) and p the number of exposures
- plot_simdata() output contains SNP-exposure scatterplots, SNP-outcome scatterplots, and the B̂θ̂ vs α̂ association plot

## Limitations

- Both generate_summary() and generate_individual() generate simulated data for a single replicate only; multi-iteration Monte Carlo runs require looping or the cli/ pixi pipeline
- No version number is stated for simmrd or R, complicating reproducibility pinning
- Individual-level simulation via generate_individual() is slower than the summary-level path
- MR simulation studies that reproduce marginally realistic settings (few IVs, extreme sample sizes) can be unreliable; very few IVs in MVMR may unrealistically reflect optimal performance since weak IV bias generally grows with more IVs

## Evidence

- [methods] Call set_params() to build a parameter list. Only specify the values you want to change — every argument has a sensible default.: "Call `set_params()` to build a parameter list. Only specify the values you want to change — every argument has a sensible default."
- [methods] generate_summary() is the faster path and is suitable for most simulation studies.: "`generate_summary()` is the faster path and is suitable for most simulation studies."
- [readme] simmrd is an R package for generating simulated GWAS data for univariable or multivariable Mendelian Randomization (MR) under various conditions of weak instrument bias, GWAS sample overlap, uncorrelated horizontal pleiotropy (UHP), and correlated horizontal pleiotropy (CHP).: "`simmrd` is an R package for generating simulated GWAS data for univariable or multivariable Mendelian Randomization (MR) under various conditions of weak instrument bias, GWAS sample overlap,"
- [readme] Both simmrd::generate_individual() and simmrd::generate_summary() generate simulated data for a single simulation replicate.: "Both `simmrd::generate_individual()` and `simmrd::generate_summary()` generate simulated data for a **single** simulation replicate."
- [readme] set_params() validates your inputs immediately and returns a clear error message if something is misconfigured, rather than failing deep inside the simulation.: [evidence span withheld — anchor status: external_doc]
- [abstract] Researchers can directly modify the simmrd source code so that the research community may arrive at a widely accepted framework.: "Researchers can directly modify the simmrd source code so that the research community may arrive at a widely accepted framework for researchers to evaluate the performance of different MR methods."
