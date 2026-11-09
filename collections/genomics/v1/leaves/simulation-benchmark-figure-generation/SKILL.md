---
name: simulation-benchmark-figure-generation
description: 'Use when validating a kinetic-rate inference pipeline on synthetic data:
  you need a simulated RNA-life-cycle benchmark with known ground-truth synthesis/processing/export/degradation
  rates and corresponding benchmark figures.'
when_to_use_negative:
- The goal is analysing real Nanodynamo experimental data (untreated, Pladienolide
  B, Leptomycin B, Harringtonine conditions) rather than benchmarking inference —
  use experimentalDataAnalysis.R / mainModelingResults.R instead.
- The aim is to quantify RNA dynamics in samples without ground-truth kinetic rates;
  simulation is only a validation exercise, not a measurement method.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3524
tools:
- name: Nanodynamo
  role: Provides the simulation scripts (dataSimulationBash.sh, dataSimulationMain.R,
    dataSimulationAnalysis.R) that generate the simulated gene set and produce the
    benchmark figures
  repo: github.com/mfurla/Nanodynamo
- name: nano-ID model
  role: Model fitted to the simulated dataset to produce inferred kinetic rates for
    benchmarking against ground truth
  repo: birdumbrella/nano-ID
- name: INSPEcT
  role: Source of 3T9 fibroblast median kinetic rates used to anchor the gaussian
    sampling distributions for simulated genes
- name: R
  role: Language in which the simulation and analysis scripts (dataSimulationMain.R,
    dataSimulationAnalysis.R) are implemented and run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:49954d3831863325894183e5a0b07796c5a87038b8f9d570d0a1e1c500d0adcb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mfurla/Nanodynamo
  tools:
  - INSPEcT
  - Nanodynamo
  - R
  - nano-ID model
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# simulation-benchmark-figure-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates and analyzes a simulated Nanodynamo benchmark dataset (1000 genes, 2 replicates, 20-min 4U metabolic labelling) with rates sampled from gaussian distributions anchored to INSPEcT 3T9 fibroblast medians (CV=5), to validate kinetic-rate inference and reproduce Figures 1D, 1E, S4-5.

## When to use

Use when validating a kinetic-rate inference pipeline on synthetic data: you need a simulated RNA-life-cycle benchmark with known ground-truth synthesis/processing/export/degradation rates and corresponding benchmark figures.

## When NOT to use

- The goal is analysing real Nanodynamo experimental data (untreated, Pladienolide B, Leptomycin B, Harringtonine conditions) rather than benchmarking inference — use experimentalDataAnalysis.R / mainModelingResults.R instead.
- The aim is to quantify RNA dynamics in samples without ground-truth kinetic rates; simulation is only a validation exercise, not a measurement method.

## Inputs

- regulated_genes_features.xls (rate initial-condition features for simulations)
- CVs.rds (RNA species coefficients of variation from untreated experimental data)
- dataSimulationBash.sh, dataSimulationMain.R, dataSimulationAnalysis.R (Nanodynamo Scripts folder)

## Outputs

- Simulated expression levels and ground-truth kinetic rates for 1000 genes (2 replicates, 20-min labelling)
- Inferred kinetic rates from the nano-ID/Nanodynamo model fitted to the simulated data
- Simulated folder contents in the Results directory reproducing Figures 1D, 1E, S4-5, S12A, S37-38, S43-52

## How to apply

In R, first load the rate initial-condition features (regulated_genes_features.xls) and the untreated RNA species coefficient of variations (CVs.rds) that set the noise level. Using dataSimulationMain.R (launched via dataSimulationBash.sh) and dataSimulationAnalysis.R from the Nanodynamo package, sample synthesis, processing, export and degradation rates for 1000 genes from gaussian distributions anchored to the INSPEcT 3T9 medians with a coefficient of variation of 5. Simulate RNA species expression levels for 2 replicates with a 20-min metabolic labelling time, injecting noise matched to the untreated-condition RNA species CVs. Fit the nano-ID/Nanodynamo model to the simulated data to obtain inferred kinetic rates, write both to the Simulated folder of the Results directory, and generate Figures 1D, 1E, S4-5 as a consistency check against Spearman correlations of 0.68-0.96 (rates) and >0.98 (RNA species abundances).

## Related tools

- **Nanodynamo** (Provides the simulation scripts (dataSimulationBash.sh, dataSimulationMain.R, dataSimulationAnalysis.R) that generate the simulated gene set and produce the benchmark figures) — github.com/mfurla/Nanodynamo
- **nano-ID model** (Model fitted to the simulated dataset to produce inferred kinetic rates for benchmarking against ground truth) — birdumbrella/nano-ID
- **INSPEcT** (Source of 3T9 fibroblast median kinetic rates used to anchor the gaussian sampling distributions for simulated genes)
- **R** (Language in which the simulation and analysis scripts (dataSimulationMain.R, dataSimulationAnalysis.R) are implemented and run)

## Examples

```
bash dataSimulationBash.sh && Rscript dataSimulationAnalysis.R
```

## Evaluation signals

- dataSimulationAnalysis.R executes without error and writes to the Simulated folder of the Results directory.
- Inferred kinetic rates correlate with expected ground-truth values with Spearman coefficients in the 0.68-0.96 range.
- Simulated vs. inferred RNA species abundances show median Spearman correlations >0.98.
- The Simulated folder contains expression levels and inferred rates for 1000 genes comparable to the deposited 'Simulated data and corresponding inferred rates' output.
- Generated figures match Figures 1D, 1E, S4-5 (plus S12A, S37-38, S43-52) from the published manuscript.

## Limitations

- Simulation parameters (1000 genes, CV=5 gaussian sampling, 20-min pulse, 2 replicates) are anchored to INSPEcT 3T9 fibroblast medians, so ground-truth realism depends on how well those medians match the system under study.
- Correlations are benchmark-quality but not perfect (Spearman 0.68-0.96 for rates), so inference accuracy on real data may be lower than on this synthetic benchmark.
- The Anaconda environment must be reproduced via the repository's condaEnvironment.yml for the scripts to run as published.

## Evidence
