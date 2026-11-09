---
name: simulation-trajectory-visualization
description: Use when the task requires simulating allele-frequency change over generations
  under combined deterministic and stochastic forces (e.g., selection and drift) and
  producing time-series trajectories plus plots from a biallelic population-genetic
  model.
when_to_use_negative:
- The goal is purely deterministic analysis with no stochastic (drift) component —
  use a deterministic selection or frequency-function tool instead
- The trait is quantitative/phenotypic rather than a biallelic gene frequency — learnPopGen
  handles phenotypic trait changes under selection via other functions
- The user cannot use R and needs a result without any R session — the article recommends
  web interfaces for those 'less adept in the R environment', which may be a better
  route than scripting this simulation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3056
tools:
- name: learnPopGen
  role: Provides drift.selection (and plotting methods) to simulate joint selection
    and drift at a biallelic locus and visualize the replicate allele-frequency trajectories
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Scientific computing environment in which learnPopGen simulations are run,
    trajectory CSVs written, and plots/graphics exported
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:c9672a8e72407bf6e9ab5c4183c4826f305cbc2f7c18d8929764c95bcf0d1f8a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3056
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# simulation-trajectory-visualization

## Summary

Simulate allele-frequency trajectories under deterministic and stochastic evolutionary processes (e.g., selection plus genetic drift at a biallelic locus) using learnPopGen's Wright–Fisher-style simulation functions, then export and visualize the generation-by-generation trajectories as plots/graphics.

## When to use

Use when the task requires simulating allele-frequency change over generations under combined deterministic and stochastic forces (e.g., selection and drift) and producing time-series trajectories plus plots from a biallelic population-genetic model.

## When NOT to use

- The goal is purely deterministic analysis with no stochastic (drift) component — use a deterministic selection or frequency-function tool instead
- The trait is quantitative/phenotypic rather than a biallelic gene frequency — learnPopGen handles phenotypic trait changes under selection via other functions
- The user cannot use R and needs a result without any R session — the article recommends web interfaces for those 'less adept in the R environment', which may be a better route than scripting this simulation

## Inputs

- Initial allele frequency (p0) at a biallelic locus
- Genotype fitness vector (w) defining the deterministic selection step
- Effective population size (Ne)
- Number of generations to simulate
- Number of replicate populations

## Outputs

- Allele-frequency trajectory object per replicate (generation-by-generation frequencies)
- drift_selection_trajectories.csv with one frequency row per sampled generation per replicate
- drift_selection_plot.png showing replicate trajectories through time

## How to apply

Install/load learnPopGen from its source repository (github.com/liamrevell/learnPopGen) in an interactive R session. For joint selection-and-drift, call drift.selection with the initial allele frequency, a genotype fitness vector, effective population size (Ne), number of generations, and number of replicate populations; the function performs Wright–Fisher-style binomial sampling of allele frequencies each generation modulated by the deterministic selection step defined by the genotype fitnesses. Extract the returned per-replicate allele-frequency trajectory objects and write generation-by-generation frequencies to a CSV (e.g., drift_selection_trajectories.csv). Plot the replicate trajectories through time with the function's built-in plotting method and save the figure (e.g., drift_selection_plot.png); the article notes plots and animations can be exported from R as flat or animated graphics. Students/instructors less fluent in R may instead use one of the package's web interfaces, which run R on a remote server from a browser or smartphone.

## Related tools

- **learnPopGen** (Provides drift.selection (and plotting methods) to simulate joint selection and drift at a biallelic locus and visualize the replicate allele-frequency trajectories) — github.com/liamrevell/learnPopGen
- **R** (Scientific computing environment in which learnPopGen simulations are run, trajectory CSVs written, and plots/graphics exported)

## Examples

```
Rscript -e 'library(learnPopGen); sim <- drift.selection(p0=0.5, w=c(1,0.9,0.8), Ne=100, ngen=200, nrep=10); plot(sim)'
```

## Evaluation signals

- The CSV contains a frequency row per sampled generation per replicate, and every value lies in [0, 1]
- Plotted replicate trajectories show stochastic fluctuation around the selection-driven deterministic trend (not smooth deterministic curves)
- The output figure is exported successfully as a flat graphic from R (drift_selection_plot.png)
- Trajectory object returned by drift.selection includes one trajectory per requested replicate population
- Generation-by-generation frequencies reflect Wright–Fisher binomial sampling modulated by the specified genotype fitness vector

## Limitations

- No package version number or changelog is stated for learnPopGen, and no R version requirement is documented
- Web interface URLs are not provided in the article text; interfaces 'use R on a remote server' but locations must be found separately
- The package is designed primarily for teaching evolutionary biology, population genetics, and evolutionary theory, not for large-scale research-grade simulation

## Evidence

- [abstract] a new R package called learnPopGen that has been designed primarily for the purposes of teaching evolutionary biology, population genetics, and evolutionary theory: "a new R package called learnPopGen that has been designed primarily for the purposes of teaching evolutionary biology, population genetics, and evolutionary theory"
- [abstract] to visualize gene frequency changes through time under multiple deterministic and stochastic processes: "to visualize gene frequency changes through time under multiple deterministic and stochastic processes"
- [discussion] Students can use the package within an interactive R session: "Students can use the package within an interactive R session"
- [abstract] it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos: "it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos"
- [readme] This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.": "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
