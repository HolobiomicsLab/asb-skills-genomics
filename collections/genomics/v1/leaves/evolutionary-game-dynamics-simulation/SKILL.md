---
name: evolutionary-game-dynamics-simulation
description: Use when analyzing frequency-dependent selection between two behavioral
  strategies (hawk vs. dove), where the user supplies an initial hawk frequency and
  a payoff matrix for hawk-hawk, hawk-dove, and dove-dove interactions.
when_to_use_negative:
- The model of interest is frequency-independent selection on phenotypic traits or
  gene frequencies under drift/migration rather than strategy-based game dynamics
- The analysis targets more than two competing strategies or asymmetric games not
  expressible as a 2x2 hawk-dove payoff matrix
- The user is less adept in R and only needs a prebuilt interactive web interface
  rather than a scripted simulation
edam_operation: http://edamontology.org/operation_0438
tools:
- name: learnPopGen
  role: R package whose hawk-dove game function takes an initial frequency of hawks
    and a pay-off matrix, evolving the population via frequency-dependent relative
    fitnesses
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Scientific computing environment used for the interactive simulation session,
    data frame assembly, CSV output, and base graphics plotting
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:8d4e188ee0ad7b7cb443fb06a46d5a7c189915ec2f401d3ad6fdd90011264455
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_0438
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# evolutionary-game-dynamics-simulation

## Summary

Simulates frequency-dependent evolutionary game dynamics (e.g., the hawk-dove game) by iteratively computing per-strategy expected payoffs from a user-specified payoff matrix and updating strategy frequencies via relative fitness, as implemented in the learnPopGen R package.

## When to use

Use when analyzing frequency-dependent selection between two behavioral strategies (hawk vs. dove), where the user supplies an initial hawk frequency and a payoff matrix for hawk-hawk, hawk-dove, and dove-dove interactions.

## When NOT to use

- The model of interest is frequency-independent selection on phenotypic traits or gene frequencies under drift/migration rather than strategy-based game dynamics
- The analysis targets more than two competing strategies or asymmetric games not expressible as a 2x2 hawk-dove payoff matrix
- The user is less adept in R and only needs a prebuilt interactive web interface rather than a scripted simulation

## Inputs

- Initial frequency of hawks (numeric value in [0, 1])
- Pay-off matrix with explicit entries for hawk-hawk, hawk-dove, and dove-dove interaction outcomes (2x2 numeric matrix in R)
- Number of generations / convergence criterion for the simulation

## Outputs

- Data frame of per-generation hawk and dove frequencies (e.g., written to hawkdove_frequencies.csv)
- Trajectory plot of strategy frequencies through time (base R graphics, exportable as flat or animated graphics)

## How to apply

In an interactive R session with learnPopGen (source available at github.com/liamrevell/learnPopGen), define the payoff matrix with explicit entries for each interaction outcome and set the initial frequency of hawks as the starting point. Each generation, hawks and doves interact in the proportions they occur, and the expected payoff of each strategy is computed given the current strategy frequencies; the fitnesses assigned by the payoff matrix then determine relative fitness. Update strategy frequencies by normalizing payoffs into relative fitnesses and multiplying the current frequencies by those relative fitnesses, iterating over a fixed number of generations or until convergence. Assemble per-generation hawk and dove frequencies into a data frame (e.g., hawkdove_frequencies.csv) and plot the frequency trajectory through time with base R graphics; learnPopGen also supports exporting plots and animations as flat or animated graphics.

## Related tools

- **learnPopGen** (R package whose hawk-dove game function takes an initial frequency of hawks and a pay-off matrix, evolving the population via frequency-dependent relative fitnesses) — github.com/liamrevell/learnPopGen
- **R** (Scientific computing environment used for the interactive simulation session, data frame assembly, CSV output, and base graphics plotting)

## Examples

```
A <- matrix(c(-1, 0, 2, 1), 2, 2); res <- hawkdove(p = 0.1, A = A); write.csv(res, "hawkdove_frequencies.csv")
```

## Evaluation signals

- All simulated strategy frequencies remain within [0, 1] at every generation
- Hawk and dove frequencies sum to 1 at every generation
- The trajectory converges to the expected frequency-dependent equilibrium implied by the specified pay-off matrix
- Per-generation output data frame has one row per generation with exactly the two strategy-frequency columns before writing hawkdove_frequencies.csv

## Limitations

- The package is designed primarily for teaching evolutionary biology, population genetics, and evolutionary theory, not for production research-grade game-theoretic analysis
- No changelog, package version number, or R version requirement is stated in the article or repository documentation
- Web interfaces run R on a remote server; specific web interface URLs are not provided in the text

## Evidence

- [abstract] Hawk-dove simulation implemented via learnPopGen: "a new R package called learnPopGen that has been designed primarily for the purposes of teaching evolutionary biology, population genetics, and evolutionary theory"
- [abstract] Package supports simulations and numerical analyses of evolutionary phenomena: "Functions of the package can be used to conduct simulations and numerical analyses of a wide range of evolutionary phenomena"
- [discussion] Interactive R session usage: "Students can use the package within an interactive R session"
- [intro] Source code availability via GitHub: "the source code has been available online via my GitHub page for several years (previously under the alternative moniker, PopGen)."
- [readme] Repository contents: "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
