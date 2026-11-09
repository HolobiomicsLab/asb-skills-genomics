---
name: gwas-sample-overlap-parameterization
description: Use when setting up a Mendelian Randomization simulation in simmrd that
  must reflect a specific degree of exposure/outcome GWAS sample overlap — e.g. non-overlapping
  (0) vs fully overlapping (1) GWAS samples — in either UVMR or MVMR settings.
when_to_use_negative:
- The goal is a standard summary-level simulation where the default full GWAS overlap
  (prop_gwas_overlap_Xs_and_Y = 1) is acceptable — generate_summary() is the faster
  path and most simulation studies do not need individual-level generation.
- The task is to estimate or detect sample overlap in real GWAS summary statistics
  rather than to simulate data with a controlled, known overlap proportion.
- The analysis evaluates MR methods only under weak-instrument or pleiotropy conditions
  and the overlap condition is out of scope.
edam_operation: http://edamontology.org/operation_0282
tools:
- name: simmrd
  role: R package providing set_params() to specify prop_gwas_overlap_Xs_and_Y and
    generate_individual()/generate_summary() to simulate GWAS data under the configured
    overlap
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Runtime environment for the simmrd package; library(simmrd) workflow and system.time()
    timing of single replicates
- name: pixi
  role: Command-line runner for simmrd simulations (multi-iteration Monte Carlo runs)
    via the cli/ sub-repo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:1d59d433914ad6cc3b88272f32cd617d1c58bcb326cdc52cf2f780d1c944a9a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - R
  - pixi
  - simmrd
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# gwas-sample-overlap-parameterization

## Summary

Configures controlled GWAS sample overlap between exposure and outcome samples in simmrd simulations via the prop_gwas_overlap_Xs_and_Y parameter (0 = no overlap, 1 = full overlap), enabling evaluation of MR estimators under sample-overlap-induced correlated measurement error.

## When to use

Use when setting up a Mendelian Randomization simulation in simmrd that must reflect a specific degree of exposure/outcome GWAS sample overlap — e.g. non-overlapping (0) vs fully overlapping (1) GWAS samples — in either UVMR or MVMR settings.

## When NOT to use

- The goal is a standard summary-level simulation where the default full GWAS overlap (prop_gwas_overlap_Xs_and_Y = 1) is acceptable — generate_summary() is the faster path and most simulation studies do not need individual-level generation.
- The task is to estimate or detect sample overlap in real GWAS summary statistics rather than to simulate data with a controlled, known overlap proportion.
- The analysis evaluates MR methods only under weak-instrument or pleiotropy conditions and the overlap condition is out of scope.

## Inputs

- simmrd parameter list from set_params() with prop_gwas_overlap_Xs_and_Y (0–1), sample_size_Xs, sample_size_Y, number_of_causal_SNPs, and optionally type = "individual"
- Named preset scenario from load_preset() (e.g. overlap = "none" or "full") optionally overridden with a custom prop_gwas_overlap_Xs_and_Y

## Outputs

- Named simulation list with bx (m x p IV–exposure associations), by (m x 1 IV–outcome associations), bxse/byse, RhoME (measurement-error correlation matrix reflecting sample overlap), LDMatrix/LDhatMatrix, theta, and IVtype classifications
- Runtime log (e.g. runtime_report.txt) with elapsed seconds and parameter settings when timing replicates

## How to apply

Build the parameter list with set_params(), setting prop_gwas_overlap_Xs_and_Y to the desired proportion of overlap between the exposure and outcome GWAS samples (default 1 = full overlap; 0 = non-overlapping) and choosing sample_size_Xs and sample_size_Y (defaults 1e5). If individual-level genotype and phenotype data are needed, also set type = "individual" and pass the list to generate_individual(); otherwise use the faster generate_summary() path. Sample overlap matters because overlapping samples induce correlated measurement error between bx and by, which the output exposes through the (p+1) x (p+1) measurement-error correlation matrix RhoME. set_params() validates inputs immediately and returns a clear error message on misconfiguration, so parameter errors surface at setup rather than deep inside the simulation. Remember that each call produces data for a single simulation replicate, and at the largest reported setting (1M nonoverlapping samples, 500 causal SNPs) generate_individual() can take up to ~2 minutes per replicate.

## Related tools

- **simmrd** (R package providing set_params() to specify prop_gwas_overlap_Xs_and_Y and generate_individual()/generate_summary() to simulate GWAS data under the configured overlap) — https://github.com/noahlorinczcomi/simmrd
- **R** (Runtime environment for the simmrd package; library(simmrd) workflow and system.time() timing of single replicates)
- **pixi** (Command-line runner for simmrd simulations (multi-iteration Monte Carlo runs) via the cli/ sub-repo)

## Examples

```
params <- set_params(type = "individual", sample_size_Xs = 1e6, sample_size_Y = 1e6, prop_gwas_overlap_Xs_and_Y = 0, number_of_causal_SNPs = 500); data <- generate_individual(params)
```

## Evaluation signals

- set_params() accepts the requested prop_gwas_overlap_Xs_and_Y value without error; it validates inputs immediately and returns a clear error message on misconfiguration.
- The returned simulation list contains RhoME, the (p+1) x (p+1) measurement-error correlation matrix, which should reflect the configured overlap (identity-like under non-overlap).
- Each call to generate_individual() or generate_summary() yields data for exactly a single simulation replicate — batch results indicate misuse.
- Recorded runtime for a single generate_individual() replicate falls within the reported envelope: seconds or less at small settings and up to roughly 2 minutes at 1e6 nonoverlapping samples with 500 causal SNPs; flag runs exceeding ~2 minutes as out of envelope.
- Output dimensionality checks: bx is m x p and by is m x 1, with number_of_causal_SNPs (default 100) consistent with m IVs.

## Limitations

- generate_individual() builds individual-level genotype and phenotype data before running GWAS and is slower than generate_summary(); at the largest reported setting (1M nonoverlapping samples, 500 causal SNPs) a single replicate can take up to ~2 minutes.
- Both generate_individual() and generate_summary() generate data for a single replicate only; multi-iteration Monte Carlo runs require the cli/ command-line interface.
- The default simtype is "winners" (P-value-based IV selection at IV_Pvalue_threshold = 5e-8); overlap effects are entangled with winner's curse unless simtype is changed to "weak" with fix_Fstatistic_at.

## Evidence

- [readme] prop_gwas_overlap_Xs_and_Y | 1 | Proportion of overlap between exposure and outcome GWAS: [evidence span withheld — anchor status: not_found]
- [readme] two exposures with CHP, no GWAS overlap: "# two exposures with CHP, no GWAS overlap"
- [intro] simmrd can accommodate each of these and even more scenarios simultaneously in either the univariable Mendelian randomization (UVMR) or MVMR settings.: "simmrd can accommodate each of these and even more scenarios simultaneously in either the univariable Mendelian randomization (UVMR) or MVMR settings."
- [methods] generate_individual() builds individual-level genotype and phenotype data before running GWAS, which is slower but lets you set confounding directly.: "`generate_individual()` builds individual-level genotype and phenotype data before running GWAS, which is slower but lets you set confounding directly."
- [readme] Both simmrd::generate_individual() and simmrd::generate_summary() generate simulated data for a single simulation replicate.: "Both `simmrd::generate_individual()` and `simmrd::generate_summary()` generate simulated data for a **single** simulation replicate."
- [readme] RhoME | (p+1) x (p+1) measurement-error correlation matrix: [evidence span withheld — anchor status: external_doc]
