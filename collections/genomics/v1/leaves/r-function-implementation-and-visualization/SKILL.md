---
name: r-function-implementation-and-visualization
description: Use when implementing an evolutionary model (e.g. hawk-dove game with
  a user-specified pay-off matrix and strategy-frequency dynamics) in R and producing
  per-generation trajectories plus plots/animations.
when_to_use_negative:
- The learner is less adept in R — the article recommends guiding them to one of the
  package's web interfaces instead
- The analysis requires stochastic population-genetic simulation or coalescence under
  drift rather than a deterministic game-theory frequency update
- A pre-existing learnPopGen function already covers the model and only its default
  parameters need adjusting
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3299
tools:
- name: learnPopGen
  role: R package whose source (loaded from GitHub) serves as reference implementation
    of the hawk-dove game model; its functions perform simulations, numerical analyses,
    plotting, and animation
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Scientific computing environment in which the hawk-dove frequency update is
    implemented, trajectories are written to CSV, and base R graphics render the plots
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:4b1c3dbafc1e1e095d4c67d39eb11a428620afecdfae5429102ba338cb4bda4b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# r-function-implementation-and-visualization

## Summary

Implement a population-genetics or evolutionary-game-theory model as an R function and visualize its dynamics, following the learnPopGen package's approach of using R simulations and graphics/animations to teach evolutionary theory.

## When to use

Use when implementing an evolutionary model (e.g. hawk-dove game with a user-specified pay-off matrix and strategy-frequency dynamics) in R and producing per-generation trajectories plus plots/animations.

## When NOT to use

- The learner is less adept in R — the article recommends guiding them to one of the package's web interfaces instead
- The analysis requires stochastic population-genetic simulation or coalescence under drift rather than a deterministic game-theory frequency update
- A pre-existing learnPopGen function already covers the model and only its default parameters need adjusting

## Inputs

- R pay-off matrix with explicit entries for hawk-hawk, hawk-dove, and dove-dove interaction outcomes
- Initial frequency of 'hawks' (numeric in [0,1]) as the simulation starting point
- Number of generations or a convergence criterion

## Outputs

- hawkdove_frequencies.csv containing per-generation hawk and dove frequencies
- Base R plot (or animation/video export) of strategy-frequency trajectories through time

## How to apply

Open an R session and load the learnPopGen source from github.com/liamrevell/learnPopGen as reference context for the model's implementation. Define the model parameters explicitly (for the hawk-dove game: an initial frequency of 'hawks' and a pay-off matrix with entries for hawk-hawk, hawk-dove, and dove-dove interactions). Iterate generation by generation: compute each strategy's expected payoff given current frequencies, normalize payoffs into relative fitness values, and multiply current frequencies by those fitnesses, updating until a fixed number of generations or convergence. Assemble per-generation frequencies into a data frame, write it to CSV (e.g. hawkdove_frequencies.csv), and plot the trajectory through time with base R graphics or package animations, which can be exported as flat graphics or videos.

## Related tools

- **learnPopGen** (R package whose source (loaded from GitHub) serves as reference implementation of the hawk-dove game model; its functions perform simulations, numerical analyses, plotting, and animation) — github.com/liamrevell/learnPopGen
- **R** (Scientific computing environment in which the hawk-dove frequency update is implemented, trajectories are written to CSV, and base R graphics render the plots)

## Examples

```
Rscript -e 'popgen::hawkdove(p0=0.2, W=matrix(c(-0.5,0.8,1.0,0.5),2,2))' ; write.csv(freq_df, "hawkdove_frequencies.csv"); plot(hawk ~ generation, data=freq_df, type="l")
```

## Evaluation signals

- All simulated hawk and dove frequencies remain within [0, 1] at every generation
- The two strategy frequencies sum to 1 at every generation
- The trajectory converges to the expected frequency-dependent equilibrium implied by the specified pay-off matrix
- hawkdove_frequencies.csv contains one row per generation with columns for both strategy frequencies
- Exported plot/animation renders without error and matches the CSV trajectory

## Limitations

- No changelog and no package version number are stated for learnPopGen, so reproducibility depends on which GitHub snapshot is used
- No R version requirement is stated in the article
- Web interface URLs are not provided in the article text

## Evidence

- [other] hawk-dove game function takes as inputs an initial frequency of hawks and a pay-off matrix: [evidence span withheld — anchor status: not_found]
- [abstract] package purpose: "a new R package called learnPopGen that has been designed primarily for the purposes of teaching evolutionary biology, population genetics, and evolutionary theory"
- [abstract] export of graphics: "it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos"
- [discussion] interactive session: "Students can use the package within an interactive R session"
- [discussion] web interface for novices: [evidence span withheld — anchor status: not_found]
- [readme] README project description: "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
