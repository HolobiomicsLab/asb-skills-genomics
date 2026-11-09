---
name: directional-selection-response-visualization
description: Use when you need to demonstrate or inspect the response of a quantitative/polygenic
  trait distribution to directional selection across generations, especially to show
  that response is possible from standing genetic variation alone without new mutation.
when_to_use_negative:
- The question concerns allele-frequency change under deterministic or stochastic
  processes at a single locus rather than shifts of a quantitative phenotypic distribution
  — other learnPopGen functions cover gene-frequency dynamics.
- New mutation is part of the model and you need mutation-driven response rather than
  response from standing genetic variation only.
- You need numerical population-genetic analysis without animation/visualization,
  e.g. purely tabular simulation output.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0625
tools:
- name: learnPopGen
  role: Provides the phenotypic.selection function that computes and animates the
    change in phenotypic trait distributions under natural selection; also supports
    exporting animations
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Interactive scientific computing environment in which the simulation is run
    and the distribution frames are rendered and exported
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:ef059dfbe8597c77c36ac8d0bd5786091a937362445a41a609a37a7e4ede51b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# directional-selection-response-visualization

## Summary

Animates how directional selection shifts the phenotypic distribution of a genetically based polygenic trait beyond its original range using learnPopGen's phenotypic.selection function in R, which recomputes the trait distribution each generation from standing variation only (no new mutation).

## When to use

Use when you need to demonstrate or inspect the response of a quantitative/polygenic trait distribution to directional selection across generations, especially to show that response is possible from standing genetic variation alone without new mutation.

## When NOT to use

- The question concerns allele-frequency change under deterministic or stochastic processes at a single locus rather than shifts of a quantitative phenotypic distribution — other learnPopGen functions cover gene-frequency dynamics.
- New mutation is part of the model and you need mutation-driven response rather than response from standing genetic variation only.
- You need numerical population-genetic analysis without animation/visualization, e.g. purely tabular simulation output.

## Inputs

- A simulated polygenic (quantitative) trait distribution with genetically based variation, passed to the phenotypic.selection function
- Directional-selection simulation parameters (generations over which the response is iterated)

## Outputs

- A sequence of phenotypic-distribution frames rendered in R's graphics device
- An exported animated GIF (e.g. phenotypic_selection.gif) showing the trait distribution shifting beyond its original range

## How to apply

Install learnPopGen from its GitHub repository (github.com/liamrevell/learnPopGen) and load it in an interactive R session. Call the phenotypic.selection function on a simulated polygenic trait distribution, iterating the directional-selection response over generations so the distribution is recomputed each generation from standing (genetically based) variation only. Render the returned sequence of phenotypic-distribution frames in R's graphics device to visualize the distribution's mean shifting beyond its starting range. Export the frame sequence as an animated GIF (learnPopGen supports exporting plots and animations from R as flat or animated graphics, or as videos) and write it to phenotypic_selection.gif. The rationale is pedagogical: the package was developed for teaching population genetics, quantitative genetics, and evolutionary theory, and many of its functions employ graphics and animations for exactly this purpose.

## Related tools

- **learnPopGen** (Provides the phenotypic.selection function that computes and animates the change in phenotypic trait distributions under natural selection; also supports exporting animations) — github.com/liamrevell/learnPopGen
- **R** (Interactive scientific computing environment in which the simulation is run and the distribution frames are rendered and exported)

## Examples

```
library(learnPopGen); phenotypic.selection()  # run in an interactive R session after installing from github.com/liamrevell/learnPopGen
```

## Evaluation signals

- phenotypic_selection.gif exists and is a valid animated GIF containing multiple frames.
- Successive frames show the phenotypic distribution's mean shifting beyond its original range under directional selection (visual QC of the rendered animation).
- The shift arises from standing (genetically based) variation only — no new mutation is introduced during the iteration.
- Each rendered frame is recomputed from the previous generation's distribution, i.e. the animation corresponds to an iterated generational response rather than a single static plot.

## Limitations

- Demonstrates response from standing genetic variation in the absence of new mutation; it is not a mutation-driven adaptation simulation.
- No package version number or changelog is stated in the article, and no R version requirement is given, so reproducibility of exact function behavior across versions is not guaranteed.
- Web interfaces exist for some functions, but their URLs are not provided in the article text; visualization must be done in R otherwise.

## Evidence

- [abstract] to compute and animate the changes in phenotypic trait values or distributions under natural: "to visualize gene frequency changes through time under multiple deterministic and stochastic processes, to compute and animate the changes in phenotypic trait values or distributions under natural"
- [abstract] it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos: "it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos"
- [discussion] instructors can export plots and animations as flat graphics or animations for use in lecture: "instructors can export plots and animations as flat graphics or animations for use in lecture"
- [readme] This repository contains source code, documentation, and R data for the project: "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
- [discussion] Many functions of the package employ compelling graphics and animations.: "Many functions of the package employ compelling graphics and animations."
