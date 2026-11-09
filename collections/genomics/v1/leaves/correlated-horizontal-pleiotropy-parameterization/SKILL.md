---
name: correlated-horizontal-pleiotropy-parameterization
description: Use when building MR simulation scenarios in which invalid instruments
  violate the InSIDE assumption via CHP — e.g. evaluating MR estimators' bias under
  correlated pleiotropy using simmrd.
when_to_use_negative:
- The scenario targets uncorrelated horizontal pleiotropy only, where IV effects are
  independent (use number_of_UHP_causal_SNPs instead of the CHP parameters)
- You need individual-level confounding control via generate_individual() and the
  research question concerns confounder U rather than pleiotropy structure
- Only valid instruments are desired (default number_of_CHP_causal_SNPs = 0 should
  be left unchanged)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_2269
tools:
- name: simmrd
  role: R package whose set_params(), load_preset(), and generate_summary()/generate_individual()
    functions parameterize and realize CHP in simulated GWAS data
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Runtime environment in which the simmrd package is installed and executed
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:ca0d58f0951f3428d13cf54f81e65d6d1bc3ea88c0c311432dc30c3addb2648e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - R
  - simmrd
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Correlated horizontal pleiotropy parameterization

## Summary

Configure correlated horizontal pleiotropy (CHP) in simmrd simulations by setting the number of CHP causal SNPs, the ratio of CHP variance, and the CHP correlation, so that IV–outcome associations share a correlated pleiotropic component across replicates.

## When to use

Use when building MR simulation scenarios in which invalid instruments violate the InSIDE assumption via CHP — e.g. evaluating MR estimators' bias under correlated pleiotropy using simmrd.

## When NOT to use

- The scenario targets uncorrelated horizontal pleiotropy only, where IV effects are independent (use number_of_UHP_causal_SNPs instead of the CHP parameters)
- You need individual-level confounding control via generate_individual() and the research question concerns confounder U rather than pleiotropy structure
- Only valid instruments are desired (default number_of_CHP_causal_SNPs = 0 should be left unchanged)

## Inputs

- simmrd parameter list from set_params() with number_of_CHP_causal_SNPs, ratio_of_CHP_variance, and CHP_correlation specified
- Named preset string (e.g. 'CHP', 'UHP_CHP', 'UHP_CHP_WEAK') passed to load_preset() with n, snps, exposures, and overlap options

## Outputs

- Simulated GWAS summary data (bx, by, bxse, byse) containing CHP-affected IV–outcome associations
- RhoME (p+1)x(p+1) measurement-error correlation matrix reflecting the CHP-induced covariance structure
- IVtype vector classifying each instrument as 'valid', 'UHP', or 'CHP'

## How to apply

Call set_params() with number_of_CHP_causal_SNPs (e.g. 20 of 100 causal SNPs), ratio_of_CHP_variance (e.g. 0.25), and CHP_correlation (e.g. -0.5) to induce a pleiotropic component whose effects on the outcome are correlated across SNPs, alongside the structural model Y = x'θ + πY U + g'γU + εY. Keep the number_of_causal_SNPs and prop_gwas_overlap_Xs_and_Y settings consistent across scenario comparisons so that only the CHP parameters vary. Alternatively, use load_preset('CHP') or load_preset('UHP_CHP'), noting that preset pleiotropy parameters scale automatically with snps — 20% of SNPs are pleiotropic for single-bias scenarios (UHP or CHP) and 10% each for combined scenarios. Pass the parameter list to generate_summary() (or generate_individual()) and use the returned IVtype element, which classifies each IV as 'valid', 'UHP', or 'CHP', to confirm the intended pleiotropic SNP allocation.

## Related tools

- **simmrd** (R package whose set_params(), load_preset(), and generate_summary()/generate_individual() functions parameterize and realize CHP in simulated GWAS data) — https://github.com/noahlorinczcomi/simmrd
- **R** (Runtime environment in which the simmrd package is installed and executed)

## Examples

```
params <- set_params(number_of_exposures=3, number_of_causal_SNPs=100, number_of_CHP_causal_SNPs=20, ratio_of_CHP_variance=0.25, CHP_correlation=-0.5); data <- generate_summary(params)
```

## Evaluation signals

- IVtype output contains exactly number_of_CHP_causal_SNPs SNPs classified as 'CHP' (e.g. 20 of 100)
- Preset scenarios scale pleiotropy automatically: 20% pleiotropic SNPs for 'CHP' and 10% each for 'UHP_CHP'/'UHP_CHP_WEAK'
- set_params() validates inputs immediately and returns a clear error message if CHP parameters are misconfigured
- Empirical error covariance of (bx - beta_true, by - alpha_true) across replicates matches the returned RhoME matrix within Monte Carlo tolerance
- generate_summary() returns data for a single replicate, so CHP-induced bias must be assessed over many independent calls

## Limitations

- Both generate_summary() and generate_individual() generate data for a single simulation replicate, so Monte Carlo looping is required to characterize CHP effects on estimator bias
- Preset CHP parameters are fixed ratios of the SNP count (20% single-bias, 10% each combined); custom scenarios require overriding set_params() values directly
- MR simulation studies without a unified framework can produce conflicting results about bias/power/Type I error, so CHP settings should be reported precisely and shared via the open-source code

## Evidence

- [readme] CHP parameters via set_params and presets: [evidence span withheld — anchor status: external_doc]
- [readme] IVtype classification: [evidence span withheld — anchor status: external_doc]
- [readme] set_params validation: [evidence span withheld — anchor status: external_doc]
- [intro] simmrd scope of scenarios: [evidence span withheld — anchor status: not_found]
- [methods] parameter list construction: "Call `set_params()` to build a parameter list. Only specify the values you want to change — every argument has a sensible default."
