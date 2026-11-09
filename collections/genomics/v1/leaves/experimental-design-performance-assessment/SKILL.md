---
name: experimental-design-performance-assessment
description: Use when planning a metabolic labelling experiment with Nanodynamo and
  you need to decide how many labelling pulses and what duration will still yield
  accurate kinetic rate estimates.
when_to_use_negative:
- The goal is to quantify nano-ID labelling detection accuracy rather than experimental
  design — use nanoIDAccuracy.R (Figure S2) instead
- Experimental data are already collected and rates should be modelled directly with
  experimentalDataAnalysis.R / mainModelingResults.R rather than assessed by simulation
- The design question concerns experimental replicates or sequencing depth rather
  than the number and duration of metabolic labelling pulses
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0203
tools:
- name: Nanodynamo
  role: Kinetic model whose inferred synthesis, processing, export and degradation
    rates parametrize the simulated trajectories used for the design sweep
  repo: github.com/mfurla/Nanodynamo
- name: R
  role: Language used to run nascentSaturation.R (the nascent RNA saturation sweep)
    and dataSimulationAnalysis.R (the simulation engine)
- name: INSPEcT
  role: Prior suite for quantification of RNA synthesis, processing and degradation
    kinetic rates; context for the rate-inference framework being optimized
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:9a9fb7dd265d32acfd3358b977170e9df5f12744a7c09f2138a138e9edaff693
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# experimental-design-performance-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulates Nanodynamo kinetic-model data to assess how many metabolic labelling pulses and which pulse duration (20, 60, or 120 min) suffice for RNA kinetic rate inference, quantifying the marginal performance gain of multi-pulse over single-pulse designs.

## When to use

Use when planning a metabolic labelling experiment with Nanodynamo and you need to decide how many labelling pulses and what duration will still yield accurate kinetic rate estimates.

## When NOT to use

- The goal is to quantify nano-ID labelling detection accuracy rather than experimental design — use nanoIDAccuracy.R (Figure S2) instead
- Experimental data are already collected and rates should be modelled directly with experimentalDataAnalysis.R / mainModelingResults.R rather than assessed by simulation
- The design question concerns experimental replicates or sequencing depth rather than the number and duration of metabolic labelling pulses

## Inputs

- regulated_genes_features.xls — gene features used to estimate rates initial conditions for simulations
- CVs.rds — RNA species coefficients of variation from experimental data
- Rates and gene expression levels estimated with Nanodynamo for the Untreated condition (Untreated)
- dataSimulationAnalysis.R — simulation of labelled and unlabelled RNA abundance trajectories

## Outputs

- Nascent RNA saturation curves across pulse counts and durations (20, 60, 120 min)
- Relative performance gain of multiple-pulse vs. single 20-minute pulse designs
- Figure S3 (rendered by nascentSaturation.R)

## How to apply

Load gene features and rate initial conditions from regulated_genes_features.xls and RNA species coefficients of variation from CVs.rds to parametrize the kinetic model. Use the repo's dataSimulationAnalysis.R to simulate labelled and unlabelled RNA abundance trajectories from the inferred synthesis, processing, export, and degradation rates of the Untreated condition. Run nascentSaturation.R to sweep the number of labelling pulses at 20-, 60-, and 120-minute durations, estimating nascent RNA species from simulated fractions at each design point. Compare inferred nascent levels against steady-state ground truth per design, computing the relative gain of multiple-pulse over single-pulse designs and the steady-state fraction captured by a single 20-minute pulse, then render saturation curves as Figure S3. The rationale is that 20 minutes brings nuclear nascent RNA to 83–95% of steady state — informative, but far from both steady state and the initial exponential regime — so the sweep tests whether extra pulses compensate.

## Related tools

- **Nanodynamo** (Kinetic model whose inferred synthesis, processing, export and degradation rates parametrize the simulated trajectories used for the design sweep) — github.com/mfurla/Nanodynamo
- **R** (Language used to run nascentSaturation.R (the nascent RNA saturation sweep) and dataSimulationAnalysis.R (the simulation engine))
- **INSPEcT** (Prior suite for quantification of RNA synthesis, processing and degradation kinetic rates; context for the rate-inference framework being optimized)

## Examples

```
Rscript nascentSaturation.R
```

## Evaluation signals

- The marginal performance gain from multiple pulses over a single 20-minute pulse does not exceed 15% (maximum observed gain)
- A single 20-minute pulse recovers nuclear nascent RNA species within the 83–95%-of-steady-state range
- Figure S3 is produced with saturation curves comparing 20-, 60-, and 120-minute pulse designs
- Simulated labelled and unlabelled RNA fraction trajectories are generated from the Untreated-condition Nanodynamo rates, not arbitrary parameters
- Inferred nascent levels are compared per design against steady-state ground truth rather than only against other designs

## Limitations

- Conclusions are drawn from simulated Nanodynamo datasets parametrized by Untreated-condition rates; other conditions or cell types may shift the optimal pulse design
- A 20-minute pulse leaves nuclear nascent RNA far from both steady state and the initial exponential regime, so single-pulse estimates are inherently approximate (83–95% of steady state)
- The assessment caps the benefit of added pulses at a 15% gain, so multi-pulse designs should not be expected to resolve substantially more information than a single 20-minute pulse

## Evidence

- [supplementary] A single 20-minute labelling pulse is a good trade-off: multiple pulses yield only a minor performance gain (maximum 15%), and 20 minutes brings nuclear nascent RNA species to 83-95% of their steady-state value: [evidence span withheld — anchor status: not_found]
- [readme] An R script to investigate nascent RNA saturation with simulated data (nascentSaturation.R): "An R script to investigate nascent RNA saturation with simulated data (nascentSaturation.R)"
- [methods] Data required to estimate rates initial conditions for simulations and models fit (regulated_genes_features.xls): "Data required to estimate rates initial conditions for simulations and models fit (regulated_genes_features.xls)"
- [methods] RNA species CVs (CVs.rds).: "RNA species CVs (CVs.rds)."
- [methods] ~ _nascentSaturation.R_: Figure S3.: "~ _nascentSaturation.R_: Figure S3."
