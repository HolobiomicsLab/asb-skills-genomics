---
name: rna-kinetics-simulation
description: 'Use when validating or benchmarking a kinetic-rate inference pipeline
  for RNA metabolic labelling data: you need synthetic ground-truth synthesis/processing/degradation
  rates to measure inference accuracy (Spearman correlations) or to tune experimental
  design (e.g. number of replicates).'
when_to_use_negative:
- The goal is to analyse real experimental Nanopore reads from metabolic labelling
  experiments — use the experimentalDataAnalysis.R / nanoIDMain.R workflows instead
  of the simulation pipeline.
- Only a single sequencing replicate is available and near-optimal rate inference
  is required — the gain curve shows one sample falls up to ~20% short of the 2-replicate
  near-optimal performance.
- The kinetic rates of interest are outside the Nanodynamo model scope (birth through
  processing, export, translation and degradation) or the labelling time must differ
  substantially from the validated 20-min pulse without re-validation.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3308
tools:
- name: Nanodynamo
  role: Framework whose dataSimulationAnalysis.R pipeline generates the in silico
    datasets and fits the kinetic model to infer synthesis, processing, degradation,
    export and translation rates
  repo: github.com/mfurla/Nanodynamo
- name: INSPEcT
  role: Source of the 3T9 fibroblast median kinetic rates used to anchor the simulated
    rate distributions
- name: R
  role: Language in which the simulation and analysis scripts (dataSimulationAnalysis.R,
    dataSimulationMain.R) are implemented
- name: nano-ID model
  role: Machine-learning/model layer whose inference is benchmarked on the simulated
    ground-truth dataset
  repo: birdumbrella/nano-ID
provenance:
  source_task_ids:
  - task_006
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:a3c3d361da8b2d8d64100bdc09057aa926386f80804661b7e7745bb56f738bec
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
  - dataSimulationAnalysis.R
  - nano-ID model
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# rna-kinetics-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulates in silico RNA metabolic labelling datasets with Nanodynamo (1000 genes, rates anchored to INSPEcT 3T9 fibroblast medians, CV=5 gaussian sampling, 20-min 4sU pulse) and fits the kinetic model to benchmark inference of synthesis, processing, export and degradation rates against ground truth.

## When to use

Use when validating or benchmarking a kinetic-rate inference pipeline for RNA metabolic labelling data: you need synthetic ground-truth synthesis/processing/degradation rates to measure inference accuracy (Spearman correlations) or to tune experimental design (e.g. number of replicates).

## When NOT to use

- The goal is to analyse real experimental Nanopore reads from metabolic labelling experiments — use the experimentalDataAnalysis.R / nanoIDMain.R workflows instead of the simulation pipeline.
- Only a single sequencing replicate is available and near-optimal rate inference is required — the gain curve shows one sample falls up to ~20% short of the 2-replicate near-optimal performance.
- The kinetic rates of interest are outside the Nanodynamo model scope (birth through processing, export, translation and degradation) or the labelling time must differ substantially from the validated 20-min pulse without re-validation.

## Inputs

- regulated_genes_features.xls (rates initial conditions for simulations and model fitting)
- CVs.rds (untreated-condition RNA species CVs used to set noise level)
- dataSimulationAnalysis.R / dataSimulationBash.sh / dataSimulationMain.R scripts from the Nanodynamo repository

## Outputs

- Simulated expression levels and expected kinetic rates for 1000 genes (written to the Simulated folder of the Results directory)
- Inferred synthesis, processing, degradation, export and translation rates from model fitting
- Spearman correlation statistics between inferred and expected rates (e.g. simulated_inference_gof.csv)
- Replicate-number gain curve (replicates_gain_curve.png; Figures 1D, S4-5)

## How to apply

Load the rates initial-condition features from regulated_genes_features.xls and the untreated RNA species CVs from CVs.rds, then use the dataSimulationAnalysis.R workflow from the Nanodynamo repository (github.com/mfurla/Nanodynamo) to sample synthesis, processing, export and degradation rates for 1000 genes from gaussian distributions anchored to INSPEcT 3T9 medians with coefficient of variation 5, injecting noise matched to the untreated-condition CVs. Simulate RNA species expression for the desired design (default 2 replicates, 20-min metabolic labelling pulse), fit the nano-ID/Nanodynamo model to each simulated dataset, and compare inferred rates to expected values via Spearman coefficients. To optimise experimental design, repeat with 1–5 simulated replicates at fixed 20-min labelling and plot the correlation gain (Figures 1D, S4-5): two replicates already approach optimal performance, with a maximum additional gain of ~20%.

## Related tools

- **Nanodynamo** (Framework whose dataSimulationAnalysis.R pipeline generates the in silico datasets and fits the kinetic model to infer synthesis, processing, degradation, export and translation rates) — github.com/mfurla/Nanodynamo
- **INSPEcT** (Source of the 3T9 fibroblast median kinetic rates used to anchor the simulated rate distributions)
- **R** (Language in which the simulation and analysis scripts (dataSimulationAnalysis.R, dataSimulationMain.R) are implemented)
- **nano-ID model** (Machine-learning/model layer whose inference is benchmarked on the simulated ground-truth dataset) — birdumbrella/nano-ID

## Examples

```
Rscript Scripts/dataSimulationAnalysis.R  # simulate 1000 genes (2 replicates, 20-min pulse) from regulated_genes_features.xls + CVs.rds and benchmark inferred vs expected rates
```

## Evaluation signals

- dataSimulationAnalysis.R executes without error and the Simulated folder contains simulated expression levels and inferred rates for 1000 genes comparable to the deposited 'Simulated data and corresponding inferred rates' output
- Median Spearman correlation between modelled and expected RNA species abundances exceeds 0.98
- Spearman coefficients between inferred and expected kinetic rates fall within the 0.68–0.96 range reported in the paper
- Replicate gain curve shows monotonically improving (or saturating) correlation with replicate count, with two replicates remarkably close to optimal and a maximum additional increase of 20%
- Simulated rates are drawn from gaussian distributions anchored to INSPEcT 3T9 medians with coefficient of variation 5, and injected noise matches the untreated-condition RNA species CVs

## Limitations

- Benchmark validity depends on simulated rates following gaussian distributions (CV=5) anchored to INSPEcT 3T9 fibroblast medians; genes outside this rate regime may not be represented
- With a single replicate, inference performance is up to ~20% below the near-optimal (≥2 replicate) performance
- The validated design uses a 20-minute metabolic labelling pulse; other labelling times are not covered by this specific benchmark
- Parameter identifiability is a known concern and is evaluated separately via the parametersIdentifiability Julia notebook in the repository

## Evidence
