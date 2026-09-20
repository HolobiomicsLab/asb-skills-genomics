---
name: simulation-based-model-benchmarking
description: Use when you need to validate that a kinetic rate-inference pipeline
  recovers known parameter values, by simulating data (e.g. 1000 genes, 2 replicates,
  20 min labelling pulse) and correlating inferred vs expected outputs.
when_to_use_negative:
- Input is real experimental data without ground-truth rates — use experimental data
  analysis workflows (experimentalDataAnalysis.R, mainModelingResults.R) instead
- You want to assess nano-ID read classification accuracy rather than rate-recovery
  accuracy — that is handled by nanoIDAccuracy.R
- The kinetic model under test has different rate types than synthesis/processing/degradation/export/translation,
  so the ground-truth correlations are not comparable
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3308
tools:
- name: Nanodynamo
  role: Mathematical modelling framework whose simulation scripts (dataSimulationMain.R,
    dataSimulationBash.sh) generate the simulated genes and whose model is fitted
    in the benchmark
  repo: github.com/mfurla/Nanodynamo
- name: R
  role: Execution environment; the benchmark itself is the dataSimulationAnalysis.R
    R script and output is written to CSV
- name: dataSimulationAnalysis.R
  role: R script to reproduce analyses on simulated data, producing Figures 1D, 1E,
    S4-5, S12A, S37-38, S43-52
  repo: github.com/mfurla/Nanodynamo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:445fa28532f13f8f41294ea21de2213e7c52fae4ef128fc1d44165c6ba5ba846
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# simulation-based-model-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a kinetic modelling pipeline by fitting it to simulated RNA species abundances and comparing modelled and inferred quantities against known ground truth. Used in Nanodynamo to validate inference of synthesis, processing, degradation, export and translation rates using Spearman correlations as goodness-of-fit metrics.

## When to use

Use when you need to validate that a kinetic rate-inference pipeline recovers known parameter values, by simulating data (e.g. 1000 genes, 2 replicates, 20 min labelling pulse) and correlating inferred vs expected outputs.

## When NOT to use

- Input is real experimental data without ground-truth rates — use experimental data analysis workflows (experimentalDataAnalysis.R, mainModelingResults.R) instead
- You want to assess nano-ID read classification accuracy rather than rate-recovery accuracy — that is handled by nanoIDAccuracy.R
- The kinetic model under test has different rate types than synthesis/processing/degradation/export/translation, so the ground-truth correlations are not comparable

## Inputs

- Simulated RNA species abundances for 1000 genes (from task_003 simulation output / Nanodynamo simulated data)
- Expected (ground-truth) kinetic rates: synthesis, processing, degradation, export, translation
- Simulation configuration: 2 replicates, 20 min labelling pulse
- regulated_genes_features.xls (data required to estimate rates initial conditions for simulations)

## Outputs

- simulated_inference_gof.csv with inferred rates and goodness-of-fit statistics
- Median per-gene Spearman correlation between modelled and expected RNA species abundances
- Per-rate-type Spearman coefficients between inferred and expected kinetic rates
- Figures equivalent to _dataSimulationAnalysis.R_: Figures 1D, 1E, S4-5, S12A, S37-38, S43-52

## How to apply

Load simulated RNA species abundances and expected kinetic rates (generated with the Nanodynamo simulation scripts, e.g. dataSimulationMain.R / dataSimulationBash.sh) into R. Run the dataSimulationAnalysis.R workflow from github.com/mfurla/Nanodynamo to fit the model and estimate synthesis, processing, degradation, export, and translation rates from the simulated data. Compute per-gene Spearman correlations between modelled and expected RNA species abundances and take their median as the first goodness-of-fit metric. Compute Spearman coefficients between inferred and expected kinetic rates for each rate type as the second metric. The rationale is that simulation provides ground truth for both expression levels and rates, which cannot be obtained from experimental data, so correlation thresholds directly quantify inference accuracy (in the paper: median abundance Spearman >0.98; rate Spearman 0.68-0.96 with 2 replicates and a 20 min 4sU-style labelling pulse).

## Related tools

- **Nanodynamo** (Mathematical modelling framework whose simulation scripts (dataSimulationMain.R, dataSimulationBash.sh) generate the simulated genes and whose model is fitted in the benchmark) — github.com/mfurla/Nanodynamo
- **R** (Execution environment; the benchmark itself is the dataSimulationAnalysis.R R script and output is written to CSV)
- **dataSimulationAnalysis.R** (R script to reproduce analyses on simulated data, producing Figures 1D, 1E, S4-5, S12A, S37-38, S43-52) — github.com/mfurla/Nanodynamo

## Examples

```
Rscript dataSimulationAnalysis.R  # fits Nanodynamo to simulated data (1000 genes, 2 replicates, 20 min pulse) and computes Spearman goodness-of-fit
```

## Evaluation signals

- Median Spearman correlation of modelled vs expected RNA species abundances exceeds 0.98
- Spearman coefficients between inferred and expected kinetic rates fall within the 0.68-0.96 range reported in the paper
- Simulation used the reference configuration: 1000 genes, 2 replicates, 20 min labelling pulse
- simulated_inference_gof.csv contains inferred rates for all five rate types (synthesis, processing, degradation, export, translation) plus goodness-of-fit statistics
- Results reproduce the figure panels assigned to _dataSimulationAnalysis.R_ (Figures 1D, 1E, S4-5, S12A, S37-38, S43-52)

## Limitations

- Rate-recovery accuracy (Spearman 0.68-0.96) is markedly lower than abundance recovery (>0.98), so inferred rates should be treated with caution for individual genes
- Benchmark performance depends on the simulation configuration (number of replicates, labelling pulse duration); 2 replicates with a 20 min pulse is the validated setting
- The repository notes parameters identifiability was evaluated separately (parametersIdentifiability.ipynb in Julia), indicating some parameters may not be uniquely identifiable

## Evidence

- [other] dataSimulationAnalysis.R workflow from the Nanodynamo repository fit the model and estimate synthesis, processing, degradation, export, and translation rates from the simulated data: [evidence span withheld — anchor status: not_found]
- [other] Simulated-data benchmark performance thresholds: [evidence span withheld — anchor status: not_found]
