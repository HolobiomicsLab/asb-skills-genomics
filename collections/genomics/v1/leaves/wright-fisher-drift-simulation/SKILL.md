---
name: wright-fisher-drift-simulation
description: Use when the analysis goal is to simulate or visualize allele-frequency
  change under genetic drift, coalescence of lineages in a Wright–Fisher population,
  or the joint effects of selection and drift at a biallelic locus.
when_to_use_negative:
- The question concerns deterministic allele-frequency change without stochastic drift
  (e.g., pure selection equilibria with no sampling noise), which drift.selection's
  Wright–Fisher binomial sampling is not designed for
- The goal is multi-locus or continuous-trait analysis rather than single biallelic-locus
  drift/selection dynamics, which is outside the scope of the functions documented
  here
- The user cannot use R and needs a point-and-click environment — the article instead
  directs such users to the package's web interfaces, not these functions
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: learnPopGen
  role: R package providing coalescent.plot (coalescence of alleles under drift in
    a Wright–Fisher population) and drift.selection (joint selection-and-drift simulation
    at a biallelic locus) with built-in plotting methods
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Scientific computing environment in which learnPopGen functions are called,
    trajectories extracted, and plots exported as flat graphics such as PNG
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:f5c50913a830df27d3f79eba014a7d3dfddbb31042527d024b190a056db30150
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# wright-fisher-drift-simulation

## Summary

Simulate genetic drift and coalescence in a Wright–Fisher population using the learnPopGen R package's coalescent.plot and drift.selection functions, illustrating Kingman (1982) coalescence and selection-plus-drift allele-frequency trajectories.

## When to use

Use when the analysis goal is to simulate or visualize allele-frequency change under genetic drift, coalescence of lineages in a Wright–Fisher population, or the joint effects of selection and drift at a biallelic locus.

## When NOT to use

- The question concerns deterministic allele-frequency change without stochastic drift (e.g., pure selection equilibria with no sampling noise), which drift.selection's Wright–Fisher binomial sampling is not designed for
- The goal is multi-locus or continuous-trait analysis rather than single biallelic-locus drift/selection dynamics, which is outside the scope of the functions documented here
- The user cannot use R and needs a point-and-click environment — the article instead directs such users to the package's web interfaces, not these functions

## Inputs

- Initial biallelic allele frequency (numeric, [0,1])
- Genotype fitness vector (numeric vector, one fitness per genotype for drift.selection)
- Effective population size (integer)
- Number of generations (integer)
- Number of replicate populations (integer)

## Outputs

- Allele-frequency trajectory object per replicate (learnPopGen return object from drift.selection)
- Generation-by-generation allele frequencies written to drift_selection_trajectories.csv
- Trajectory figure drift_selection_plot.png
- Coalescent genealogy plot coalescent_plot.png showing lineages converging to a common ancestor

## How to apply

Install and load learnPopGen from its GitHub source (github.com/liamrevell/learnPopGen) within an interactive R session. For drift/coalescence visualization, call coalescent.plot with its default coalescent simulation settings to generate a Wright–Fisher population of alleles experiencing genetic drift, then render the genealogy plot in which allele lineages converge backward in time to a common ancestor, and export it as a flat graphic (e.g., coalescent_plot.png). For joint selection-and-drift dynamics at a biallelic locus, call drift.selection supplying the initial allele frequency, genotype fitness vector, effective population size, number of generations, and number of replicate populations; the function performs Wright–Fisher-style binomial sampling of allele frequencies each generation, modulated by the deterministic selection step defined by the genotype fitnesses. Extract the returned per-replicate allele-frequency trajectory object, write generation-by-generation frequencies to CSV, and plot replicate trajectories with the built-in plotting method. Validate that allele-frequency values fall in [0, 1] per generation per replicate and that trajectories show stochastic fluctuation around the selection-driven trend.

## Related tools

- **learnPopGen** (R package providing coalescent.plot (coalescence of alleles under drift in a Wright–Fisher population) and drift.selection (joint selection-and-drift simulation at a biallelic locus) with built-in plotting methods) — github.com/liamrevell/learnPopGen
- **R** (Scientific computing environment in which learnPopGen functions are called, trajectories extracted, and plots exported as flat graphics such as PNG)

## Examples

```
library(learnPopGen); obj <- drift.selection(p0=0.5, w=c(1,1,0.9), Ne=100, ngen=200, nrep=10); plot(obj)
```

## Evaluation signals

- coalescent_plot.png exists, is a valid PNG, and depicts allele lineages coalescing over generations under drift, matching the visual form of Figure 2 of the article
- drift_selection_trajectories.csv contains a frequency row per sampled generation per replicate
- All allele-frequency values in the CSV are within [0, 1]
- Trajectories show the expected stochastic fluctuation around the selection-driven trend
- The drift.selection call received all required arguments: initial allele frequency, genotype fitness vector, effective population size, number of generations, and number of replicate populations

## Limitations

- No package version number is stated for learnPopGen, so exact function signatures may vary with the GitHub source
- No R version requirement is stated; the package is intended for use within an interactive R session
- No changelog is provided for the package
- Web interface URLs for the functions are not provided in the article text

## Evidence

- [results] The package includes the function coalescent.plot, which illustrates genetic coalescence (Kingman, 1982) in a Wright–Fisher population of alleles experiencing genetic drift: [evidence span withheld — anchor status: not_found]
- [results] The learnPopGen package includes the drift.selection function, which simulates the simultaneous effects of selection and genetic drift at a biallelic locus: [evidence span withheld — anchor status: not_found]
- [results] Wright–Fisher-style binomial sampling each generation is the stochastic mechanism underlying the simulation: [evidence span withheld — anchor status: not_found]
- [abstract] The package is designed for use in R for teaching population genetics and evolutionary theory: "a new R package called learnPopGen that has been designed primarily for the purposes of teaching evolutionary biology, population genetics, and evolutionary theory"
- [abstract] Plots can be exported from R as flat graphics, supporting the PNG export steps of this skill: "it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos"
