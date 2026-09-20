---
name: game-theoretic-payoff-matrix-analysis
description: Use when the analysis goal is to model two-strategy evolutionary game
  dynamics (e.g., hawk vs. dove) from an initial strategy frequency and a specified
  pay-off matrix, tracking frequency trajectories to a frequency-dependent equilibrium
  in R.
when_to_use_negative:
- The question concerns gene frequency change under deterministic or stochastic population
  genetic processes (drift, selection at a locus) rather than strategic interaction
  payoffs — learnPopGen provides separate functions for those models
- The model requires more than two strategies or continuous phenotypic traits rather
  than a two-strategy hawk-dove game with a 2x2 pay-off matrix
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3941
- http://edamontology.org/topic_0621
tools:
- name: learnPopGen
  role: R package providing the hawk-dove game function that takes an initial hawk
    frequency and a pay-off matrix and evolves strategy frequencies via relative fitnesses;
    also supports plotting/exporting the resulting trajectories
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Interactive scientific computing environment in which the pay-off matrix is
    defined, the generational frequency updates are computed, the frequencies data
    frame is written, and trajectories are plotted with base R graphics
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:16338d5f00794af0871d4fb6a662ff79ffa9f063098ad5590299410495c0541a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3941
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# game-theoretic-payoff-matrix-analysis

## Summary

Simulates frequency-dependent evolutionary game dynamics (e.g., the hawk-dove game) in R using learnPopGen: an initial frequency of hawks and a user-specified pay-off matrix are iterated so that hawks and doves interact in the proportions they occur and relative fitnesses determined by the pay-off matrix update strategy frequencies through time.

## When to use

Use when the analysis goal is to model two-strategy evolutionary game dynamics (e.g., hawk vs. dove) from an initial strategy frequency and a specified pay-off matrix, tracking frequency trajectories to a frequency-dependent equilibrium in R.

## When NOT to use

- The question concerns gene frequency change under deterministic or stochastic population genetic processes (drift, selection at a locus) rather than strategic interaction payoffs — learnPopGen provides separate functions for those models
- The model requires more than two strategies or continuous phenotypic traits rather than a two-strategy hawk-dove game with a 2x2 pay-off matrix

## Inputs

- Pay-off matrix for hawk-dove interactions with explicit entries for hawk-hawk, hawk-dove, and dove-dove outcomes (R matrix)
- Initial frequency of hawks (numeric value in [0, 1]) as the simulation starting point
- Number of generations to simulate (or a convergence criterion)

## Outputs

- Data frame of per-generation hawk and dove frequencies (e.g., written to hawkdove_frequencies.csv)
- Plot of strategy frequency trajectories through time (base R graphics)
- Converged frequency-dependent equilibrium strategy frequencies

## How to apply

Define the hawk-dove pay-off matrix in R with explicit entries for hawk-hawk, hawk-dove, and dove-dove interaction outcomes, and set the initial hawk frequency as the simulation start. For each generation, compute the expected payoff of each strategy (hawk and dove) given current strategy frequencies, then normalize payoffs into relative fitness values and multiply current frequencies by those relative fitnesses, iterating for a fixed number of generations or until convergence. Assemble per-generation hawk and dove frequencies into a data frame (e.g., written to hawkdove_frequencies.csv) and plot the frequency trajectory with base R graphics. This mirrors the learnPopGen function described by Revell (2019), which 'takes as inputs an initial frequency of hawks and a pay-off matrix for hawk-dove interactions' and evolves the population by having hawks and doves interact in the proportions they occur.

## Related tools

- **learnPopGen** (R package providing the hawk-dove game function that takes an initial hawk frequency and a pay-off matrix and evolves strategy frequencies via relative fitnesses; also supports plotting/exporting the resulting trajectories) — github.com/liamrevell/learnPopGen
- **R** (Interactive scientific computing environment in which the pay-off matrix is defined, the generational frequency updates are computed, the frequencies data frame is written, and trajectories are plotted with base R graphics)

## Examples

```
library(learnPopGen); hawkdove(p0=0.1, M=matrix(c(-0.5,0.5,1.0,0.0),2,2,byrow=TRUE))
```

## Evaluation signals

- All simulated strategy frequencies remain within [0, 1] at every generation
- Hawk and dove frequencies sum to 1 at every generation
- The trajectory converges to the expected frequency-dependent equilibrium implied by the specified pay-off matrix
- The output CSV (hawkdove_frequencies.csv) contains one row per generation with both strategy frequencies present
- The exported plot shows monotonic or oscillating approach of frequencies toward the equilibrium consistent with the pay-off entries

## Limitations

- No package version number or changelog is stated for learnPopGen, so reproducibility against a pinned release must be established by the user
- No R version requirement is stated in the documentation; verify compatibility in the target session
- The package is designed primarily for teaching evolutionary biology, population genetics, and evolutionary theory — it is pedagogical rather than a production research simulator

## Evidence

- [other] hawk-dove game function inputs: [evidence span withheld — anchor status: not_found]
- [abstract] package purpose: "a new R package called learnPopGen that has been designed primarily for the purposes of teaching evolutionary biology, population genetics, and evolutionary theory"
- [discussion] export plots: "instructors can export plots and animations as flat graphics or animations for use in lecture"
- [readme] README: "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
