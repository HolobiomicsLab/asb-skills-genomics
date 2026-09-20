---
name: julia-sian-notebook-execution
description: Use when you need to establish whether kinetic-rate parameters of the
  Nanodynamo ODE system (synthesis, chromatin detachment, processing, export, translation,
  degradation) are globally structurally identifiable, or when auditing identifiability
  of nascent-RNA-informed RNA-dynamics models.
when_to_use_negative:
- Your model output observations lack nascent RNA (4sU metabolic labelling) measurements
  — identifiability of every rate is only guaranteed when nascent RNA profiling is
  included in the observation model
- You are estimating kinetic rates from experimental data rather than testing structural
  identifiability — use the nano-ID (nanoIDMain.R) or Nanodynamo modelling scripts
  (experimentalDataAnalysis.R) instead
- Input is already a fitted rate table (e.g. nanoIDResults.rds or the Untreated/PladienolideB
  Results folders) rather than an ODE system specification
tools:
- name: Nanodynamo
  role: Provides the ODE model specification (rates k1-k9), the Julia/SIAN identifiability
    notebook (parametersIdentifiability.ipynb), and the simulation support data (regulated_genes_features.xls)
    used for the identifiability analysis
  repo: https://github.com/mfurla/Nanodynamo
- name: R
  role: Used in the surrounding Nanodynamo workflow for handling rate initial conditions
    from regulated_genes_features.xls and downstream analyses (allInternalFunctions.R,
    mainModelingResults.R)
  repo: https://github.com/mfurla/Nanodynamo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:abb4ab4e8fd27860deadd185e14b3ec2eb045839efb32e84d963b64534a37579
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# julia-sian-notebook-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce the structural identifiability analysis of the Nanodynamo 9-rate ODE model (rates k1–k9) by executing the repository's Julia SIAN notebook (parametersIdentifiability.ipynb), verifying that every kinetic rate is globally identifiable when nascent RNA (4sU metabolic labelling) is declared as a measurable output.

## When to use

Use when you need to establish whether kinetic-rate parameters of the Nanodynamo ODE system (synthesis, chromatin detachment, processing, export, translation, degradation) are globally structurally identifiable, or when auditing identifiability of nascent-RNA-informed RNA-dynamics models.

## When NOT to use

- Your model output observations lack nascent RNA (4sU metabolic labelling) measurements — identifiability of every rate is only guaranteed when nascent RNA profiling is included in the observation model
- You are estimating kinetic rates from experimental data rather than testing structural identifiability — use the nano-ID (nanoIDMain.R) or Nanodynamo modelling scripts (experimentalDataAnalysis.R) instead
- Input is already a fitted rate table (e.g. nanoIDResults.rds or the Untreated/PladienolideB Results folders) rather than an ODE system specification

## Inputs

- Nanodynamo ODE system specification (rates k1-k9) from parametersIdentifiability.ipynb (Julia/SIAN notebook in the Scripts folder)
- Nascent RNA observation functions with nascent RNA species declared as measurable outputs
- regulated_genes_features.xls — data required to estimate rates initial conditions for simulations and models fit

## Outputs

- identifiability_results.csv — one verdict row for each of the 9 kinetic rates, classified as globally identifiable or non-identifiable
- Per-parameter identifiability verdicts confirming all nine rates are globally structurally identifiable when nascent RNA profiling is included

## How to apply

Load the Nanodynamo ODE system specification (rates k1–k9 covering synthesis, chromatin detachment, processing, export, translation and degradation) together with the nascent RNA observation functions from parametersIdentifiability.ipynb in the mfurla/Nanodynamo repository's Scripts folder. Instantiate model states by supplying rate initial conditions and simulation support data from regulated_genes_features.xls. Run the SIAN structural identifiability algorithm (differential-algebra/Rosenfeld elimination) on the ODE-plus-observation system with nascent RNA species declared as measurable outputs — the 4sU metabolic labelling measurements are what guarantee global identifiability of all nine rates. Parse the SIAN output to classify each of the nine kinetic rates as globally identifiable or non-identifiable and write per-parameter verdicts to identifiability_results.csv. The analysis environment is reproduced via the repository's condaEnvironment.yml Anaconda environment.

## Related tools

- **Nanodynamo** (Provides the ODE model specification (rates k1-k9), the Julia/SIAN identifiability notebook (parametersIdentifiability.ipynb), and the simulation support data (regulated_genes_features.xls) used for the identifiability analysis) — https://github.com/mfurla/Nanodynamo
- **R** (Used in the surrounding Nanodynamo workflow for handling rate initial conditions from regulated_genes_features.xls and downstream analyses (allInternalFunctions.R, mainModelingResults.R)) — https://github.com/mfurla/Nanodynamo

## Examples

```
cd Scripts && jupyter nbconvert --to notebook --execute parametersIdentifiability.ipynb
```

## Evaluation signals

- identifiability_results.csv exists and contains exactly one verdict row for each of the 9 kinetic rates (k1-k9)
- Every parameter is reported as globally (structurally) identifiable when nascent RNA profiling is included in the observation model
- The SIAN run declares nascent RNA species as measurable outputs; omitting them should not be expected to yield all-global identifiability
- Rate initial conditions instantiated in the notebook are traceable to regulated_genes_features.xls
- The Julia/Anaconda environment used matches condaEnvironment.yml from the repository

## Limitations

- Global identifiability of all nine rates holds only when nascent RNA (4sU metabolic labelling) profiling is included in the observations; with other observation models rates may be non-identifiable
- The identifiability result is structural (differential-algebra based), not a statement about practical/numerical identifiability from noisy experimental data
- The notebook requires the repository's Anaconda/Julia environment (condaEnvironment.yml) to reproduce the SIAN analysis

## Evidence
