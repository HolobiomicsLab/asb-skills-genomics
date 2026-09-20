---
name: individual-level-gwas-simulation
description: Use when you need individual-level (not summary-level) simulated GWAS
  data with direct control over confounding (e.g., Xs_variance_explained_by_U, Y_variance_explained_by_U)
  for evaluating MR methods, or when benchmarking generate_individual() runtime at
  large settings.
when_to_use_negative:
- You only need summary-level GWAS statistics for most simulation studies — generate_summary()
  is the faster path and is suitable for most simulation studies.
- You need to set confounding directly at scale but runtime matters more than flexibility
  — generate_individual() is slower because it builds individual-level genotype and
  phenotype data before running GWAS.
- You require multi-iteration Monte Carlo output from one call — both generate_individual()
  and generate_summary() generate data for a single simulation replicate; use the
  cli/ pixi workflow for multi-iteration runs.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0625
tools:
- name: simmrd
  role: R package providing set_params(), generate_individual(), load_preset(), and
    list_presets() for individual-level GWAS simulation in MR
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Runtime environment in which simmrd is installed from GitHub and generate_individual()
    is executed (timed with system.time())
- name: MendelianRandomization R package
  role: Popular MR R package with which simmrd's simulated output (bx, by, bxse, byse)
    integrates for downstream method evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:0eab6adb0b81d1c5f4d4a0a4ffbf40123ade0005c9585384b4898b10afc8d0d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - MendelianRandomization R package
  - R
  - simmrd
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# individual-level-gwas-simulation

## Summary

Generate a single replicate of simulated individual-level GWAS genotype and phenotype data for Mendelian randomization using simmrd's generate_individual(), which builds individual-level data before running GWAS. This is the slower but more flexible simmrd path because confounding can be set directly.

## When to use

Use when you need individual-level (not summary-level) simulated GWAS data with direct control over confounding (e.g., Xs_variance_explained_by_U, Y_variance_explained_by_U) for evaluating MR methods, or when benchmarking generate_individual() runtime at large settings.

## When NOT to use

- You only need summary-level GWAS statistics for most simulation studies — generate_summary() is the faster path and is suitable for most simulation studies.
- You need to set confounding directly at scale but runtime matters more than flexibility — generate_individual() is slower because it builds individual-level genotype and phenotype data before running GWAS.
- You require multi-iteration Monte Carlo output from one call — both generate_individual() and generate_summary() generate data for a single simulation replicate; use the cli/ pixi workflow for multi-iteration runs.

## Inputs

- simmrd parameter list from set_params() with type = "individual" (e.g., sample_size_Xs, sample_size_Y, number_of_causal_SNPs, prop_gwas_overlap_Xs_and_Y, simtype, IV_Pvalue_threshold, fix_Fstatistic_at, Xs_variance_explained_by_U, Y_variance_explained_by_U)
- Optional preset from load_preset() (bias scenarios: none, UHP, CHP, UHP_CHP, UHP_CHP_WEAK, WEAK; n: 3e4 or 1e5; snps: 100 or 500) with per-argument overrides

## Outputs

- Named list with bx (m x p IV-exposure associations), by (m x 1 IV-outcome associations), bxse, byse, RhoME measurement-error correlation matrix, LDMatrix and LDhatMatrix (true and estimated LD correlation matrices), theta (true causal effects), IVtype classification ('valid', 'UHP', 'CHP'), and bx_unstd/by_unstd/bxse_unstd/byse_unstd
- Runtime record (e.g., runtime_report.txt) with elapsed seconds from system.time() for a single replicate plus parameter settings

## How to apply

Load the simmrd R package (installed via remotes::install_github('noahlorinczcomi/simmrd')) and call set_params() with type = "individual", specifying only the values you want to change since every argument has a sensible default (e.g., default sample sizes are 1e5 and 100 causal SNPs). Key parameters include sample_size_Xs, sample_size_Y, prop_gwas_overlap_Xs_and_Y, number_of_causal_SNPs, simtype ("winners" for P-value selection at IV_Pvalue_threshold 5e-8, or "weak" with fix_Fstatistic_at, e.g., 10), and confounding terms Xs_variance_explained_by_U and Y_variance_explained_by_U. Pass the validated parameter list to generate_individual(), which builds individual-level genotype and phenotype data before running GWAS and returns a named list containing bx, by, bxse, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, and unstandardized variants. The rationale for choosing generate_individual() over the faster generate_summary() is that it lets you set confounding directly; note that a single call produces data for only one simulation replicate. Time large runs with system.time(): at 1 million nonoverlapping samples and 500 causal SNPs a single replicate takes up to ~2 minutes, while small settings complete in seconds or less.

## Related tools

- **simmrd** (R package providing set_params(), generate_individual(), load_preset(), and list_presets() for individual-level GWAS simulation in MR) — https://github.com/noahlorinczcomi/simmrd
- **R** (Runtime environment in which simmrd is installed from GitHub and generate_individual() is executed (timed with system.time()))
- **MendelianRandomization R package** (Popular MR R package with which simmrd's simulated output (bx, by, bxse, byse) integrates for downstream method evaluation)

## Examples

```
library(simmrd); params <- set_params(type="individual", sample_size_Xs=1e6, sample_size_Y=1e6, prop_gwas_overlap_Xs_and_Y=0, number_of_causal_SNPs=500); system.time(data <- generate_individual(params))
```

## Evaluation signals

- set_params() validates inputs immediately and returns a clear error message if something is misconfigured — no validation errors should appear for a correct parameter list.
- Returned object is a named list containing bx, by, bxse, byse, RhoME, LDMatrix, LDhatMatrix, theta, IVtype, and the *_unstd variants.
- Recorded single-replicate runtime falls within the reported envelope: seconds or less at small settings and up to roughly 2 minutes at the largest reported setting (1M nonoverlapping samples, 500 causal SNPs); flag as out of envelope if elapsed time exceeds ~2 minutes.
- IVtype entries are limited to 'valid', 'UHP', or 'CHP', consistent with the simulated pleiotropy scenario (e.g., 0 UHP/CHP SNPs by default means all 'valid').
- A single call yields data for exactly one simulation replicate — repeat calls (or the cli/ multi-iteration workflow) are needed for Monte Carlo studies.

## Limitations

- generate_individual() is slower than generate_summary() because it builds individual-level genotype and phenotype data before running GWAS; at 1M nonoverlapping samples and 500 causal SNPs a single replicate can take up to 2 minutes.
- Both generate_individual() and generate_summary() generate simulated data for a single simulation replicate only.
- No version number for simmrd or R is stated in the paper's methods; version provenance must be recorded separately.
- Preset n options via load_preset() are limited to 3e4 and 1e5, so the 1e6-sample runtime setting requires explicit set_params() overrides rather than a preset.

## Evidence

- [methods] generate_individual() builds individual-level genotype and phenotype data before running GWAS, which is slower but lets you set confounding directly.: "`generate_individual()` builds individual-level genotype and phenotype data before running GWAS, which is slower but lets you set confounding directly."
- [methods] Call set_params() to build a parameter list. Only specify the values you want to change — every argument has a sensible default.: "Call `set_params()` to build a parameter list. Only specify the values you want to change — every argument has a sensible default."
- [readme] Both simmrd::generate_individual() and simmrd::generate_summary() generate simulated data for a single simulation replicate.: "Both `simmrd::generate_individual()` and `simmrd::generate_summary()` generate simulated data for a **single** simulation replicate."
- [readme] set_params() validates your inputs immediately and returns a clear error message if something is misconfigured, rather than failing deep inside the simulation.: [evidence span withheld — anchor status: external_doc]
- [methods] generate_summary() is the faster path and is suitable for most simulation studies.: "`generate_summary()` is the faster path and is suitable for most simulation studies."
