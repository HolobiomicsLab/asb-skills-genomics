---
name: allele-frequency-trajectory-simulation
description: Use when the task requires simulating or visualizing how allele frequencies
  change through time at a biallelic locus under selection, drift, or their combination
  — e.g., reproducing dominance-regime dynamics (overdominance, underdominance, codominance)
  or Wright–Fisher drift-selection trajectories.
when_to_use_negative:
- The question concerns phenotypic trait evolution or game theory dynamics rather
  than biallelic allele-frequency change — learnPopGen provides separate functions
  for those phenomena
- A multi-allelic or continuous-trait model is required; selection() and drift.selection()
  are defined for a biallelic locus with three genotype fitnesses
- The user cannot use R and only needs static visualizations — the article notes web
  interfaces exist for many functions for students 'less adept in the R environment'
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3056
tools:
- name: learnPopGen
  role: Provides the selection() and drift.selection() functions that run the numerical
    analysis and Wright–Fisher simulations and plot allele-frequency trajectories
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Interactive computing environment in which the package functions are called
    and from which plots/animations are exported
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:f7a1f423de78cfe99d5eeca935020bce7d99c39898904f22ae66273b8633f4f1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3056
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# allele-frequency-trajectory-simulation

## Summary

Simulates allele-frequency trajectories at a biallelic locus under deterministic selection and stochastic genetic drift using the learnPopGen R package's selection() and drift.selection() functions, exporting generation-by-generation trajectories and trajectory plots.

## When to use

Use when the task requires simulating or visualizing how allele frequencies change through time at a biallelic locus under selection, drift, or their combination — e.g., reproducing dominance-regime dynamics (overdominance, underdominance, codominance) or Wright–Fisher drift-selection trajectories.

## When NOT to use

- The question concerns phenotypic trait evolution or game theory dynamics rather than biallelic allele-frequency change — learnPopGen provides separate functions for those phenomena
- A multi-allelic or continuous-trait model is required; selection() and drift.selection() are defined for a biallelic locus with three genotype fitnesses
- The user cannot use R and only needs static visualizations — the article notes web interfaces exist for many functions for students 'less adept in the R environment'

## Inputs

- Initial allele frequency (numeric, [0,1])
- Genotype fitness vector w = c(wAA, wAa, waa) defining the dominance regime
- Effective population size (for drift.selection)
- Number of generations to simulate
- Number of replicate populations (for drift.selection)

## Outputs

- Generation-by-generation allele-frequency trajectory object per replicate
- drift_selection_trajectories.csv with a frequency row per sampled generation per replicate
- Trajectory, mean-fitness, delta-p, landscape, and cobweb plots (drift_selection_plot.png or multi-panel figure)

## How to apply

Install learnPopGen in R via devtools::install_github('liamrevell/learnPopGen'). For deterministic genotypic selection, call selection() with a genotype fitness vector w=c(wAA, wAa, waa) defining the dominance regime (e.g., overdominance w=c(0.9, 1.0, 0.7) where wAa>wAA>=waa), and use show='p' for the allele-frequency trajectory, show='fitness' for mean fitness, and show='deltap'/'cobweb' for the delta-p and cobweb plots. For combined selection and drift, call drift.selection() supplying initial allele frequency, genotype fitness vector, effective population size, number of generations, and number of replicate populations; the function performs Wright–Fisher-style binomial sampling each generation modulated by the deterministic selection step. Write generation-by-generation frequencies per replicate to CSV and export trajectory plots via the built-in plotting method using png()/pdf() devices.

## Related tools

- **learnPopGen** (Provides the selection() and drift.selection() functions that run the numerical analysis and Wright–Fisher simulations and plot allele-frequency trajectories) — github.com/liamrevell/learnPopGen
- **R** (Interactive computing environment in which the package functions are called and from which plots/animations are exported)

## Examples

```
selection(w=c(0.9, 1.0, 0.7), show="p"); drift.selection(p0=0.5, w=c(0.9, 1.0, 0.7), Ne=100, ngen=200, nrep=10)
```

## Evaluation signals

- CSV contains a frequency row per sampled generation per replicate, with all values in [0, 1]
- Under drift.selection, trajectories show stochastic fluctuation around the selection-driven trend
- Under overdominance (wAa>wAA>=waa), both alleles coexist at a stable equilibrium; under underdominance, the equilibrium is unstable and fixation depends on initial frequency
- Plots exported successfully as flat graphics (png/pdf) via R graphics devices
- Trajectory object returned by drift.selection is plottable through the function's built-in plotting method

## Limitations

- No package version number, changelog, or R version requirement is stated in the article
- Web interface URLs are not provided in the text, limiting the no-R fallback path
- Models are biallelic-locus Wright–Fisher-style teaching simulations, not inference tools for empirical genomic data

## Evidence

- [abstract] drift.selection produces allele-frequency trajectories under combined selection and drift: "a new R package called learnPopGen that has been designed primarily for the purposes of teaching evolutionary biology, population genetics, and evolutionary theory"
- [abstract] Package functions visualize gene frequency changes through time under deterministic and stochastic processes: "to visualize gene frequency changes through time under multiple deterministic and stochastic processes"
- [abstract] Plots and animations can be exported from R as flat or animated graphics: "it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos"
- [discussion] Students can run the package in an interactive R session: "Students can use the package within an interactive R session"
- [readme] Repository contains source code and documentation for learnPopGen: "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
