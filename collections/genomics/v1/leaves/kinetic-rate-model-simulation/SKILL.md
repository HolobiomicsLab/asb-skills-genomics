---
name: kinetic-rate-model-simulation
description: Decision support for how many metabolic labelling pulses (20/60/120 min)
  are needed before kinetic rate inference, or for validating rate-estimation pipelines
  on simulated rather than experimental Nanodynamo data.
when_to_use_negative:
- Experimental Nanodynamo data are already available and the goal is rate estimation
  rather than design optimisation — use experimentalDataAnalysis.R/mainModelingResults.R
  instead of simulation
- The labelling protocol is fixed by the experiment and cannot be changed — the pulse-count
  sweep is only actionable when the design is still being chosen
- The question concerns post-inference coupling analysis or polysome profiling (couplings.R,
  polysomalProfiling.R), not kinetic-rate simulation
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0204
tools:
- name: Nanodynamo
  role: Simulation engine and kinetic model used to generate labelled/unlabelled RNA
    trajectories from inferred synthesis, processing, export, and degradation rates
  repo: https://github.com/mfurla/Nanodynamo
- name: R
  role: Language for the simulation and analysis scripts (dataSimulationMain.R, dataSimulationAnalysis.R,
    nascentSaturation.R)
  repo: https://github.com/mfurla/Nanodynamo
- name: INSPEcT
  role: Benchmark rate-estimation suite run on simulated data (INSPEcTAnalysisAllGenes.R)
    to contextualise Nanodynamo inference quality
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:d4eaf15cd03e8c9448b941bed9bbc297ed0b2d485fe768ef87b1ac7d850c3183
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# kinetic-rate-model-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulates labelled and unlabelled RNA abundance trajectories from Nanodynamo-inferred synthesis, processing, export, and degradation rates to evaluate kinetic-rate inference and metabolic-labelling pulse design (nascent RNA saturation).

## When to use

Decision support for how many metabolic labelling pulses (20/60/120 min) are needed before kinetic rate inference, or for validating rate-estimation pipelines on simulated rather than experimental Nanodynamo data.

## When NOT to use

- Experimental Nanodynamo data are already available and the goal is rate estimation rather than design optimisation — use experimentalDataAnalysis.R/mainModelingResults.R instead of simulation
- The labelling protocol is fixed by the experiment and cannot be changed — the pulse-count sweep is only actionable when the design is still being chosen
- The question concerns post-inference coupling analysis or polysome profiling (couplings.R, polysomalProfiling.R), not kinetic-rate simulation

## Inputs

- regulated_genes_features.xls (gene features and rate initial conditions for simulations and model fit)
- CVs.rds (RNA species coefficients of variation from experimental data)
- Nanodynamo-inferred synthesis, processing, export, and degradation rates (e.g. Untreated condition rates)
- condaEnvironment.yml (Anaconda environment for reproducibility)

## Outputs

- Simulated labelled and unlabelled RNA abundance trajectories per design point
- Nascent RNA saturation curves for 20-, 60-, and 120-minute pulse designs (Figure S3)
- Relative gain of multiple-pulse vs single-pulse designs (capped at ~15%)
- Single-pulse recovery of nuclear nascent species as fraction of steady state (83-95%)

## How to apply

Load gene features and rate initial conditions from regulated_genes_features.xls and RNA species coefficients of variation from CVs.rds to parametrize the kinetic model. Run the repo's dataSimulationMain.R/dataSimulationBash.sh to simulate genes with Nanodynamo and dataSimulationAnalysis.R to generate labelled/unlabelled RNA abundance trajectories from the inferred synthesis, processing, export, and degradation rates. For pulse-design optimisation, run nascentSaturation.R to sweep the number of labelling pulses at 20-, 60-, and 120-minute pulse durations, estimating nascent RNA species from the simulated fractions at each design point. Compare inferred nascent levels against steady-state ground truth per design, computing the relative gain of multiple-pulse over single-pulse designs and the fraction of steady-state nuclear nascent signal captured by a single 20-minute pulse. Render saturation curves and trade-off panels (Figure S3).

## Related tools

- **Nanodynamo** (Simulation engine and kinetic model used to generate labelled/unlabelled RNA trajectories from inferred synthesis, processing, export, and degradation rates) — https://github.com/mfurla/Nanodynamo
- **R** (Language for the simulation and analysis scripts (dataSimulationMain.R, dataSimulationAnalysis.R, nascentSaturation.R)) — https://github.com/mfurla/Nanodynamo
- **INSPEcT** (Benchmark rate-estimation suite run on simulated data (INSPEcTAnalysisAllGenes.R) to contextualise Nanodynamo inference quality)

## Examples

```
Rscript nascentSaturation.R  # sweep 20/60/120-min pulse designs; then Rscript dataSimulationAnalysis.R
```

## Evaluation signals

- Marginal performance gain from multiple pulses does not exceed 15% over a single 20-minute pulse
- A single 20-minute pulse recovers nuclear nascent species within the 83-95%-of-steady-state range
- Figure S3 (nascent saturation curves and trade-off panels) is produced from nascentSaturation.R
- Figures 1D, 1E, S4-5, S12A, S37-38, S43-52 are reproduced from dataSimulationAnalysis.R
- Simulated trajectories are consistent with rates seeded from regulated_genes_features.xls initial conditions and CVs.rds variability

## Limitations

- Conclusions are specific to simulated Nanodynamo datasets; a single 20-minute pulse captures only 83-95% of steady-state nuclear nascent signal — it is a trade-off, not full recovery
- Multiple-pulse designs offer at most ~15% performance gain, so extrapolating larger gains to other cell types or protocols is unsupported
- Simulation initial conditions depend on the Untreated-condition rates and RNA species CVs estimated from the triple-negative breast cancer dataset; other systems may require re-parameterisation

## Evidence

- [other] A single 20-minute labelling pulse is a good trade-off: multiple pulses yield only a minor performance gain (maximum 15%), and 20 minutes brings nuclear nascent RNA species to 83-95% of their steady-state value: [evidence span withheld — anchor status: not_found]
