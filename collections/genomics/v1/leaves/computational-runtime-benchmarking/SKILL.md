---
name: computational-runtime-benchmarking
description: Use when validating that a simulation tool reproduces the runtime reported
  in its paper, e.g. timing a single simmrd generate_individual() replicate at sample_size_Xs
  = 1e6 and number_of_causal_SNPs = 500.
when_to_use_negative:
- The goal is to run many Monte Carlo replicates or estimate bias/power of MR estimators
  — use generate_summary() or the cli/ pixi workflow for multi-iteration runs rather
  than timing a single individual-level replicate.
- Summary-level GWAS simulation is sufficient — generate_summary() is described as
  the faster path and the ~2 minute bound applies specifically to generate_individual()
  at the largest setting.
- The tool or parameter setting differs from the reported envelope (e.g. different
  package, more than 500 causal SNPs, multi-exposure MVMR with CHP) — the envelope
  is only defined for the paper's reported settings.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0091
tools:
- name: simmrd
  role: R package whose generate_individual() function is timed; set_params() builds
    and validates the parameter list for the largest reported setting
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Language in which simmrd runs; system.time() provides the single-replicate
    timing measurement
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:3f54a0f450d795c687a3b42220809cec9b76a6cf5aee4f4a57e7f2fbd68b4da9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - R
  - simmrd
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# computational-runtime-benchmarking

## Summary

Benchmark the wall-clock time of a single simulation replicate in the simmrd R package — here, generate_individual() at the largest reported setting (1 million nonoverlapping GWAS samples, 500 causal SNPs) — to verify it falls within the paper's reported runtime envelope (seconds at small settings, up to ~2 minutes at the largest).

## When to use

Use when validating that a simulation tool reproduces the runtime reported in its paper, e.g. timing a single simmrd generate_individual() replicate at sample_size_Xs = 1e6 and number_of_causal_SNPs = 500.

## When NOT to use

- The goal is to run many Monte Carlo replicates or estimate bias/power of MR estimators — use generate_summary() or the cli/ pixi workflow for multi-iteration runs rather than timing a single individual-level replicate.
- Summary-level GWAS simulation is sufficient — generate_summary() is described as the faster path and the ~2 minute bound applies specifically to generate_individual() at the largest setting.
- The tool or parameter setting differs from the reported envelope (e.g. different package, more than 500 causal SNPs, multi-exposure MVMR with CHP) — the envelope is only defined for the paper's reported settings.

## Inputs

- simmrd parameter list from set_params() with type='individual', sample_size_Xs=1e6, sample_size_Y=1e6, prop_gwas_overlap_Xs_and_Y=0, number_of_causal_SNPs=500
- R environment with simmrd installed from github.com/noahlorinczcomi/simmrd

## Outputs

- Single-replicate elapsed runtime in seconds from system.time(generate_individual(params))
- runtime_report.txt containing the elapsed seconds together with the parameter settings used

## How to apply

Install simmrd from GitHub (remotes::install_github('noahlorinczcomi/simmrd')) and load it with library(simmrd). Build the parameter list with set_params(), setting type = 'individual', sample_size_Xs = 1e6, sample_size_Y = 1e6, prop_gwas_overlap_Xs_and_Y = 0 (non-overlapping GWAS samples), and number_of_causal_SNPs = 500, leaving all other arguments at their defaults — set_params() validates inputs immediately and errors early on misconfiguration. Time exactly one call to generate_individual(params) with system.time(); generate_individual() builds individual-level genotype and phenotype data before running GWAS, which is why it is slower than generate_summary() but allows confounding to be set directly. Record elapsed seconds and the parameter settings to runtime_report.txt. Judge success against the reported envelope: small settings complete in seconds or less and the largest reported setting (1M nonoverlapping samples, 500 causal SNPs) takes up to 2 minutes for a single replicate; flag the run as out of envelope if elapsed time exceeds ~2 minutes.

## Related tools

- **simmrd** (R package whose generate_individual() function is timed; set_params() builds and validates the parameter list for the largest reported setting) — https://github.com/noahlorinczcomi/simmrd
- **R** (Language in which simmrd runs; system.time() provides the single-replicate timing measurement)

## Examples

```
library(simmrd); params <- set_params(type='individual', sample_size_Xs=1e6, sample_size_Y=1e6, prop_gwas_overlap_Xs_and_Y=0, number_of_causal_SNPs=500); system.time(generate_individual(params))
```

## Evaluation signals

- A single generate_individual() replicate at sample_size_Xs = 1e6, sample_size_Y = 1e6, prop_gwas_overlap_Xs_and_Y = 0, and number_of_causal_SNPs = 500 completes within roughly 2 minutes.
- Small parameter settings complete in seconds or less, consistent with the reported runtime envelope in Table 1.
- runtime_report.txt contains both the elapsed seconds and the exact parameter settings, enabling reproducibility of the timing run.
- set_params() accepted the configuration without error, confirming the individual-type parameters were valid before timing began.
- Timing reflects exactly one call to generate_individual() — both generate_summary() and generate_individual() produce data for a single simulation replicate, so elapsed time must not be averaged over multiple iterations unless explicitly stated.

## Limitations

- The ~2 minute bound applies to a single replicate at the largest reported setting; runtime scales with sample size and number of causal SNPs and grows from seconds to ~2 minutes.
- generate_individual() is inherently slower than generate_summary() because it builds individual-level genotype and phenotype data before running GWAS.
- No version number for simmrd or R is stated in the methods, so runtime comparisons may be sensitive to package version differences.
- Hardware-dependent: the reported envelope was measured on the authors' machine; slower systems may exceed 2 minutes without indicating a bug.

## Evidence

- [methods] single-replicate runtime bound: "generate_individual() builds individual-level genotype and phenotype data before running GWAS, which is slower but lets you set confounding directly."
- [readme] single replicate semantics: "Both `simmrd::generate_individual()` and `simmrd::generate_summary()` generate simulated data for a **single** simulation replicate."
- [readme] parameter construction and validation: "`set_params()` builds a parameter list with sensible defaults. You only need to specify the values you want to change."
- [readme] install source: "remotes::install_github('noahlorinczcomi/simmrd')"
- [readme] default parameter values: [evidence span withheld — anchor status: external_doc]
