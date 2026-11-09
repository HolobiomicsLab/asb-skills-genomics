---
name: r-based-reproducible-figure-generation
description: Use when the goal is to reproduce specific manuscript figures (e.g. Figures
  3C, S10, S17 or 2B-E) from a published native-RNA Nanopore analysis by executing
  the authors' versioned R scripts against their released results files.
when_to_use_negative:
- The goal is novel analysis or new visualizations rather than reproducing the specific
  figures enumerated per script in the Nanodynamo README
- Only raw Nanopore dRNA-seq reads are available and the released Results/Files artifacts
  (nanoIDResults.rds, nanoID.modification.probabilities files) have not been downloaded,
  since the figure scripts consume these intermediates
- The working environment cannot reproduce the pinned Anaconda environment (condaEnvironment.yml),
  risking version drift between R package versions and the published panels
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Nanodynamo
  role: Repository of per-figure R scripts and conda environment used to regenerate
    all manuscript figures
  repo: github.com/mfurla/Nanodynamo
- name: R
  role: Execution language for the figure-reproduction scripts (mainModelingResults.R
    and companions)
- name: nano-ID model
  role: Supplies the trained classifier outputs (nanoIDResults.rds, modification-probability
    training data) consumed by the classification and figure scripts
  repo: birdumbrella/nano-ID
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:5d4724c3b86c794c6ebae4c4847f9e583e31ce9ec5b459a419270c37639ca7d6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mfurla/Nanodynamo
  tools:
  - Nanodynamo
  - R
  - nano-ID model
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# R-based reproducible figure generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Regenerate publication figures by running the Nanodynamo repository's per-figure R scripts (e.g. mainModelingResults.R, fractionsReadsClassification.R) inside the pinned condaEnvironment.yml Anaconda environment, mapping each script to the exact manuscript figures it reproduces.

## When to use

Use when the goal is to reproduce specific manuscript figures (e.g. Figures 3C, S10, S17 or 2B-E) from a published native-RNA Nanopore analysis by executing the authors' versioned R scripts against their released results files.

## When NOT to use

- The goal is novel analysis or new visualizations rather than reproducing the specific figures enumerated per script in the Nanodynamo README
- Only raw Nanopore dRNA-seq reads are available and the released Results/Files artifacts (nanoIDResults.rds, nanoID.modification.probabilities files) have not been downloaded, since the figure scripts consume these intermediates
- The working environment cannot reproduce the pinned Anaconda environment (condaEnvironment.yml), risking version drift between R package versions and the published panels

## Inputs

- mfurla/Nanodynamo R scripts (mainModelingResults.R, fractionsReadsClassification.R, couplings.R, nanoIDAccuracy.R, nascentSaturation.R, polysomalProfiling.R)
- condaEnvironment.yml (Anaconda environment specification)
- Released results artifacts: nanoIDResults.rds, Untreated/PladienolideB/LeptomycinB/Harringtonine rate estimates, CVs.rds
- nano-ID training data: nanoID.modification.probabilities.labeled.RData, nanoID.modification.probabilities.unlabeled.RData
- Per-read classification labels / premature_fraction.csv from the fractionsReadsClassification step

## Outputs

- Manuscript figures matching numbered panels (e.g. Figures 2B-E, 3C, 3E-I, 4B-E, 5B-F, 6A-E, S2, S3, S10, S17, S27-29)
- Polysome profiling plots (polysomalProfiling.R)
- Coupling analysis figures (couplings.R)

## How to apply

Clone the mfurla/Nanodynamo repository and first rebuild the exact runtime with the condaEnvironment.yml file, which 'allows to reproduce the Anaconda environment used for our analyses'. Each R script in the Scripts folder is bound to specific figures: fractionsReadsClassification.R generates Figures 3C, S10, S17 by characterizing nascent and premature read fractions, nanoIDAccuracy.R produces Figure S2, nascentSaturation.R Figure S3, polysomalProfiling.R Figures 2A/3D/4A/5A, couplings.R Figures 6A-E/S27-29, and mainModelingResults.R reproduces Figures 2B-E, 3E-I, 4B-E, 4G, 5B-F, 6A-D and a large panel of supplementary figures. Feed the scripts the released Results and Files artifacts (nanoIDResults.rds, nanoID.modification.probabilities.labeled.RData, nanoID.modification.probabilities.unlabeled.RData, polysomalProfiling, CVs.rds) rather than re-deriving intermediates, unless the analysis goal is to re-run the full pipeline from raw dRNA-seq reads. Validate by comparing the regenerated panels against the figure numbers and expected quantitative values reported in the paper (e.g. ~83% premature reads for the nano-COP K562 chromatin sample).

## Related tools

- **Nanodynamo** (Repository of per-figure R scripts and conda environment used to regenerate all manuscript figures) — github.com/mfurla/Nanodynamo
- **R** (Execution language for the figure-reproduction scripts (mainModelingResults.R and companions))
- **nano-ID model** (Supplies the trained classifier outputs (nanoIDResults.rds, modification-probability training data) consumed by the classification and figure scripts) — birdumbrella/nano-ID

## Examples

```
conda env create -f condaEnvironment.yml && conda run -n nanodynamo Rscript Scripts/fractionsReadsClassification.R
```

## Evaluation signals

- Regenerated panel content matches the figure-to-script mapping declared in the README (e.g. fractionsReadsClassification.R yields Figures 3C, S10, S17)
- Computed premature read fraction reproduces the reported ~83% premature reads for the nano-COP K562 chromatin sample (GSM4663623)
- The R session runs under the environment defined by condaEnvironment.yml without package-version warnings
- Scripts execute end-to-end against the released Files (nanoIDResults.rds, nanoID.modification.probabilities.labeled.RData/unlabeled.RData) without missing-object errors

## Limitations

- Figure reproduction depends on the released intermediate artifacts; raw-read reprocessing is handled by separate scripts (nanoIDMain.R, experimentalDataAnalysis.R) and is not covered by the figure scripts alone
- Nano-ID classification accuracy was characterized as its own analysis (nanoIDAccuracy.R, Figure S2), so figure-level reproduction does not itself validate classifier performance
- One patch script variant (allInternalFunctionsPatch_nuclearPrematureDecay.R) exists for core functions, so script source must be checked against the model variant being reproduced

## Evidence
