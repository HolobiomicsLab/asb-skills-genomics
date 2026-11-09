---
name: nascent-rna-observation-modeling
description: Use when testing whether ODE kinetic parameters (synthesis, detachment,
  processing, export, translation, degradation rates) of an RNA life-cycle model are
  structurally identifiable from nascent RNA profiling observations.
when_to_use_negative:
- The goal is practical parameter estimation from experimental data (use Nanodynamo/nano-ID
  rate inference scripts such as nanoIDMain.R or experimentalDataAnalysis.R) rather
  than a priori structural identifiability analysis
- No nascent RNA / metabolic labelling observations are part of the measurement model,
  since the guarantee of global identifiability relies on including 4sU nascent RNA
  profiling
- The model is not an RNA life-cycle ODE with synthesis, processing, export, translation
  and degradation rates comparable to k1-k9
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: Nanodynamo
  role: Provides the ODE system (rates k1-k9) and nascent RNA observation framework
    for the identifiability analysis
  repo: https://github.com/mfurla/Nanodynamo
- name: R
  role: Language of the Nanodynamo pipeline that supplies simulation support data
    and downstream rate estimation (regulated_genes_features.xls inputs, mainModelingResults.R
    analyses)
  repo: https://github.com/mfurla/Nanodynamo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:9ff0d6cdb6bc9bac9318563b2c8aa7879c4a7ef2d51fb214d93f2638d2ad8fc3
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# nascent-rna-observation-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Declares nascent RNA species, measured via 4sU metabolic labelling, as measurable observation outputs of the Nanodynamo ODE system (rates k1-k9) so that a structural identifiability analysis (SIAN, differential-algebra/Rosenfeld elimination) can establish whether every kinetic rate is globally identifiable.

## When to use

Use when testing whether ODE kinetic parameters (synthesis, detachment, processing, export, translation, degradation rates) of an RNA life-cycle model are structurally identifiable from nascent RNA profiling observations.

## When NOT to use

- The goal is practical parameter estimation from experimental data (use Nanodynamo/nano-ID rate inference scripts such as nanoIDMain.R or experimentalDataAnalysis.R) rather than a priori structural identifiability analysis
- No nascent RNA / metabolic labelling observations are part of the measurement model, since the guarantee of global identifiability relies on including 4sU nascent RNA profiling
- The model is not an RNA life-cycle ODE with synthesis, processing, export, translation and degradation rates comparable to k1-k9

## Inputs

- Nanodynamo ODE system specification for rates k1-k9 (Julia SIAN notebook parametersIdentifiability.ipynb from mfurla/Nanodynamo)
- Nascent RNA observation functions from 4sU metabolic labelling profiling
- Rates initial conditions and simulation support data (regulated_genes_features.xls)

## Outputs

- Per-parameter identifiability verdicts for the 9 kinetic rates (identifiability_results.csv)
- SIAN structural identifiability classification (globally identifiable vs non-identifiable) for each parameter

## How to apply

Load the Nanodynamo ODE specification covering rates k1-k9 (synthesis, chromatin detachment, processing, export, translation and degradation) from the repository's Julia SIAN notebook (parametersIdentifiability.ipynb), and augment the observation model with nascent RNA species measured by 4sU metabolic labelling as declared measurable outputs. Instantiate model states using rates initial conditions and simulation support data from regulated_genes_features.xls. Run the SIAN structural identifiability algorithm (differential-algebra / Rosenfeld elimination) on the ODE-plus-observation system, then parse its output to classify each of the nine rates as globally identifiable or non-identifiable. The rationale is that without nascent RNA in the observation model the rate map may be degenerate; with 4sU-labelled nascent RNA included, every Nanodynamo parameter admits a unique rate set per model output.

## Related tools

- **Nanodynamo** (Provides the ODE system (rates k1-k9) and nascent RNA observation framework for the identifiability analysis) — https://github.com/mfurla/Nanodynamo
- **R** (Language of the Nanodynamo pipeline that supplies simulation support data and downstream rate estimation (regulated_genes_features.xls inputs, mainModelingResults.R analyses)) — https://github.com/mfurla/Nanodynamo

## Examples

```
jupyter notebook parametersIdentifiability.ipynb
```

## Evaluation signals

- identifiability_results.csv exists and contains exactly one verdict row for each of the 9 kinetic rates k1-k9
- Every parameter is reported as globally (structurally) identifiable when nascent RNA profiling is included in the observation model
- The SIAN run terminates without algebraic exceptions on the ODE-plus-observation system assembled with states initialised from regulated_genes_features.xls
- Re-running the SIAN analysis with nascent RNA outputs removed should show the identifiability guarantee no longer holds, confirming the observation model is the discriminating factor

## Limitations

- The global identifiability guarantee is structural: it holds for the ODE-plus-observation model, not for noisy finite-sample rate estimates produced from experimental data
- Identifiability verdicts depend on declaring nascent RNA species as measurable outputs; omitting 4sU metabolic labelling from the observation model invalidates the conclusion
- Initial conditions for rates and simulations must come from regulated_genes_features.xls; missing or malformed gene feature data can prevent model instantiation

## Evidence
