---
name: relative-fitness-frequency-update-implementation
description: Use when simulating evolutionary game dynamics in R where two behavioral
  strategies interact at their current frequencies and relative fitnesses derived
  from a payoff matrix must update those frequencies each generation.
when_to_use_negative:
- The question concerns deterministic or stochastic allele-frequency change under
  selection or drift rather than frequency-dependent game-theoretic strategies
- A ready-made learnPopGen function suffices and reimplementing the update loop is
  unnecessary
- Students less adept in R would be better served by one of the package's web interfaces
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0610
- http://edamontology.org/topic_3299
tools:
- name: learnPopGen
  role: Reference implementation of the hawk-dove game model whose frequency update
    by relative fitness this skill reproduces; also the source of plotting/animation
    conventions
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Interactive computing environment in which the payoff matrix, per-generation
    frequency update, and base R trajectory plot are implemented
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:25f9cafa817a9b39e54bf55416468af61e81930efbcde896e4018d1e81330b03
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0610
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Relative-fitness frequency-update implementation

## Summary

Implements the frequency-dependent dynamics of the learnPopGen hawk-dove game model: each generation, strategy payoffs from a user-specified matrix are normalized into relative fitnesses and multiplied by current strategy frequencies to evolve hawk-dove proportions through time.

## When to use

Use when simulating evolutionary game dynamics in R where two behavioral strategies interact at their current frequencies and relative fitnesses derived from a payoff matrix must update those frequencies each generation.

## When NOT to use

- The question concerns deterministic or stochastic allele-frequency change under selection or drift rather than frequency-dependent game-theoretic strategies
- A ready-made learnPopGen function suffices and reimplementing the update loop is unnecessary
- Students less adept in R would be better served by one of the package's web interfaces

## Inputs

- Initial frequency of hawks (numeric value in [0, 1])
- Pay-off matrix with entries for hawk-hawk, hawk-dove, and dove-dove interaction outcomes
- Number of generations or convergence criterion

## Outputs

- Data frame of per-generation hawk and dove frequencies (e.g. hawkdove_frequencies.csv)
- Plot of strategy frequencies through time (base R graphics)

## How to apply

Define the hawk-dove pay-off matrix with explicit entries for hawk-hawk, hawk-dove, and dove-dove interaction outcomes, and choose an initial frequency of hawks as the simulation starting point. Each generation, have hawks and doves interact in the proportions they occur, computing the expected payoff of each strategy (hawk and dove) given the current strategy frequencies. Normalize those payoffs into relative fitness values and multiply the current hawk and dove frequencies by their relative fitnesses, iterating for a fixed number of generations or until convergence. Assemble the per-generation hawk and dove frequencies into a data frame (e.g. written to hawkdove_frequencies.csv) and plot the strategy-frequency trajectory through time with base R graphics, mirroring the game theory model analysis functionality of learnPopGen.

## Related tools

- **learnPopGen** (Reference implementation of the hawk-dove game model whose frequency update by relative fitness this skill reproduces; also the source of plotting/animation conventions) — github.com/liamrevell/learnPopGen
- **R** (Interactive computing environment in which the payoff matrix, per-generation frequency update, and base R trajectory plot are implemented)

## Examples

```
Rscript -e 'source hawkdove simulation; w <- payoffs %*% c(p, 1-p); p <- p * (w[1] / sum(w * c(p, 1-p))); write.csv(freqs, "hawkdove_frequencies.csv")'
```

## Evaluation signals

- All simulated hawk and dove frequencies remain within [0, 1] at every generation
- Hawk and dove frequencies sum to 1 at every generation
- The trajectory converges to the expected frequency-dependent equilibrium implied by the specified pay-off matrix
- Per-generation output data frame has consistent generation indices and matches the values plotted in the trajectory figure

## Limitations

- No package version number or R version requirement is stated for learnPopGen in the article
- The model assumes only two strategies whose interaction proportions equal their current frequencies; payoff structure is fully specified by the user-supplied matrix
- Plots and animations from learnPopGen are intended for teaching contexts rather than production analysis

## Evidence

- [full_text] hawk-dove game function inputs: [evidence span withheld — anchor status: not_found]
- [full_text] evolution by interaction proportions and relative fitnesses: [evidence span withheld — anchor status: not_found]
- [abstract] game theory model analysis among package functions: "to visualize gene frequency changes through time under multiple deterministic and stochastic processes, to compute and animate the changes in phenotypic trait values or distributions under natural"
- [discussion] interactive R session usage: "Students can use the package within an interactive R session"
