---
name: replicate-number-sensitivity-analysis
description: 'Decision support for planning Nanodynamo or similar RNA-kinetics experiments:
  when you must choose how many sequencing replicates suffice for reliable rate inference
  before committing wet-lab budget.'
when_to_use_negative:
- Input is real experimental data rather than simulated data — this skill quantifies
  the in silico relationship between replicate count and rate-inference correlation,
  not accuracy on an existing experimental dataset.
- The question is nascent-RA read depth/saturation rather than replicate count — use
  the nascentSaturation.R analysis (Figure S3) instead.
- You need per-gene coupling or post-transcriptional processing analyses (couplings.R,
  fractionsReadsClassification.R), not experimental-design optimisation.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Nanodynamo
  role: Framework whose kinetic model is fitted to simulated 1–5 replicate datasets
    to infer synthesis, processing, and degradation rates for the sensitivity curve
  repo: https://github.com/mfurla/Nanodynamo
- name: R
  role: Language of the dataSimulationAnalysis.R pipeline used to simulate datasets,
    fit rates, compute Pearson correlations, and plot the gain curve
  repo: https://github.com/mfurla/Nanodynamo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:38da3cf93e740148bfeea5e9bf2fcaf871a0ea7f41a9d5257fa30f4062201ca4
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# replicate-number-sensitivity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Empirical replicate-count optimisation for Nanodynamo kinetic inference: simulate in silico metabolic-labelling datasets at a fixed 20-minute labelling time with 1–5 replicates, fit the kinetic model to each setting, and quantify the plateau in correlation between inferred and real synthesis/processing/degradation rates.

## When to use

Decision support for planning Nanodynamo or similar RNA-kinetics experiments: when you must choose how many sequencing replicates suffice for reliable rate inference before committing wet-lab budget.

## When NOT to use

- Input is real experimental data rather than simulated data — this skill quantifies the in silico relationship between replicate count and rate-inference correlation, not accuracy on an existing experimental dataset.
- The question is nascent-RA read depth/saturation rather than replicate count — use the nascentSaturation.R analysis (Figure S3) instead.
- You need per-gene coupling or post-transcriptional processing analyses (couplings.R, fractionsReadsClassification.R), not experimental-design optimisation.

## Inputs

- regulated_genes_features.xls — rates initial conditions for simulations and model fit
- dataSimulationAnalysis.R / dataSimulationMain.R / dataSimulationBash.sh scripts from mfurla/Nanodynamo
- Fixed metabolic labelling time parameter (20 minutes) and replicate counts (1–5)

## Outputs

- Per-replicate-count inferred synthesis, processing, and degradation rates
- Pearson correlation between inferred and real rates for each replicate number
- Replicate gain curve plot (replicates_gain_curve.png; Figures 1D/S4-5)

## How to apply

Load rates initial conditions for simulations and model fitting from regulated_genes_features.xls to define the simulated genes' kinetic parameters. Using the dataSimulationAnalysis.R pipeline from the mfurla/Nanodynamo repository (driven via dataSimulationBash.sh/dataSimulationMain.R), generate in silico datasets at a fixed 20-minute metabolic labelling time with 1, 2, 3, 4, and 5 simulated replicates. Fit the Nanodynamo kinetic model to each replicate setting to infer synthesis, processing, and degradation rates per gene. Compute the Pearson correlation between inferred and real (true) rates for each replicate number and plot the correlation gain as a function of replicate count (Figures 1D/S4-5, replicates_gain_curve.png). In the article, the gain plateaus quickly: two replicates already give performance remarkably close to optimal, with a maximum additional increase of 20% — use this saturation behaviour as the decision rule for replicate budgeting.

## Related tools

- **Nanodynamo** (Framework whose kinetic model is fitted to simulated 1–5 replicate datasets to infer synthesis, processing, and degradation rates for the sensitivity curve) — https://github.com/mfurla/Nanodynamo
- **R** (Language of the dataSimulationAnalysis.R pipeline used to simulate datasets, fit rates, compute Pearson correlations, and plot the gain curve) — https://github.com/mfurla/Nanodynamo

## Examples

```
Rscript dataSimulationAnalysis.R
```

## Evaluation signals

- The correlation-gain curve over 1→5 replicates is monotonically improving or saturating, matching Figure 1D/S4-5.
- Two replicates yield a correlation remarkably close to the optimum, with a maximum additional increase of ~20% at higher replicate counts.
- Pearson correlations between inferred and real rates are computed for each of the five replicate settings (1, 2, 3, 4, 5) at the same fixed 20-min labelling time.
- Simulated gene kinetic parameters reproduce the initial conditions defined in regulated_genes_features.xls.

## Limitations

- Result is specific to the simulated gene set and kinetic parameters drawn from regulated_genes_features.xls and the 20-minute metabolic labelling time; other labelling times or gene panels may shift the plateau.
- It is a simulation-based assessment; the two-replicate near-optimal conclusion depends on Nanodynamo's model assumptions (e.g., no premature nuclear decay unless patched).

## Evidence

- [results] Simulating five datasets with an increasing number of replicates showed that the gain in correlation between inferred and real rates plateaus quickly: two samples are sufficient: [evidence span withheld — anchor status: not_found]
