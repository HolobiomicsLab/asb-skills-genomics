---
name: population-genetics-figure-reproduction
description: Use when a task asks to reproduce a population-genetics figure (e.g.
  allele coalescence under genetic drift, Figure 2) by running a simulation in R and
  exporting the plot as a graphic file.
when_to_use_negative:
- The target figure involves deterministic gene frequency change or phenotypic trait
  evolution under selection rather than stochastic coalescence under drift — use the
  corresponding learnPopGen function (e.g. frequency-change or trait-animation functions)
  instead of coalescent.plot.
- The user lacks the R environment and only needs a quick visualization — the article
  notes web interfaces exist for many functions for students less adept in R, which
  may be preferable to a local R reproduction.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3056
tools:
- name: learnPopGen
  role: Provides coalescent.plot and related simulation functions that generate the
    population-genetic figures (e.g. Wright–Fisher coalescence under drift) to be
    reproduced and exported
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Scientific computing environment in which learnPopGen is loaded, the simulation
    is run, and the plot is exported as a flat graphic (PNG)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:a9d002e1ef87f616d376cee9efc0a18f454a7cd1a97da204addd212d3eb32eb6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3056
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# population-genetics-figure-reproduction

## Summary

Reproduces figures from population genetics teaching papers by running the learnPopGen R package's simulation functions (e.g. coalescent.plot for Wright–Fisher drift coalescence) and exporting the resulting plots as flat graphic files such as PNG for validation against the published figure.

## When to use

Use when a task asks to reproduce a population-genetics figure (e.g. allele coalescence under genetic drift, Figure 2) by running a simulation in R and exporting the plot as a graphic file.

## When NOT to use

- The target figure involves deterministic gene frequency change or phenotypic trait evolution under selection rather than stochastic coalescence under drift — use the corresponding learnPopGen function (e.g. frequency-change or trait-animation functions) instead of coalescent.plot.
- The user lacks the R environment and only needs a quick visualization — the article notes web interfaces exist for many functions for students less adept in R, which may be preferable to a local R reproduction.

## Inputs

- learnPopGen package installed in R from github.com/liamrevell/learnPopGen
- Simulation function name and default parameters for the target figure (e.g. coalescent.plot for Figure 2)
- Reference figure from the article (e.g. Figure 2, coalescent genealogy under drift)

## Outputs

- coalescent_plot.png — a flat graphic file depicting allele lineages coalescing over generations under genetic drift
- R coalescent.plot simulation object rendering allele lineages converging backward in time to a common ancestor

## How to apply

Install and load the learnPopGen package in R from its GitHub repository (github.com/liamrevell/learnPopGen). Call the specific simulation function matching the target figure — for Figure 2, coalescent.plot — using the function's default settings, which simulate allele coalescence (Kingman, 1982) in a Wright–Fisher population experiencing genetic drift. Render the resulting coalescent genealogy as a plot showing allele lineages converging backward in time to a common ancestor. Export the plot from R as a flat graphic file (e.g. coalescent_plot.png), consistent with the article's note that it is straightforward to export plots and animations from R in the form of flat or animated graphics. Validate by confirming the file exists, is a valid PNG, and visually matches the target figure's form (lineages coalescing over generations under drift).

## Related tools

- **learnPopGen** (Provides coalescent.plot and related simulation functions that generate the population-genetic figures (e.g. Wright–Fisher coalescence under drift) to be reproduced and exported) — github.com/liamrevell/learnPopGen
- **R** (Scientific computing environment in which learnPopGen is loaded, the simulation is run, and the plot is exported as a flat graphic (PNG))

## Examples

```
Rscript -e 'library(learnPopGen); png("coalescent_plot.png"); coalescent.plot(); dev.off()'
```

## Evaluation signals

- coalescent_plot.png exists at the expected output path and is a valid PNG graphic
- The plot depicts allele lineages coalescing backward in time to a common ancestor, matching the visual form of Figure 2 of the article
- The simulation reflects genetic drift in a Wright–Fisher population, consistent with Kingman (1982) coalescence as described in the article
- The function was run with the article's stated settings (default coalescent simulation settings of coalescent.plot)

## Limitations

- No package version number or changelog is stated for learnPopGen, so reproducibility against a pinned version cannot be confirmed from the article
- No R version requirement is stated in the article
- Figure 2 illustrates a stochastic simulation of drift, so individual runs will vary visually from the published figure; only the qualitative coalescence pattern should be expected to match

## Evidence

- [results] coalescent.plot illustrates coalescence under drift: [evidence span withheld — anchor status: not_found]
- [abstract] export plots as flat graphics: "it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos"
- [abstract] package purpose: "a new R package called learnPopGen that has been designed primarily for the purposes of teaching evolutionary biology, population genetics, and evolutionary theory"
- [readme] README description: "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
