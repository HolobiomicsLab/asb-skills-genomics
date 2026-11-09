---
name: nascent-rna-saturation-analysis
description: Decision support for choosing a metabolic labelling pulse design (single
  vs multiple pulses, 20/60/120 min) before running Nanodynamo kinetic-rate inference
  on Nanopore native-RNA data.
when_to_use_negative:
- The goal is inference of synthesis, processing, export, or degradation rates themselves
  rather than experimental-design of the labelling pulse schedule.
- The RNA species of interest are already far into the exponential-labelling or steady-state
  regime, where pulse-duration saturation behaviour is not the limiting design factor.
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Nanodynamo
  role: Provides the kinetic model and simulation machinery whose rates parametrize
    the pulse-design sweep
  repo: https://github.com/mfurla/Nanodynamo
- name: R
  role: Language in which nascentSaturation.R and dataSimulationAnalysis.R are implemented
    and executed
  repo: https://github.com/mfurla/Nanodynamo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:76bb5fde27b7df2fa9639fe13e29cf9ab12ecbeafdce88663c17a6e14a7f2b4a
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
  - Nanodynamo
  - R
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# nascent-rna-saturation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A simulation-based experimental-design analysis for Nanodynamo metabolic labelling that sweeps the number and duration (20, 60, 120 min) of labelling pulses to assess how much nascent RNA signal and kinetic-inference performance is gained over a single 20-minute pulse.

## When to use

Decision support for choosing a metabolic labelling pulse design (single vs multiple pulses, 20/60/120 min) before running Nanodynamo kinetic-rate inference on Nanopore native-RNA data.

## When NOT to use

- The goal is inference of synthesis, processing, export, or degradation rates themselves rather than experimental-design of the labelling pulse schedule.
- The RNA species of interest are already far into the exponential-labelling or steady-state regime, where pulse-duration saturation behaviour is not the limiting design factor.

## Inputs

- regulated_genes_features.xls (gene features for rate initial conditions in simulations)
- CVs.rds (RNA species coefficients of variation)
- Simulated labelled/unlabelled RNA abundance trajectories from Nanodynamo (dataSimulationAnalysis.R)

## Outputs

- Saturation curves of nascent RNA recovery vs number and duration of labelling pulses (Figure S3)
- Relative performance gain of multiple-pulse vs single 20-minute pulse designs
- Fraction of steady-state nuclear nascent signal captured per pulse design

## How to apply

Parametrize the kinetic model with gene features from regulated_genes_features.xls (initial conditions for synthesis, processing, export, and degradation rates) and RNA species coefficients of variation from CVs.rds. Simulate labelled and unlabelled RNA abundance trajectories with the repo's dataSimulationAnalysis.R, then run nascentSaturation.R to sweep the number of labelling pulses at 20-, 60-, and 120-minute durations and estimate nascent RNA species from simulated fractions at each design point. Compare inferred nascent levels against steady-state ground truth per design, computing the relative performance gain of multiple-pulse over single-pulse designs and the fraction of steady-state nuclear nascent signal captured by one 20-minute pulse. Render saturation curves and trade-off panels as Figure S3; the expected conclusion is that multiple pulses yield at most ~15% marginal gain while a single 20-minute pulse brings nuclear nascent RNA to 83-95% of steady state, making 20 min the chosen trade-off.

## Related tools

- **Nanodynamo** (Provides the kinetic model and simulation machinery whose rates parametrize the pulse-design sweep) — https://github.com/mfurla/Nanodynamo
- **R** (Language in which nascentSaturation.R and dataSimulationAnalysis.R are implemented and executed) — https://github.com/mfurla/Nanodynamo

## Examples

```
Rscript Scripts/nascentSaturation.R
```

## Evaluation signals

- Marginal performance gain from multiple pulses does not exceed 15% over a single pulse.
- A single 20-minute pulse recovers nuclear nascent RNA species within the 83-95%-of-steady-state range.
- Figure S3 is produced with saturation curves at 20-, 60-, and 120-minute pulse durations.
- Inferred nascent levels are compared per design against the steady-state ground truth from the simulated kinetics.

## Limitations

- Conclusions are derived from simulated Nanodynamo datasets (simulated fractions, ground truth known); gains on real experimental data may differ.
- The 20-minute design point sits far from both steady state and the initial exponential-labelling regime, so very short-pulse designs are not captured by the same linear regime assumptions.
- Results apply to nuclear nascent RNA species quantified by Nanodynamo-style metabolic labelling and Nanopore native-RNA sequencing, not to other labelling quantification modalities.

## Evidence
