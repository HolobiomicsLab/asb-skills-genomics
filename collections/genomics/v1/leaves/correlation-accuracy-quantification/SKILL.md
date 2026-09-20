---
name: correlation-accuracy-quantification
description: Decision support for how many sequencing replicates are needed for near-optimal
  Nanodynamo rate inference, e.g. when planning metabolic-labelling experiments or
  validating rate-estimation accuracy on simulated data.
when_to_use_negative:
- The goal is to model real experimental Nanopore data rather than simulated data
  — use experimentalDataAnalysis.R / mainModelingResults.R instead of the dataSimulationAnalysis.R
  pipeline.
- The question is nascent RNA saturation or labelling-time optimisation rather than
  replicate-number optimisation — use nascentSaturation.R.
- The analysis is an accuracy comparison between methods (e.g. nano-ID vs Nanodynamo)
  rather than replicate-count optimisation.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Nanodynamo
  role: Simulates in silico datasets with 1–5 replicates at 20-min labelling, fits
    the kinetic model, and provides the inferred vs real rates correlated in the gain
    curve
  repo: https://github.com/mfurla/Nanodynamo
- name: R
  role: Language in which the dataSimulationAnalysis.R pipeline reproduces the replicate
    gain analysis (Figures 1D, 1E, S4-5)
  repo: https://github.com/mfurla/Nanodynamo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:1ed82a8ae66a43c0528c0133f0728d6b19badcb500b3a0e9d9faf39336755025
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
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# correlation-accuracy-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify Nanodynamo rate-inference accuracy by simulating sequencing datasets with 1–5 replicates at a fixed 20-minute metabolic labelling time and plotting the Pearson correlation gain between inferred and real synthesis, processing, and degradation rates.

## When to use

Decision support for how many sequencing replicates are needed for near-optimal Nanodynamo rate inference, e.g. when planning metabolic-labelling experiments or validating rate-estimation accuracy on simulated data.

## When NOT to use

- The goal is to model real experimental Nanopore data rather than simulated data — use experimentalDataAnalysis.R / mainModelingResults.R instead of the dataSimulationAnalysis.R pipeline.
- The question is nascent RNA saturation or labelling-time optimisation rather than replicate-number optimisation — use nascentSaturation.R.
- The analysis is an accuracy comparison between methods (e.g. nano-ID vs Nanodynamo) rather than replicate-count optimisation.

## Inputs

- regulated_genes_features.xls (rates initial conditions for simulations and models fit)
- Simulated Nanodynamo datasets at fixed 20-min metabolic labelling time with 1, 2, 3, 4, and 5 replicates
- dataSimulationAnalysis.R pipeline from github.com/mfurla/Nanodynamo

## Outputs

- Pearson correlations between Nanodynamo-inferred and real synthesis, processing, and degradation rates per replicate number
- Replicate gain curve (replicates_gain_curve.png) reproducing Figures 1D and S4-5
- Conclusion that two replicates are sufficient for near-optimal performance (max additional gain ~20%)

## How to apply

Load the rates initial conditions for simulations and model fitting from regulated_genes_features.xls to define the simulated genes' kinetic parameters. Use the dataSimulationAnalysis.R pipeline from the mfurla/Nanodynamo repository (run via dataSimulationBash.sh / dataSimulationMain.R) to generate in silico datasets at a fixed 20-minute metabolic labelling time with 1, 2, 3, 4, and 5 simulated replicates. Fit the Nanodynamo kinetic model to each simulated dataset to infer synthesis, processing, and degradation rates per replicate setting, then compute the Pearson correlation between inferred and real (true) rates for each replicate number. Plot the correlation gain as a function of replicate number (reproducing Figures 1D and S4-5), where two replicates already approach optimal performance with a maximum additional increase of about 20%.

## Related tools

- **Nanodynamo** (Simulates in silico datasets with 1–5 replicates at 20-min labelling, fits the kinetic model, and provides the inferred vs real rates correlated in the gain curve) — https://github.com/mfurla/Nanodynamo
- **R** (Language in which the dataSimulationAnalysis.R pipeline reproduces the replicate gain analysis (Figures 1D, 1E, S4-5)) — https://github.com/mfurla/Nanodynamo

## Examples

```
Rscript Scripts/dataSimulationAnalysis.R
```

## Evaluation signals

- The gain curve shows monotonically improving (or saturating) Pearson correlation with replicate count.
- Two replicates reach performance remarkably close to optimal, with a maximum additional increase of about 20% for going to more replicates.
- Correlations are computed separately for synthesis, processing, and degradation rates inferred by the Nanodynamo model against the true simulated values.
- Output figure corresponds to Figures 1D and S4-5 of the manuscript, as mapped in the README for dataSimulationAnalysis.R.

## Limitations

- The gain in correlation plateaus quickly, so adding replicates beyond two yields at most ~20% additional improvement — large replicate numbers give diminishing returns.
- The analysis is performed on simulated datasets at a fixed 20-minute metabolic labelling time; results are specific to this labelling duration.
- The workflow depends on regulated_genes_features.xls for rates initial conditions; without appropriate initial conditions the simulations and model fits may not reflect realistic kinetics.

## Evidence

- [results] Simulating five datasets with an increasing number of replicates showed that the gain in correlation between inferred and real rates plateaus quickly: two samples are sufficient: [evidence span withheld — anchor status: not_found]
- [methods] Data required to estimate rates initial conditions for simulations and models fit (regulated_genes_features.xls): "Data required to estimate rates initial conditions for simulations and models fit (regulated_genes_features.xls)"
- [readme] A script to reproduce our analyses on simulated data (dataSimulationAnalysis.R): "An R script to reproduce our analyses on simulated data (dataSimulationAnalysis.R)"
- [readme] dataSimulationAnalysis.R: Figures 1D, 1E, S4-5, S12A, S37-38, S43-52.: "~ _dataSimulationAnalysis.R_: Figures 1D, 1E, S4-5, S12A, S37-38, S43-52."
