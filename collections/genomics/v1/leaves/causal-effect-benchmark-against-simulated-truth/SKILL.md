---
name: causal-effect-benchmark-against-simulated-truth
description: Use when you have simulated IV–exposure (bx, bxse) and IV–outcome (by,
  byse) summary associations from simmrd's generate_summary() and want to benchmark
  MR estimator causal estimates against known true effects.
when_to_use_negative:
- Input is real, not simulated, GWAS summary data — there is no known truth (theta)
  to benchmark against
- 'You need multiple simulation replicates from a single call: simmrd''s generate_summary()
  and generate_individual() each generate data for a single simulation replicate only'
- You need individual-level genotype/phenotype simulation with direct confounding
  control — use generate_individual() workflow instead, which is slower but lets you
  set confounding directly
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: simmrd
  role: Generates the simulated GWAS summary replicate (bx, bxse, by, byse, theta)
    that serves as the benchmark input data
  repo: https://github.com/noahlorinczcomi/simmrd
- name: simmrd (R package)
  role: R interface for set_params()/generate_summary() to build parameter lists and
    emit the summary replicate
  repo: https://github.com/noahlorinczcomi/simmrd
- name: MendelianRandomization R package
  role: Consumes the simulated bx/bxse/by/byse via an MR input object and runs summary-data
    estimators such as IVW or MR-Egger
- name: R
  role: Runtime environment in which simmrd and MendelianRandomization are loaded
    and mr_estimates.csv is written
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/gepi.22544
    title: 'simmrd: An open‐source tool to perform simulations in Mendelian randomization'
  source_content_hash: sha256:012ddde0222bd4f20b8c309764307b911690d610828b0fc7baa1af8678e8b660
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/noahlorinczcomi/simmrd
  tools:
  - MendelianRandomization R package
  - R
  - simmrd
  - simmrd (R package)
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# causal-effect-benchmark-against-simulated-truth

## Summary

Fit competing summary-data Mendelian randomization estimators (e.g. IVW, MR-Egger via the MendelianRandomization R package) on simmrd-generated GWAS summary replicates and compare each estimate against the true causal effects (theta) that simmrd returns, enabling benchmarking of estimator bias and validity.

## When to use

Use when you have simulated IV–exposure (bx, bxse) and IV–outcome (by, byse) summary associations from simmrd's generate_summary() and want to benchmark MR estimator causal estimates against known true effects.

## When NOT to use

- Input is real, not simulated, GWAS summary data — there is no known truth (theta) to benchmark against
- You need multiple simulation replicates from a single call: simmrd's generate_summary() and generate_individual() each generate data for a single simulation replicate only
- You need individual-level genotype/phenotype simulation with direct confounding control — use generate_individual() workflow instead, which is slower but lets you set confounding directly

## Inputs

- simmrd generate_summary() output list: bx (m x p IV–exposure associations), bxse, by (m x 1 IV–outcome associations), byse
- true causal effects theta returned by simmrd for the replicate
- simulated replicate parameter list from set_params() or load_preset() (e.g. number_of_exposures, true_causal_effects, prop_gwas_overlap_Xs_and_Y)

## Outputs

- mr_estimates.csv with one structured record per estimator: estimator name and finite numeric causal effect estimate(s)
- Comparison of each estimate against simmrd's true causal effects (theta)

## How to apply

Load a single simulated replicate produced by simmrd's generate_summary(), which returns the m x p matrix bx, the m x 1 vector by, their standard errors bxse/byse, and the true causal effects theta. Build an MR input object in the MendelianRandomization R package from bx/bxse and by/byse, then run one or more summary-data estimators (e.g. mr_ivw, mr_egger). Record the estimator name and causal effect estimate(s) to mr_estimates.csv, and interpret each estimate against theta from simmrd (e.g. default true_causal_effects = 0.3 per exposure). Because each generate_summary() call simulates exactly one replicate, repeat across replicate iterations (e.g. via the simmrd cli with pixi) to characterize bias or variability rather than relying on a single draw.

## Related tools

- **simmrd** (Generates the simulated GWAS summary replicate (bx, bxse, by, byse, theta) that serves as the benchmark input data) — https://github.com/noahlorinczcomi/simmrd
- **simmrd (R package)** (R interface for set_params()/generate_summary() to build parameter lists and emit the summary replicate) — https://github.com/noahlorinczcomi/simmrd
- **MendelianRandomization R package** (Consumes the simulated bx/bxse/by/byse via an MR input object and runs summary-data estimators such as IVW or MR-Egger)
- **R** (Runtime environment in which simmrd and MendelianRandomization are loaded and mr_estimates.csv is written)

## Examples

```
library(simmrd); d <- generate_summary(set_params()); mrdat <- mr_input(bx = d$bx, bxse = d$bxse, by = d$by, byse = d$byse); est <- mr_ivw(mrdat)
```

## Evaluation signals

- mr_estimates.csv exists and parses as valid CSV
- Every estimator record contains a finite numeric causal effect estimate
- Estimates can be compared against the theta element returned by simmrd for the same replicate (e.g. theta = 0.3 under default true_causal_effects)
- IV classification from simmrd (IVtype: valid/UHP/CHP) explains deviation of estimates from theta under pleiotropy settings
- Repeated single-replicate runs (e.g. via the cli's pixi simulate with --n iterations) yield a distribution of estimates rather than a single deterministic value

## Limitations

- Both simmrd::generate_individual() and simmrd::generate_summary() generate data for a single simulation replicate, so one run cannot characterize estimator bias or power
- Weak IV bias generally becomes more severe as more IVs are used, so few-IV MVMR simulations may unrealistically reflect optimal estimator performance
- Reported settings must match: independent simulations with nearly identical reported settings have produced conflicting results about the bias/power/Type I error of MR estimators (e.g., dIVW, MR-Robust, MR-RAPS)
- Pleiotropy must be configured explicitly (number_of_UHP_causal_SNPs, number_of_CHP_causal_SNPs default to 0); with defaults all IVs are valid and IVW performance will look optimal

## Evidence

- [intro] how simmr integrates with the popular MendelianRandomization R package (Yavorska & Burgess, 2017): "how simmr integrates with the popular MendelianRandomization R package (Yavorska & Burgess, 2017)"
- [readme] bx | m x p matrix of IV–exposure associations: [evidence span withheld — anchor status: external_doc]
- [readme] Both generate_summary and generate_individual generate simulated data for a single simulation replicate: "Both `simmrd::generate_individual()` and `simmrd::generate_summary()` generate simulated data for a **single** simulation replicate."
- [results] Figure 4 demonstrates integration of simmrd-generated summary data (three exposures) with the MendelianRandomization R package: [evidence span withheld — anchor status: not_found]
- [methods] Pass the list to generate_summary() or generate_individual(): "2. Pass the list to `generate_summary()` or `generate_individual()`."
