---
name: measurement-error-covariance-verification
description: Use when reimplementing simmrd's structural MR data-generating model
  and you need to confirm that empirical SNP-exposure/SNP-outcome estimation-error
  covariances match the analytic error covariance target (RhoME) computed via LeBlanc
  et al. (2018).
when_to_use_negative:
- You only need to simulate GWAS summary data for MR method evaluation (generate_summary()
  alone suffices); this skill is for validating a reimplementation of the generator
  itself
- Your data come from real GWAS summary statistics rather than simmrd's generator
  — there is no beta_true/alpha_true to compare against
- You need individual-level genotype/phenotype simulation with directly set confounding;
  use generate_individual() instead of the summary-level error-covariance path
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2269
tools:
- name: simmrd
  role: R package providing set_params() and generate_summary() whose returned RhoME
    measurement-error correlation matrix and beta_true/alpha_true supply the analytic
    verification target
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Runtime environment in which simmrd is installed (remotes::install_github)
    and the standalone structural-equation script and covariance comparison are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:a678d481a551097f4c0beadc4759c2679b090a4a23b62c07c12387438757809a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - R
  - simmrd
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# measurement-error-covariance-verification

## Summary

Verifies a standalone reimplementation of simmrd's direct summary-statistics data-generating algorithm (U = g'γC + εU; x = B'g + πxU + εx; Y = x'θ + πYU + g'γU + εY) by checking that the empirical covariance of GWAS estimation errors (bx − beta_true, by − alpha_true) reproduces the true (p+1)×(p+1) measurement-error covariance RhoME returned by generate_summary().

## When to use

Use when reimplementing simmrd's structural MR data-generating model and you need to confirm that empirical SNP-exposure/SNP-outcome estimation-error covariances match the analytic error covariance target (RhoME) computed via LeBlanc et al. (2018).

## When NOT to use

- You only need to simulate GWAS summary data for MR method evaluation (generate_summary() alone suffices); this skill is for validating a reimplementation of the generator itself
- Your data come from real GWAS summary statistics rather than simmrd's generator — there is no beta_true/alpha_true to compare against
- You need individual-level genotype/phenotype simulation with directly set confounding; use generate_individual() instead of the summary-level error-covariance path

## Inputs

- simmrd parameter list from set_params() (number_of_exposures, number_of_causal_SNPs, number_of_CHP_causal_SNPs, ratio_of_CHP_variance, CHP_correlation, LD_causal_SNPs, number_of_LD_blocks)
- Repeated generate_summary() output lists containing bx, by, bxse, byse, RhoME, theta, beta_true, alpha_true, u, v, iv_index
- Standalone reimplementation of the structural equations U = g'γC + εU; x = B'g + πxU + εx; Y = x'θ + πYU + g'γU + εY

## Outputs

- covariance_verification.csv: per-element table of empirical error covariance, true RhoME covariance, and absolute difference
- Empirical (p+1)×(p+1) measurement-error covariance matrix accumulated across replicates

## How to apply

Install simmrd from https://github.com/noahlorinczcomi/simmrd and build a parameter list with set_params() specifying number_of_exposures = 3, number_of_causal_SNPs = 100, prop_gwas_overlap_Xs_and_Y = 1, number_of_CHP_causal_SNPs = 20, ratio_of_CHP_variance = 0.25, CHP_correlation = -0.5, and LD_causal_SNPs = 'ar1(0.5)' across 3 LD blocks. Implement the structural equations (confounder U, exposures x, outcome Y with UHP and CHP terms) in a standalone script, then repeatedly call generate_summary(params) to obtain bx, by, bxse, byse, RhoME, theta, beta_true, alpha_true, u, v, and iv_index. Across many replicates, compute the empirical covariance of bx − beta_true and by − alpha_true and compare it element-wise against the returned (p+1)×(p+1) RhoME matrix. Write per-element comparisons (empirical covariance, true covariance, absolute difference) to covariance_verification.csv.

## Related tools

- **simmrd** (R package providing set_params() and generate_summary() whose returned RhoME measurement-error correlation matrix and beta_true/alpha_true supply the analytic verification target) — https://github.com/noahlorinczcomi/simmrd
- **R** (Runtime environment in which simmrd is installed (remotes::install_github) and the standalone structural-equation script and covariance comparison are executed)

## Examples

```
library(simmrd); params <- set_params(number_of_exposures=3, number_of_causal_SNPs=100, prop_gwas_overlap_Xs_and_Y=1, number_of_CHP_causal_SNPs=20, ratio_of_CHP_variance=0.25, CHP_correlation=-0.5); d <- generate_summary(params)
```

## Evaluation signals

- Element-wise absolute differences between empirical error covariance and RhoME shrink as the number of replicates increases (Monte Carlo convergence)
- The empirical error covariance is symmetric and has the same (p+1)×(p+1) block structure as the returned RhoME matrix
- generate_summary() returns all expected elements: bx, by, bxse, byse, RhoME, theta, beta_true, alpha_true, u, v, iv_index
- covariance_verification.csv is produced with columns for empirical covariance, true covariance, and absolute difference for every element
- bx and by estimation errors are consistent with the bxse/byse standard errors and the LD structure (ar1(0.5)) encoded in RhoME

## Limitations

- generate_summary() and generate_individual() each generate data for a single simulation replicate, so many calls are needed for Monte Carlo convergence of the empirical covariance
- simmrd output and RhoME reflect the simtype used ('winners' P-value selection induces winner's curse; 'weak' fixes the F-statistic, e.g. at 10), which changes the error structure being verified
- There is no stated version number for simmrd or R, and the associated paper's methods rely on LeBlanc et al. (2018) covariance formulas whose exact implementation must match the installed package

## Evidence

- [other] simmrd's data-generating functions directly return the true covariance matrix of GWAS estimation errors, Cov(B̂−B, α̂−α), computed using the methods of LeBlanc et al. (2018): [evidence span withheld — anchor status: not_found]
- [readme] generate_summary returns named list with RhoME and true betas: "Both `generate_summary()` and `generate_individual()` return a named list:"
- [readme] RhoME is (p+1)x(p+1) measurement-error correlation matrix: [evidence span withheld — anchor status: external_doc]
- [methods] set_params builds parameter list with sensible defaults: "Call `set_params()` to build a parameter list. Only specify the values you want to change — every argument has a sensible default."
- [readme] install simmrd from GitHub: "remotes::install_github('noahlorinczcomi/simmrd')"
- [readme] single replicate per call: "Both `simmrd::generate_individual()` and `simmrd::generate_summary()` generate simulated data for a **single** simulation replicate."
