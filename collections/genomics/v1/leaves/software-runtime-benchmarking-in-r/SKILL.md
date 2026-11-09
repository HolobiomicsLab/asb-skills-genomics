---
name: software-runtime-benchmarking-in-r
description: Use when a paper or package claims a runtime bound for a simulation/generation
  function and you need to reproduce or verify that timing claim across named scenarios
  with R's system.time().
when_to_use_negative:
- You need individual-level genotype/phenotype simulation timing — generate_individual()
  is the slower path and is not covered by the summary-level runtime bound
- You want statistical performance of MR estimators (bias, power, Type I error) rather
  than software wall-clock runtime
- The target function requires multi-iteration Monte Carlo throughput measurement
  rather than a single-call per-scenario timing
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: simmrd
  role: R package whose generate_summary() function (GWAS summary-statistic simulation
    with IV selection) is the timed target across the five default scenarios
  repo: https://github.com/noahlorinczcomi/simmrd
- name: R
  role: Runtime environment in which library(simmrd) is attached and system.time()
    is used to measure elapsed wall-clock seconds per call
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:86df79c9640d60f8c647c1d84e3717339dd70a6ff75747e9842144393b1f3e85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - R
  - simmrd
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# software-runtime-benchmarking-in-r

## Summary

Benchmark the wall-clock runtime of an R package function (here simmrd::generate_summary()) across its documented scenario presets to verify it meets a stated performance bound (≤ 0.1 s per call), producing a reproducible timing table.

## When to use

Use when a paper or package claims a runtime bound for a simulation/generation function and you need to reproduce or verify that timing claim across named scenarios with R's system.time().

## When NOT to use

- You need individual-level genotype/phenotype simulation timing — generate_individual() is the slower path and is not covered by the summary-level runtime bound
- You want statistical performance of MR estimators (bias, power, Type I error) rather than software wall-clock runtime
- The target function requires multi-iteration Monte Carlo throughput measurement rather than a single-call per-scenario timing

## Inputs

- Scenario parameter lists (default_setups files or set_params()/load_preset() outputs) for the five default scenarios: CHP, UHP, UHP+CHP, weak IVs, UHP+CHP+weak IVs
- simmrd package installed from github.com/noahlorinczcomi/simmrd in R
- Runtime threshold to test against (0.1 s wall-clock)

## Outputs

- Results table with columns: scenario name, elapsed seconds, threshold verdict
- CSV file (e.g. generate_summary_benchmark_times.csv) of measured times
- Verdict that every measured time is at or below the 0.1 s bound

## How to apply

Load the scenario parameter files (CHP, UHP, UHP+CHP, weak IVs, UHP+CHP+weak IVs) from the simmrd GitHub repository's default_setups folder, or construct equivalent parameter lists with set_params()/load_preset(). Attach the package with library(simmrd), then for each scenario call the target function exactly once while timing it with R's system.time(), recording the 'elapsed' component. Collect the measured times into a results table listing scenario name, elapsed seconds, and a threshold verdict against the stated bound (here ≤ 0.1 s). Write the table to a CSV (e.g. generate_summary_benchmark_times.csv) so the benchmark is auditable. Note that generate_summary() includes IV selection at the IV_Pvalue_threshold (default 5e-8), so the timed call covers the full data-generation-plus-selection path, not just matrix construction.

## Related tools

- **simmrd** (R package whose generate_summary() function (GWAS summary-statistic simulation with IV selection) is the timed target across the five default scenarios) — https://github.com/noahlorinczcomi/simmrd
- **R** (Runtime environment in which library(simmrd) is attached and system.time() is used to measure elapsed wall-clock seconds per call)

## Examples

```
library(simmrd); params <- load_preset("CHP", n = 1e5, snps = 100, exposures = 1, overlap = "none"); system.time(generate_summary(params))
```

## Evaluation signals

- Every measured elapsed time in the results table is ≤ 0.1 s
- Exactly five scenarios are benchmarked (CHP, UHP, UHP+CHP, weak IVs, UHP+CHP+weak IVs), one timed generate_summary() call each
- Output CSV exists (generate_summary_benchmark_times.csv) with scenario name, elapsed seconds, and threshold verdict columns
- Each timing uses system.time() around a single generate_summary(params) call, matching the paper's benchmark protocol

## Limitations

- The benchmark covers summary-level generation only; generate_individual() is documented as the slower path and is not bounded by the 0.1 s claim
- Both generate_summary() and generate_individual() generate data for a single simulation replicate, so timings reflect per-replicate cost, not multi-iteration Monte Carlo throughput
- The bound applies to the evaluated scenarios (specific GWAS sample sizes and numbers of causal SNPs); other parameter settings (e.g. larger sample_size_Y or more causal SNPs) may exceed it

## Evidence

- [results] Timing benchmarks show that generate_summary() can generate GWAS summary statistics and perform IV selection in one-tenth of a second (0.1 s) or less for all evaluated scenarios: [evidence span withheld — anchor status: not_found]
- [methods] generate_summary() is the faster path and is suitable for most simulation studies: "`generate_summary()` is the faster path and is suitable for most simulation studies."
- [readme] Both generate_summary() and generate_individual() generate simulated data for a single simulation replicate: "Both `simmrd::generate_individual()` and `simmrd::generate_summary()` generate simulated data for a **single** simulation replicate."
- [methods] Pass the parameter list to generate_summary() or generate_individual(): "2. Pass the list to `generate_summary()` or `generate_individual()`."
