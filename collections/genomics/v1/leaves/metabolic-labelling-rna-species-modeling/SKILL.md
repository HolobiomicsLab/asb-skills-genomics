---
name: metabolic-labelling-rna-species-modeling
description: Use when a kinetic-rate inference pipeline (e.g. nano-ID/Nanodynamo)
  must be validated against a simulated metabolic-labelling benchmark before applying
  it to experimental 4sU-labelled Nanopore data.
when_to_use_negative:
- The goal is to infer rates from experimental (not simulated) metabolic-labelling
  data — use the experimentalDataAnalysis.R / mainModelingResults.R workflow instead.
- The target method is INSPEcT quantification of synthesis/processing/degradation
  rates on its own (INSPEcTAnalysisAllGenes.R), rather than Nanodynamo simulation-based
  benchmarking.
- The study lacks RNA metabolic labelling data and only requires accuracy assessment
  of the nano-ID classifier itself (nanoIDAccuracy.R).
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Nanodynamo
  role: Simulation and mathematical-modelling framework; provides dataSimulationAnalysis.R
    and related scripts to generate the benchmark dataset and fit rates
  repo: github.com/mfurla/Nanodynamo
- name: nano-ID
  role: Machine-learning/model layer whose rate inference is validated against the
    simulated ground truth
  repo: birdumbrella/nano-ID
- name: R
  role: Execution environment for all Nanodynamo simulation and analysis scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:3a719498614c1e8ea2800cdd768a9f889d71c9dd5d14a94fa6c45e0a7a301131
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
  - nano-ID
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metabolic-labelling-rna-species-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate RNA-species expression datasets for metabolic-labelling experiments with Nanodynamo — sampling synthesis, processing, export and degradation rates from gaussian distributions anchored to INSPEcT 3T9 medians (1000 genes, 2 replicates, 20-min labelling, CV=5 noise) — then fit the nano-ID model to validate rate inference against ground truth.

## When to use

Use when a kinetic-rate inference pipeline (e.g. nano-ID/Nanodynamo) must be validated against a simulated metabolic-labelling benchmark before applying it to experimental 4sU-labelled Nanopore data.

## When NOT to use

- The goal is to infer rates from experimental (not simulated) metabolic-labelling data — use the experimentalDataAnalysis.R / mainModelingResults.R workflow instead.
- The target method is INSPEcT quantification of synthesis/processing/degradation rates on its own (INSPEcTAnalysisAllGenes.R), rather than Nanodynamo simulation-based benchmarking.
- The study lacks RNA metabolic labelling data and only requires accuracy assessment of the nano-ID classifier itself (nanoIDAccuracy.R).

## Inputs

- regulated_genes_features.xls (rate initial-condition features for simulations)
- CVs.rds (RNA species coefficient of variations from experimental data)
- dataSimulationAnalysis.R / dataSimulationMain.R / dataSimulationBash.sh scripts from mfurla/Nanodynamo

## Outputs

- Simulated RNA species expression levels for 1000 genes and 2 replicates (20-min labelling pulse)
- Inferred kinetic rates from the nano-ID model fitted to the simulated dataset
- Simulated folder contents reproducing Figures 1D, 1E, S4-5, S12A, S37-38, S43-52

## How to apply

Load the rate initial-condition features (regulated_genes_features.xls) and the untreated RNA species CVs (CVs.rds) from the Nanodynamo Files folder. Using dataSimulationAnalysis.R, sample synthesis, processing, export and degradation rates for 1000 genes from gaussian distributions anchored to the INSPEcT 3T9 medians with a coefficient of variation of 5. Simulate RNA species expression levels for 2 replicates with a 20-min metabolic labelling time, injecting noise matched to the untreated-condition CVs. Fit the nano-ID/Nanodynamo model to the simulated data to obtain inferred kinetic rates, and write simulated data and inferred rates to the Simulated folder of the Results directory, reproducing Figures 1D, 1E, S4-5 as a consistency check.

## Related tools

- **Nanodynamo** (Simulation and mathematical-modelling framework; provides dataSimulationAnalysis.R and related scripts to generate the benchmark dataset and fit rates) — github.com/mfurla/Nanodynamo
- **nano-ID** (Machine-learning/model layer whose rate inference is validated against the simulated ground truth) — birdumbrella/nano-ID
- **R** (Execution environment for all Nanodynamo simulation and analysis scripts)

## Examples

```
Rscript dataSimulationAnalysis.R
```

## Evaluation signals

- Spearman correlation between inferred and expected kinetic rates falls in the 0.68–0.96 range reported for the 1000-gene simulated benchmark.
- Median Spearman correlation between simulated and re-inferred RNA species abundances exceeds 0.98.
- dataSimulationAnalysis.R executes without error.
- The Results/Simulated folder contains simulated expression levels and inferred rates for 1000 genes comparable to the deposited 'Simulated data and corresponding inferred rates' output.
- Resulting plots reproduce Figures 1D, 1E and S4-5.

## Limitations

- Simulation parameters (gaussian rates anchored to INSPEcT 3T9 fibroblast medians, CV=5, 1000 genes, 2 replicates, 20-min pulse) are dataset-specific; other designs are not covered by this recipe.
- The reproducible Anaconda environment must be recreated via condaEnvironment.yml; deviations may break the R scripts.
- The Anaconda environment reproducibility is limited to the exact scripts shipped with the repository.

## Evidence
