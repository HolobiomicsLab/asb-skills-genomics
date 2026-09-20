---
name: synthesis-processing-degradation-rate-inference
description: Use when you have metabolic-labelling (e.g. 20-min 4sU-like pulse) Nanopore
  native-RNA data from nuclear/cytoplasmic fractions or polysomes and need per-gene
  synthesis, processing, and degradation rates, or when designing how many replicates
  suffice for accurate rate inference.
when_to_use_negative:
- You only have short-read RNA-seq without metabolic labelling — Nanodynamo relies
  on RNA metabolic labelling plus Nanopore sequencing of native RNA from cellular
  fractions and polysomes
- You only need relative expression levels rather than kinetic rates — simpler quantification
  tools suffice without the kinetic model
- Isoform-level metabolism rather than gene-level synthesis/processing/degradation
  rates is the target — the companion nano-ID (birdumbrella/nano-ID) is designed for
  isoform dynamics
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Nanodynamo
  role: Experimental and computational workflow that fits the kinetic model to simulate
    genes and infer synthesis, processing, and degradation rates; scripts dataSimulationAnalysis.R,
    dataSimulationMain.R, and experimentalDataAnalysis.R implement this skill
  repo: github.com/mfurla/Nanodynamo
- name: R
  role: Language in which the Nanodynamo modelling, simulation, and correlation-analysis
    scripts are written and run
- name: INSPEcT
  role: Prior suite for kinetic-rate quantification used as a comparative benchmark
    for Nanodynamo rate estimates
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:0d20c0e6aaf8bb78c5ede0b9be4b2211b336101cad589fa298d4b466c7ed18c8
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
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# synthesis-processing-degradation-rate-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies per-gene RNA synthesis, processing, and degradation kinetic rates by fitting the Nanodynamo mathematical model to RNA metabolic labelling plus Nanopore native-RNA sequencing data from cellular fractions and polysomes. The skill also covers calibrating how many sequencing replicates are needed, since Nanodynamo's accuracy plateaus quickly with replicate number.

## When to use

Use when you have metabolic-labelling (e.g. 20-min 4sU-like pulse) Nanopore native-RNA data from nuclear/cytoplasmic fractions or polysomes and need per-gene synthesis, processing, and degradation rates, or when designing how many replicates suffice for accurate rate inference.

## When NOT to use

- You only have short-read RNA-seq without metabolic labelling — Nanodynamo relies on RNA metabolic labelling plus Nanopore sequencing of native RNA from cellular fractions and polysomes
- You only need relative expression levels rather than kinetic rates — simpler quantification tools suffice without the kinetic model
- Isoform-level metabolism rather than gene-level synthesis/processing/degradation rates is the target — the companion nano-ID (birdumbrella/nano-ID) is designed for isoform dynamics

## Inputs

- regulated_genes_features.xls (rate initial conditions for simulations and model fitting)
- Nanopore native-RNA sequencing data from cellular fractions and polysomes after RNA metabolic labelling
- Nanodynamo Anaconda environment (condaEnvironment.yml)

## Outputs

- Per-gene synthesis, processing, and degradation rates inferred by the Nanodynamo model
- Pearson correlation between inferred and real rates per replicate setting
- replicates_gain_curve.png (correlation gain vs. replicate number, Figures 1D/S4-5)

## How to apply

Load the kinetic-rate initial conditions for simulations and model fitting from regulated_genes_features.xls, then either fit the Nanodynamo model to experimental data (experimentalDataAnalysis.R / mainModelingResults.R) or generate in silico datasets with dataSimulationMain.R at a fixed 20-minute metabolic labelling time and 1–5 simulated replicates. For the replicate-optimisation variant, fit the model to each replicate setting, compute the Pearson correlation between Nanodynamo-inferred and real (true) rates, and plot the correlation gain versus replicate number (Figures 1D/S4-5), verifying that two replicates are already close to optimal with a maximum additional gain of ~20%. Validate by confirming correlations improve or saturate monotonically with replicate count and that inferred rates reproduce the real kinetic parameters per gene.

## Related tools

- **Nanodynamo** (Experimental and computational workflow that fits the kinetic model to simulate genes and infer synthesis, processing, and degradation rates; scripts dataSimulationAnalysis.R, dataSimulationMain.R, and experimentalDataAnalysis.R implement this skill) — github.com/mfurla/Nanodynamo
- **R** (Language in which the Nanodynamo modelling, simulation, and correlation-analysis scripts are written and run)
- **INSPEcT** (Prior suite for kinetic-rate quantification used as a comparative benchmark for Nanodynamo rate estimates)

## Examples

```
Rscript dataSimulationAnalysis.R
```

## Evaluation signals

- The correlation gain curve rises or saturates monotonically with replicate number, with two replicates already near-optimal (max additional gain ~20%)
- Pearson correlation between inferred and real (true) rates approaches the reported optimum at the chosen replicate number
- Fitted rates reproduce the initial conditions defined in regulated_genes_features.xls for simulated genes
- Output figures correspond to Figures 1D, 1E, S4-5 as produced by dataSimulationAnalysis.R

## Limitations

- Accuracy depends on the number of sequencing replicates: one replicate underperforms, and gains plateau beyond two replicates (max extra gain 20%)
- The labelling time is fixed at 20 minutes for this optimisation; other labelling times are not characterised by this analysis
- Identifiability of parameters was assessed separately (parametersIdentifiability.ipynb in Julia), implying some model parameters may not be uniquely identifiable

## Evidence
