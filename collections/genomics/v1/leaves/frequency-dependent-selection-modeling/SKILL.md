---
name: frequency-dependent-selection-modeling
description: Use when modeling frequency-dependent selection or evolutionary game
  theory dynamics — e.g., two strategies (hawks/doves) whose fitnesses depend on their
  proportions in the population and a specified pay-off matrix.
when_to_use_negative:
- The selection regime is constant (fitnesses do not depend on strategy frequencies)
  — use a standard deterministic or stochastic gene-frequency simulation instead of
  the hawk-dove game function.
- More than two strategies are involved; the surfaced workflow is built around the
  two-strategy hawk-dove interaction with a 2x2 pay-off matrix.
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3945
- http://edamontology.org/topic_0625
tools:
- name: learnPopGen
  role: R package supplying the hawk-dove game-theory model simulation and graphics
    for teaching evolutionary game theory
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Scientific computing environment in which the pay-off matrix is defined, the
    simulation is iterated, frequencies are written to CSV, and trajectories are plotted
    with base R graphics
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:e3b351f270af52990602184c2a591998eb934303102c111366e432dfaa3788cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3945
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# frequency-dependent-selection-modeling

## Summary

Simulates the hawk-dove evolutionary game in R using learnPopGen: given an initial hawk frequency and a user-specified pay-off matrix, iteratively updates hawk/dove frequencies through relative fitnesses until a frequency-dependent equilibrium is reached.

## When to use

Use when modeling frequency-dependent selection or evolutionary game theory dynamics — e.g., two strategies (hawks/doves) whose fitnesses depend on their proportions in the population and a specified pay-off matrix.

## When NOT to use

- The selection regime is constant (fitnesses do not depend on strategy frequencies) — use a standard deterministic or stochastic gene-frequency simulation instead of the hawk-dove game function.
- More than two strategies are involved; the surfaced workflow is built around the two-strategy hawk-dove interaction with a 2x2 pay-off matrix.

## Inputs

- Initial frequency of 'hawks' (numeric in [0,1])
- Pay-off matrix for hawk-hawk, hawk-dove, and dove-dove interaction outcomes (R matrix)
- Number of generations or convergence criterion

## Outputs

- Per-generation hawk and dove frequencies written to hawkdove_frequencies.csv (data frame)
- Base R plot of strategy-frequency trajectory through time
- Converged frequency-dependent equilibrium frequency of hawks

## How to apply

Load the learnPopGen source (github.com/liamrevell/learnPopGen) in an R session, then define the hawk-dove pay-off matrix with explicit entries for hawk-hawk, hawk-dove, and dove-dove outcomes and set the initial hawk frequency as the starting point. At each generation, compute the expected payoff of each strategy given current frequencies, normalize payoffs into relative fitnesses, and multiply current frequencies by those relative fitnesses; iterate for a fixed number of generations or until convergence. The package function takes the initial hawk frequency and pay-off matrix as inputs and evolves the population by having hawks and doves interact in the proportions they occur, with relative fitnesses determined by the pay-off matrix. Assemble per-generation hawk and dove frequencies into a data frame, write hawkdove_frequencies.csv, and plot the strategy-frequency trajectory with base R graphics. Rationale: learnPopGen is designed for teaching evolutionary biology, population genetics, and evolutionary theory, and its game-theory functions support analysis of frequency-dependent phenomena.

## Related tools

- **learnPopGen** (R package supplying the hawk-dove game-theory model simulation and graphics for teaching evolutionary game theory) — github.com/liamrevell/learnPopGen
- **R** (Scientific computing environment in which the pay-off matrix is defined, the simulation is iterated, frequencies are written to CSV, and trajectories are plotted with base R graphics)

## Examples

```
Rscript -e 'library(learnPopGen); sim <- hawkdove(p=0.1, A=matrix(c(-1,2,0,1),2,2)); write.csv(sim,"hawkdove_frequencies.csv")'
```

## Evaluation signals

- All simulated hawk and dove frequencies remain within [0, 1] at every generation.
- Hawk and dove frequencies sum to 1 at every generation.
- The trajectory converges to the expected frequency-dependent equilibrium of the specified pay-off matrix.
- hawkdove_frequencies.csv contains per-generation frequencies as a well-formed data frame matching the plotted trajectory.
- Simulated dynamics reflect that hawks and doves interact in the proportions they occur, with relative fitnesses set by the pay-off matrix.

## Limitations

- No package version number or changelog is stated for learnPopGen, and no R version requirement is documented.
- Web interfaces exist for some functions, but their URLs are not provided in the text; students less adept in R may need instructor guidance to find them.
- Plots and animations must be exported from R; the article documents export as flat or animated graphics or videos but does not specify formats in this context.

## Evidence

- [other] hawk-dove function inputs and dynamics: [evidence span withheld — anchor status: not_found]
- [abstract] package purpose: "a new R package called learnPopGen that has been designed primarily for the purposes of teaching evolutionary biology, population genetics, and evolutionary theory"
- [abstract] export of graphics: "it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos"
- [readme] repository contents: "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
- [discussion] interactive use: "Students can use the package within an interactive R session"
