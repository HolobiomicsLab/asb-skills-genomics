---
name: nano-id-model-rate-inference
description: Use when you have Nanopore native RNA sequencing of metabolic-labeling-pulsed
  samples (cellular fractions and/or polysomes) and need to quantify per-gene kinetic
  rates of the RNA life cycle or benchmark inference accuracy against ground truth.
when_to_use_negative:
- The goal is differential gene expression testing rather than kinetic rate quantification
  of RNA life-cycle steps
- Input data lack RNA metabolic labelling and Nanopore native RNA sequencing of cellular
  fractions/polysomes, so the kinetic model has no identifiability basis
edam_operation: http://edamontology.org/operation_3890
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Nanodynamo
  role: Experimental and computational workflow whose scripts (dataSimulationAnalysis.R,
    nanoIDMain.R, allInternalFunctions.R) run the data simulation, model fitting and
    rate inference
  repo: https://github.com/mfurla/Nanodynamo
- name: nano-ID
  role: Machine-learning-based method underlying the kinetic rate model fitted to
    labelled nanopore reads
  repo: https://github.com/birdumbrella/nano-ID
- name: INSPEcT
  role: Provides the 3T9 fibroblast median kinetic rates used to anchor the gaussian
    distributions sampled for the simulated dataset; also used in INSPEcTAnalysisAllGenes.R
    for comparison
- name: R
  role: Language in which the simulation, modelling and analysis scripts (dataSimulationAnalysis.R,
    nanoIDMain.R) are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:be2e65505e81d8aa0128dcc26394e99dd88f64fb2ddd0b183a368d6c6e617d89
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mfurla/Nanodynamo
  repo_url: https://github.com/mfurla/Nanodynamo
  tools:
  - INSPEcT
  - Nanodynamo
  - R
  - nano-ID
  edam_operation: http://edamontology.org/operation_3890
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# nano-ID model rate inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits the nano-ID kinetic model (via Nanodynamo) to RNA metabolic labelling plus native RNA Nanopore sequencing data to infer synthesis, processing, export, translation and degradation rates, and validates the inference on a simulated benchmark dataset (1000 genes, 2 replicates, 20-min 4sU pulse).

## When to use

Use when you have Nanopore native RNA sequencing of metabolic-labeling-pulsed samples (cellular fractions and/or polysomes) and need to quantify per-gene kinetic rates of the RNA life cycle or benchmark inference accuracy against ground truth.

## When NOT to use

- The goal is differential gene expression testing rather than kinetic rate quantification of RNA life-cycle steps
- Input data lack RNA metabolic labelling and Nanopore native RNA sequencing of cellular fractions/polysomes, so the kinetic model has no identifiability basis

## Inputs

- regulated_genes_features.xls (rate initial-condition features for simulations and model fit)
- CVs.rds (RNA species coefficient of variations from experimental data)
- Simulated or experimental nanopore native RNA sequencing data from cellular fractions and polysomes (2 replicates, 20-min labelling pulse for the benchmark)
- nano-ID model training data (nanoID.modification.probabilities.labeled.RData, nanoID.modification.probabilities.unlabeled.RData)

## Outputs

- Per-gene inferred kinetic rates (synthesis, processing, export, degradation) for 1000 genes
- Simulated RNA species expression levels and corresponding inferred rates written to the Simulated folder of the Results directory
- Figures 1D, 1E, S4-5 as consistency checks

## How to apply

Seed simulations with rate initial-condition features from regulated_genes_features.xls and set noise using the untreated RNA species CVs in CVs.rds. Using dataSimulationAnalysis.R from the Nanodynamo package, sample synthesis, processing, export and degradation rates for 1000 genes from gaussian distributions anchored to the INSPEcT 3T9 medians with a coefficient of variation of 5, then simulate RNA species expression for 2 replicates with a 20-min metabolic labelling time and noise matched to the CVs. Fit the nano-ID/Nanodynamo model to the simulated dataset to obtain inferred kinetic rates, and write simulated data and inferred rates to the Simulated folder of the Results directory. The same model-fitting machinery (nanoIDMain.R, nanoIDResults.rds) applies to experimental conditions such as Untreated, Pladienolide B, Leptomycin B and Harringtonine treatments.

## Related tools

- **Nanodynamo** (Experimental and computational workflow whose scripts (dataSimulationAnalysis.R, nanoIDMain.R, allInternalFunctions.R) run the data simulation, model fitting and rate inference) — https://github.com/mfurla/Nanodynamo
- **nano-ID** (Machine-learning-based method underlying the kinetic rate model fitted to labelled nanopore reads) — https://github.com/birdumbrella/nano-ID
- **INSPEcT** (Provides the 3T9 fibroblast median kinetic rates used to anchor the gaussian distributions sampled for the simulated dataset; also used in INSPEcTAnalysisAllGenes.R for comparison)
- **R** (Language in which the simulation, modelling and analysis scripts (dataSimulationAnalysis.R, nanoIDMain.R) are implemented)

## Examples

```
Rscript dataSimulationAnalysis.R
```

## Evaluation signals

- Inferred kinetic rates correlate well with expected ground-truth rates on the simulated dataset (Spearman coefficients in the 0.68-0.96 range)
- RNA species abundances show median Spearman correlations >0.98 between simulated/inferred and expected values
- dataSimulationAnalysis.R executes without error
- The Simulated folder contains simulated expression levels and inferred rates for 1000 genes comparable to the deposited 'Simulated data and corresponding inferred rates' output
- Reproducing Figures 1D, 1E, S4-5 serves as a consistency check

## Limitations

- Validation used simulated data with rates anchored to INSPEcT 3T9 fibroblast medians and CV=5 gaussian sampling; results on other organisms/conditions may differ
- The benchmark used a specific design (1000 genes, 2 replicates, 20-min labelling pulse); alternative labelling times or replicate counts are not covered by this validation
- Model fit relies on initial-condition estimates from regulated_genes_features.xls and nano-ID training-derived modification probabilities; poor inputs can degrade inference

## Evidence

- [other] nano-ID method description: [evidence span withheld — anchor status: external_doc]
