---
name: gaussian-rate-sampling-model-parameterization
description: Use when validating or benchmarking a kinetic-rate inference model (e.g.
  Nanodynamo/nano-ID) against ground-truth rates that must be known by construction,
  via controlled in silico simulation of RNA metabolic labelling data.
when_to_use_negative:
- The goal is to infer rates on real experimental Nanodynamo/nano-ID data (e.g. untreated
  or drug-treated TNBC cells) rather than validate the inference pipeline on synthetic
  data with known ground truth
- Only one replicate or a different labelling time is required — the benchmark design
  is specifically 2 replicates with a 20-min pulse
- The task requires quantifying RNA dynamics from Nanopore sequencing reads rather
  than simulating them
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3308
tools:
- name: Nanodynamo
  role: Provides the simulation framework (dataSimulationAnalysis.R, dataSimulationBash.sh,
    dataSimulationMain.R) used to sample gaussian-distributed kinetic rates, simulate
    RNA species expression, and fit the nano-ID model to the simulated dataset
  repo: github.com/mfurla/Nanodynamo
- name: INSPEcT
  role: Source of the 3T9 fibroblast median kinetic rates that anchor the gaussian
    distributions used for sampling the simulated genes
- name: R
  role: Execution environment for dataSimulationAnalysis.R and the supporting simulation
    scripts
  repo: github.com/mfurla/Nanodynamo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:eab99695531132e094f261ee3564767f11badba14e0dababdd4d80ae37be7e95
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gaussian-rate-sampling-model-parameterization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate a simulated benchmark RNA-dynamics dataset by sampling synthesis, processing, export and degradation rates for 1000 genes from gaussian distributions anchored to INSPEcT 3T9 fibroblast medians with a coefficient of variation of 5, then simulate RNA species expression for 2 replicates with a 20-min metabolic labelling pulse and noise matched to untreated RNA species CVs. It validates Nanodynamo's kinetic rate inference against known ground truth.

## When to use

Use when validating or benchmarking a kinetic-rate inference model (e.g. Nanodynamo/nano-ID) against ground-truth rates that must be known by construction, via controlled in silico simulation of RNA metabolic labelling data.

## When NOT to use

- The goal is to infer rates on real experimental Nanodynamo/nano-ID data (e.g. untreated or drug-treated TNBC cells) rather than validate the inference pipeline on synthetic data with known ground truth
- Only one replicate or a different labelling time is required — the benchmark design is specifically 2 replicates with a 20-min pulse
- The task requires quantifying RNA dynamics from Nanopore sequencing reads rather than simulating them

## Inputs

- regulated_genes_features.xls — rate initial-condition features required to seed the simulations
- CVs.rds — RNA species coefficient of variations from experimental (untreated) data used to set simulation noise
- dataSimulationAnalysis.R script from the Nanodynamo package (plus dataSimulationBash.sh / dataSimulationMain.R)

## Outputs

- Simulated expression levels for 1000 genes x 2 replicates (20-min labelling) with injected noise
- Ground-truth sampled kinetic rates and nano-ID-inferred kinetic rates for the simulated dataset
- Simulated folder in the Results directory reproducing Figures 1D, 1E, S4-5

## How to apply

Load the rate initial-condition features (regulated_genes_features.xls) to seed the simulations and the untreated RNA species CVs (CVs.rds) to set the noise level. Using dataSimulationAnalysis.R from the mfurla/Nanodynamo repository, sample synthesis, processing, export and degradation rates for 1000 genes from gaussian distributions anchored to the INSPEcT 3T9 medians with CV=5. Simulate RNA species expression levels for 2 replicates with a 20-min metabolic labelling time, injecting noise matched to the untreated-condition RNA species CVs. Fit the nano-ID/Nanodynamo model to the simulated data to obtain inferred kinetic rates, then write the simulated data and inferred rates to the Simulated folder of the Results directory as a consistency check against the deposited 'Simulated data and corresponding inferred rates' output.

## Related tools

- **Nanodynamo** (Provides the simulation framework (dataSimulationAnalysis.R, dataSimulationBash.sh, dataSimulationMain.R) used to sample gaussian-distributed kinetic rates, simulate RNA species expression, and fit the nano-ID model to the simulated dataset) — github.com/mfurla/Nanodynamo
- **INSPEcT** (Source of the 3T9 fibroblast median kinetic rates that anchor the gaussian distributions used for sampling the simulated genes)
- **R** (Execution environment for dataSimulationAnalysis.R and the supporting simulation scripts) — github.com/mfurla/Nanodynamo

## Examples

```
Rscript dataSimulationAnalysis.R  # within the Nanodynamo conda environment (condaEnvironment.yml); see also dataSimulationBash.sh
```

## Evaluation signals

- dataSimulationAnalysis.R executes without error and the Simulated folder of the Results directory is populated
- Spearman correlations between inferred and expected ground-truth kinetic rates fall in the 0.68-0.96 range reported in the paper
- Median Spearman correlations between simulated and inferred RNA species abundances exceed 0.98
- The Simulated folder contents are comparable to the deposited 'Simulated data and corresponding inferred rates' output
- Figures 1D, 1E, S4-5 are reproducible from the simulated data as a consistency check

## Limitations

- Simulation is parameterized for a specific design (1000 genes, 2 replicates, 20-min labelling pulse, CV=5 gaussian sampling); other designs would require re-deriving validation thresholds
- Rate sampling is anchored to INSPEcT 3T9 fibroblast medians, so simulations reflect that cell type's rate distributions and may not generalize to other biological systems
- The parameters identifiability was assessed separately via a Julia notebook (parametersIdentifiability.ipynb), suggesting some model parameters may not be uniquely inferable

## Evidence

- [methods] When Nanodynamo's inference pipeline was run on the simulated dataset (1000 genes, 2 replicates, 20-min labelling pulse), the inferred kinetic rates were well correlated with their expected ground-truth counterparts (Spearman coefficients in the 0.68-0.96 range): [evidence span withheld — anchor status: not_found]
- [methods] sample synthesis, processing, export and degradation rates for 1000 genes from gaussian distributions anchored to the INSPEcT 3T9 medians with a coefficient of variation of 5: [evidence span withheld — anchor status: not_found]
- [readme] README list of simulation and analysis scripts: "An R script to simulate genes with Nanodynamo and analyze the resulting dataset (dataSimulationMain.R)"
- [readme] README description of dataSimulationAnalysis.R figure outputs: "~ _dataSimulationAnalysis.R_: Figures 1D, 1E, S4-5, S12A, S37-38, S43-52."
- [methods] Methods listing of input files for simulations: "Data required to estimate rates initial conditions for simulations and models fit (regulated_genes_features.xls)"
