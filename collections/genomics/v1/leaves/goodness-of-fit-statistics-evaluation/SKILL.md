---
name: goodness-of-fit-statistics-evaluation
description: Use when benchmarking an RNA dynamics inference method on simulated data
  and you need quantitative goodness-of-fit metrics for recovered abundances and kinetic
  rates (synthesis, processing, degradation, export, translation).
when_to_use_negative:
- Input is real experimental data (e.g. Untreated or PladienolideB condition rates)
  rather than simulated data with known ground-truth abundances and rates, so no expected
  values exist for correlation.
- The goal is to evaluate nano-ID or INSPEcT rate estimates with their own scripts
  (nanoIDAccuracy.R, INSPEcTAnalysisAllGenes.R) rather than the Nanodynamo simulation
  benchmark.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Nanodynamo
  role: Modelling framework whose simulation and inference pipeline is being benchmarked;
    dataSimulationAnalysis.R fits the model to simulated genes and produces the inferred
    rates to be scored
  repo: github.com/mfurla/Nanodynamo
- name: dataSimulationAnalysis.R
  role: R script in the Nanodynamo repository used to reproduce the analyses on simulated
    data (Figures 1D, 1E, S4-5, S12A, S37-38, S43-52), including fitting simulated
    genes and computing goodness-of-fit
  repo: github.com/mfurla/Nanodynamo
- name: R
  role: Language used to run the workflow, compute Spearman correlations, and write
    simulated_inference_gof.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:b707a9ba305b2618d49556472c61e8e69b97e998f43e3b7110f02281c50e02dc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mfurla/Nanodynamo
  tools:
  - Nanodynamo
  - R
  - dataSimulationAnalysis.R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# goodness-of-fit-statistics-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates how well Nanodynamo's inference pipeline recovers RNA species abundances and kinetic rates by computing per-gene Spearman correlations between modelled and expected abundances (median >0.98 required) and Spearman coefficients between inferred and expected kinetic rates (0.68-0.96 range) on simulated data.

## When to use

Use when benchmarking an RNA dynamics inference method on simulated data and you need quantitative goodness-of-fit metrics for recovered abundances and kinetic rates (synthesis, processing, degradation, export, translation).

## When NOT to use

- Input is real experimental data (e.g. Untreated or PladienolideB condition rates) rather than simulated data with known ground-truth abundances and rates, so no expected values exist for correlation.
- The goal is to evaluate nano-ID or INSPEcT rate estimates with their own scripts (nanoIDAccuracy.R, INSPEcTAnalysisAllGenes.R) rather than the Nanodynamo simulation benchmark.

## Inputs

- Simulated RNA species abundances from task_003 simulation output (1000 genes, 2 replicates, 20 min labelling pulse)
- Expected (ground-truth) kinetic rates from the simulation output
- R environment with Nanodynamo scripts (dataSimulationAnalysis.R)

## Outputs

- simulated_inference_gof.csv containing inferred rates and goodness-of-fit statistics (median Spearman correlation of abundances; per-rate-type Spearman coefficients)
- Per-gene Spearman correlations between modelled and expected RNA species abundances
- Per-rate-type Spearman coefficients between inferred and expected kinetic rates

## How to apply

Load the simulated RNA species abundances and expected kinetic rates produced by the task_003 simulation (1000 genes, 2 replicates, 20 min labelling pulse) into R. Run the dataSimulationAnalysis.R script from the mfurla/Nanodynamo repository, which fits the Nanodynamo model and estimates synthesis, processing, degradation, export, and translation rates from the simulated data. For each gene, compute the Spearman correlation between modelled and expected RNA species abundances and take the median across genes as the first goodness-of-fit metric; acceptance requires this median to exceed 0.98. For each rate type, compute the Spearman coefficient between inferred and expected kinetic rates; these should fall within the 0.68-0.96 range reported in the paper. Write the inferred rates and the goodness-of-fit statistics to simulated_inference_gof.csv for downstream validation.

## Related tools

- **Nanodynamo** (Modelling framework whose simulation and inference pipeline is being benchmarked; dataSimulationAnalysis.R fits the model to simulated genes and produces the inferred rates to be scored) — github.com/mfurla/Nanodynamo
- **dataSimulationAnalysis.R** (R script in the Nanodynamo repository used to reproduce the analyses on simulated data (Figures 1D, 1E, S4-5, S12A, S37-38, S43-52), including fitting simulated genes and computing goodness-of-fit) — github.com/mfurla/Nanodynamo
- **R** (Language used to run the workflow, compute Spearman correlations, and write simulated_inference_gof.csv)

## Examples

```
Rscript dataSimulationAnalysis.R
```

## Evaluation signals

- Median per-gene Spearman correlation between modelled and expected RNA species abundances exceeds 0.98.
- Spearman coefficients between inferred and expected kinetic rates fall within the 0.68-0.96 range reported in the paper for each rate type.
- simulated_inference_gof.csv is produced and contains both inferred rates and goodness-of-fit statistics.
- Per-gene correlation values are computable for all 1000 simulated genes (2 replicates, 20 min labelling pulse).

## Limitations

- The acceptance thresholds (abundance median rho >0.98; rate rho 0.68-0.96) are calibrated for the paper's simulation setup of 1000 genes, 2 replicates and a 20 min labelling pulse; other simulation settings may yield different expected ranges.
- Kinetic rates are recovered less accurately (rho 0.68-0.96) than abundances (rho >0.98), so rate-level comparisons are inherently noisier.
- This skill evaluates the simulation pipeline (dataSimulationAnalysis.R, Figures 1D, 1E, S4-5, S12A, S37-38, S43-52) and not the experimental data analyses (experimentalDataAnalysis.R / mainModelingResults.R).

## Evidence

- [results] On simulated data with 2 replicates and a 20 min labelling pulse, modelled RNA species abundances matched expected levels with median Spearman correlations >0.98, and inferred kinetic rates correlated with expected values with Spearman coefficients in the 0.68-0.96 range.: [evidence span withheld — anchor status: not_found]
