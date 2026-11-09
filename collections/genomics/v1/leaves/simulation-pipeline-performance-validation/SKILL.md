---
name: simulation-pipeline-performance-validation
description: Use when reproducing or validating the runtime performance claims of
  simmrd's summary-level GWAS data generator, or when establishing that a simulation
  pipeline can produce GWAS summary statistics and select IVs within a 0.1 s budget.
when_to_use_negative:
- When benchmarking generate_individual(), which builds individual-level genotype
  and phenotype data before running GWAS and is explicitly slower — its runtime is
  not covered by the ≤ 0.1 s summary-level bound
- When the goal is to validate the statistical correctness (bias, power, Type I error)
  of MR estimators on simulated data rather than the computational performance of
  the data-generation pipeline
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3307
tools:
- name: simmrd
  role: R package whose generate_summary() function is being benchmarked for wall-clock
    runtime under five default MR simulation scenarios
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Runtime environment; system.time() is used to measure the elapsed wall-clock
    time of each generate_summary() call
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:d3435dd818078ef501776df3a3c7b7e193a79b69650009fdddc1d34b8e6b0893
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - R
  - simmrd
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# simulation-pipeline-performance-validation

## Summary

Benchmarks the wall-clock runtime of simmrd's generate_summary() function across the paper's five default simulation scenarios (CHP, UHP, UHP+CHP, weak IVs, UHP+CHP+weak IVs) to verify it meets the Table 1 performance bound of ≤ 0.1 s per replicate.

## When to use

Use when reproducing or validating the runtime performance claims of simmrd's summary-level GWAS data generator, or when establishing that a simulation pipeline can produce GWAS summary statistics and select IVs within a 0.1 s budget.

## When NOT to use

- When benchmarking generate_individual(), which builds individual-level genotype and phenotype data before running GWAS and is explicitly slower — its runtime is not covered by the ≤ 0.1 s summary-level bound
- When the goal is to validate the statistical correctness (bias, power, Type I error) of MR estimators on simulated data rather than the computational performance of the data-generation pipeline

## Inputs

- Five simmrd scenario parameter lists (CHP, UHP, UHP+CHP, weak IVs, UHP+CHP+weak IVs) from the default_setups folder of github.com/noahlorinczcomi/simmrd, or equivalent lists built with set_params()/load_preset()
- simmrd R package installed from GitHub via remotes::install_github('noahlorinczcomi/simmrd')

## Outputs

- Table of scenario name, elapsed wall-clock seconds from system.time(), and pass/fail verdict against the 0.1 s threshold
- CSV file generate_summary_benchmark_times.csv containing the benchmark results

## How to apply

Load the five scenario parameter lists (CHP, UHP, UHP+CHP, weak IVs, UHP+CHP+weak IVs) from the simmrd GitHub repository's default_setups folder, or reconstruct them with load_preset(). After attaching the package with library(simmrd), call generate_summary() once per scenario parameter list while timing the call with R's system.time(). Record the measured elapsed wall-clock seconds for each scenario into a results table listing scenario name, elapsed seconds, and a threshold verdict against the ≤ 0.1 s requirement. The rationale is that generate_summary() is the fast summary-level path of simmrd, so its per-replicate runtime directly determines the feasibility of large Monte Carlo studies; verifying the Table 1 bound confirms the claim that summary-level simulation is suitable for most simulation studies. Finally, write the table to generate_summary_benchmark_times.csv so the benchmark is auditable.

## Related tools

- **simmrd** (R package whose generate_summary() function is being benchmarked for wall-clock runtime under five default MR simulation scenarios) — https://github.com/noahlorinczcomi/simmrd
- **R** (Runtime environment; system.time() is used to measure the elapsed wall-clock time of each generate_summary() call)

## Examples

```
library(simmrd); params <- load_preset("CHP", n = 1e5, snps = 100, exposures = 1, overlap = "none"); system.time(generate_summary(params))
```

## Evaluation signals

- Every measured system.time() elapsed value for the five scenarios is at or below 0.1 s
- The results table contains exactly five rows covering CHP, UHP, UHP+CHP, weak IVs, and UHP+CHP+weak IVs, each with an elapsed-seconds column and a threshold verdict
- generate_summary_benchmark_times.csv is written and its values match the in-session measurements
- Each benchmarked generate_summary() call completes without error and returns a named list containing bx, by, bxse, byse, and IV selection outputs (IVtype)
- Timing reflects a single simulation replicate, since both generate_summary() and generate_individual() generate data for a single replicate

## Limitations

- The 0.1 s bound applies to summary-level data generation only; generate_individual() is slower and not covered by this benchmark
- The paper draft and README state no version numbers for simmrd or R, so exact runtime reproducibility across versions is not guaranteed
- Runtime results are machine-dependent; the ≤ 0.1 s requirement is validated on the benchmarking environment and may not transfer directly
- Timing covers a single replicate per call, not multi-iteration Monte Carlo runs

## Evidence

- [results] Timing benchmarks show that generate_summary() can generate GWAS summary statistics and perform IV selection in one-tenth of a second (0.1 s) or less for all evaluated scenarios: [evidence span withheld — anchor status: not_found]
- [methods] generate_summary is the fast path: "`generate_summary()` is the faster path and is suitable for most simulation studies."
- [readme] Both generate single replicate: "Both `simmrd::generate_individual()` and `simmrd::generate_summary()` generate simulated data for a **single** simulation replicate."
- [readme] Install from GitHub: "remotes::install_github('noahlorinczcomi/simmrd')"
- [intro] simmrd is an R package: "simmrd is an R package (R Core Team, 2021) and can be installed from our Github repository"
