---
name: ld-block-correlation-specification
description: Use when simulating GWAS summary or individual-level data for MR where
  instrument SNPs must be in LD (e.g. autoregressive correlation within blocks) instead
  of independent, or when a realistic LDMatrix/LDhatMatrix pair is needed downstream.
when_to_use_negative:
- The MR simulation requires independent (uncorrelated) instrument SNPs — specifying
  an LD model like 'ar1(0.5)' would impose unwanted correlation
- The analysis targets real empirical GWAS summary statistics rather than simulated
  data; simmrd's LD specification only governs its own simulation output
- Individual-level confounding must be set directly via generate_individual() with
  type='individual' while the LD arguments are irrelevant to that choice — do not
  substitute LD settings for confounding parameters such as Xs_variance_explained_by_U
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_2269
tools:
- name: simmrd
  role: R package whose set_params() accepts LD_causal_SNPs and number_of_LD_blocks
    and whose generate_summary()/generate_individual() emit LDMatrix and LDhatMatrix
    for the specified LD structure
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Runtime environment in which the simmrd package is installed (remotes::install_github)
    and the simulation with LD blocks is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:d07e5ca0e3710b08aed56e7beb077b293778c3efa491f7ea04382350a5242572
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - R
  - simmrd
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# LD block correlation specification

## Summary

Configure the linkage-disequilibrium (LD) structure of causal/instrument SNPs in simmrd simulations by setting LD_causal_SNPs (e.g. 'ar1(0.5)' autoregressive correlation) and number_of_LD_blocks in the set_params() parameter list, so that generate_summary() or generate_individual() returns SNPs with correlated genotypes rather than independent instruments.

## When to use

Use when simulating GWAS summary or individual-level data for MR where instrument SNPs must be in LD (e.g. autoregressive correlation within blocks) instead of independent, or when a realistic LDMatrix/LDhatMatrix pair is needed downstream.

## When NOT to use

- The MR simulation requires independent (uncorrelated) instrument SNPs — specifying an LD model like 'ar1(0.5)' would impose unwanted correlation
- The analysis targets real empirical GWAS summary statistics rather than simulated data; simmrd's LD specification only governs its own simulation output
- Individual-level confounding must be set directly via generate_individual() with type='individual' while the LD arguments are irrelevant to that choice — do not substitute LD settings for confounding parameters such as Xs_variance_explained_by_U

## Inputs

- simmrd parameter list from set_params() with LD arguments LD_causal_SNPs (e.g. 'ar1(0.5)') and number_of_LD_blocks (e.g. 3)
- Simulation configuration values: number_of_causal_SNPs (e.g. 100), number_of_exposures (e.g. 3), sample_size_Xs/sample_size_Y (e.g. 30000)

## Outputs

- gwas_data list from generate_summary()/generate_individual() containing an LDMatrix element (true LD correlation matrix among IVs) and an LDhatMatrix element (estimated LD correlation matrix among IVs)
- m x p matrix of IV-exposure associations (bx) and m x 1 vector of IV-outcome associations (by) with correlated SNPs

## How to apply

Call set_params() with the LD-specific arguments LD_causal_SNPs (a correlation model string such as 'ar1(0.5)' for a first-order autoregressive structure with rho = 0.5) and number_of_LD_blocks (e.g. 3) alongside the rest of the simulation configuration (sample_size_Xs, sample_size_Y, number_of_causal_SNPs, number_of_exposures, pleiotropy settings like number_of_CHP_causal_SNPs, ratio_of_CHP_variance, and CHP_correlation). Only specify the values you want to change — every argument has a sensible default — and set_params() validates inputs immediately, returning a clear error rather than failing inside the simulation. Pass the resulting parameter list to generate_summary() (fast, summary-level path) or generate_individual() (individual-level genotypes before GWAS). The returned list contains LDMatrix, the true LD correlation matrix among IVs, and LDhatMatrix, the estimated LD correlation matrix, which the simulation study can use to expose MR estimators to correlated and imprecisely estimated LD. Rationale: simmrd was built to accommodate correlated IVs and imprecise LD estimation simultaneously with other realistic conditions (horizontal pleiotropy, sample overlap, weak instruments, winner's curse) in UVMR or MVMR settings.

## Related tools

- **simmrd** (R package whose set_params() accepts LD_causal_SNPs and number_of_LD_blocks and whose generate_summary()/generate_individual() emit LDMatrix and LDhatMatrix for the specified LD structure) — https://github.com/noahlorinczcomi/simmrd
- **R** (Runtime environment in which the simmrd package is installed (remotes::install_github) and the simulation with LD blocks is executed)

## Examples

```
library(simmrd); params <- set_params(number_of_causal_SNPs=100, LD_causal_SNPs='ar1(0.5)', number_of_LD_blocks=3); data <- generate_summary(params)
```

## Evaluation signals

- The returned gwas_data object contains non-NULL LDMatrix and LDhatMatrix entries whose off-diagonal values reflect the requested 'ar1(0.5)' block structure rather than an identity correlation matrix
- set_params() accepted the LD arguments without error — it validates inputs immediately and returns a clear error message if something is misconfigured
- The simulated IV count matches number_of_causal_SNPs per exposure (e.g. 100) and bx has dimensions m x p for the chosen number_of_exposures
- Downstream plot_simdata(gwas_data, params) still produces the SNP-exposure/SNP-outcome scatterplots and B_hat'theta_hat vs alpha_hat plot, confirming the LD-specified data remain valid simulation output
- LDhatMatrix differs from LDMatrix (estimated vs true LD), matching simmrd's design to accommodate imprecise LD estimation

## Limitations

- Both generate_summary() and generate_individual() generate simulated data for a single simulation replicate; LD-specified Monte Carlo studies require repeated calls (e.g. via the cli/ pixi interface)
- LD model strings such as 'ar1(0.5)' and block counts are parameter-list values — the README does not document the full enumeration of supported LD structures; consult ?set_params for the full parameter list and defaults
- No version number for simmrd or R is stated in the methods; reproducibility of the LD-specified simulation depends on tracking package versions separately

## Evidence

- [other] LD arguments in parameter list: [evidence span withheld — anchor status: not_found]
- [methods] set_params defaults and validation: "Call `set_params()` to build a parameter list. Only specify the values you want to change — every argument has a sensible default."
- [readme] LD output elements: "`LDMatrix` | True LD correlation matrix among IVs"
- [readme] estimated LD output element: "`LDhatMatrix` | Estimated LD correlation matrix among IVs"
- [intro] correlated IVs as design goal: "simmrd can accommodate each of these and even more scenarios simultaneously in either the univariable Mendelian randomization (UVMR) or MVMR settings."
