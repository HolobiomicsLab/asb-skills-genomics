---
name: structural-identifiability-analysis-of-ode-models
description: Use when a mechanistic ODE model's kinetic parameters (e.g. RNA synthesis,
  processing, export, degradation rates) must be shown to be uniquely recoverable
  from a given set of measurable outputs before fitting rates to experimental data.
when_to_use_negative:
- The model parameters have already been fitted and you only need goodness-of-fit
  or uncertainty estimates rather than a structural (algebraic) uniqueness guarantee.
- Your observation model cannot include nascent RNA measurements — the identifiability
  guarantee here rests specifically on declaring nascent RNA species as measurable
  outputs.
edam_operation: http://edamontology.org/operation_3947
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0204
tools:
- name: Nanodynamo
  role: provides the 9-rate ODE model of RNA life-cycle kinetics and the identifiability
    notebook (parametersIdentifiability.ipynb) whose parameters are analysed
  repo: https://github.com/mfurla/Nanodynamo
- name: R
  role: language of the Nanodynamo analysis scripts and simulation support workflows
    that supply rate initial conditions and model-fit inputs (regulated_genes_features.xls)
  repo: https://github.com/mfurla/Nanodynamo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:e870a054cfd2336bbab5b72a6d0def5433d0e7192831c2034a0c08e5eb8bd7e7
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
  edam_operation: http://edamontology.org/operation_3947
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# structural-identifiability-analysis-of-ode-models

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess whether every kinetic rate (k1-k9) of a 9-rate ODE model of RNA life-cycle dynamics is globally structurally identifiable by running a differential-algebra SIAN analysis on the ODE system with nascent RNA species declared as measurable observation functions.

## When to use

Use when a mechanistic ODE model's kinetic parameters (e.g. RNA synthesis, processing, export, degradation rates) must be shown to be uniquely recoverable from a given set of measurable outputs before fitting rates to experimental data.

## When NOT to use

- The model parameters have already been fitted and you only need goodness-of-fit or uncertainty estimates rather than a structural (algebraic) uniqueness guarantee.
- Your observation model cannot include nascent RNA measurements — the identifiability guarantee here rests specifically on declaring nascent RNA species as measurable outputs.

## Inputs

- Nanodynamo ODE system specification with rates k1-k9 (Julia/SIAN notebook parametersIdentifiability.ipynb)
- rate initial conditions and simulation support data (regulated_genes_features.xls)
- observation functions including nascent RNA species measured via 4sU metabolic labelling

## Outputs

- identifiability_results.csv with one verdict row per kinetic rate
- per-parameter classification of each of the nine rates k1-k9 as globally identifiable or non-identifiable

## How to apply

Load the ODE system specification — synthesis, chromatin detachment, processing, export, translation and degradation rates k1-k9 — from the repository's Julia/SIAN identifiability notebook (parametersIdentifiability.ipynb). Instantiate model states using rate initial conditions and simulation support data from regulated_genes_features.xls, and declare nascent RNA species (from 4sU metabolic labelling) as measurable observation functions alongside the other model outputs. Run the SIAN structural identifiability algorithm (differential-algebra/Rosenfeld elimination) on the combined ODE-plus-observation system, then parse its output to classify each of the nine kinetic rates as globally identifiable or non-identifiable. The rationale is that including 4sU nascent RNA profiling in the observation model guarantees global structural identifiability of all Nanodynamo parameters, so the observation set is the key decision point: without it, rates may not be uniquely determined.

## Related tools

- **Nanodynamo** (provides the 9-rate ODE model of RNA life-cycle kinetics and the identifiability notebook (parametersIdentifiability.ipynb) whose parameters are analysed) — https://github.com/mfurla/Nanodynamo
- **R** (language of the Nanodynamo analysis scripts and simulation support workflows that supply rate initial conditions and model-fit inputs (regulated_genes_features.xls)) — https://github.com/mfurla/Nanodynamo

## Examples

```
jupyter execute parametersIdentifiability.ipynb  # Julia/SIAN structural identifiability on the k1-k9 ODE system
```

## Evaluation signals

- identifiability_results.csv exists and contains exactly one verdict row for each of the 9 rates (k1-k9)
- every parameter is reported as globally (structurally) identifiable in the verdict file
- the SIAN run was executed with nascent RNA species declared as measurable observation functions — rerunning without them is the negative control for the identifiability claim
- model states were instantiated with rate initial conditions traceable to regulated_genes_features.xls

## Limitations

- The global identifiability guarantee is conditional on including 4sU metabolic labelling / nascent RNA profiling in the observation model; other observation sets were not assessed here.
- Structural identifiability is a property of the idealised model plus output set — it does not by itself guarantee practical identifiability given noisy, finite experimental data.

## Evidence
