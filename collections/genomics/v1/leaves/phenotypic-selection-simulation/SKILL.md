---
name: phenotypic-selection-simulation
description: Use when the goal is to demonstrate or teach how directional selection
  shifts a polygenic-trait phenotypic distribution over generations without new mutation,
  or to export that process as an animated GIF from R.
when_to_use_negative:
- The question concerns allele-frequency change under deterministic or stochastic
  processes (e.g. drift or the coalescent) rather than phenotypic trait distributions
  under natural selection — learnPopGen provides separate functions for those phenomena.
- The user is not comfortable in the R environment — the paper recommends guiding
  such students to the author's web interfaces, which run R on a remote server from
  any browser or smartphone.
- The simulation must include new mutation supplying genetic variation — this function
  specifically demonstrates the response using standing genetically based variation
  only.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: learnPopGen
  role: Provides the phenotypic.selection function that simulates and animates directional
    selection on a polygenic-trait phenotypic distribution across generations
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Interactive scientific computing environment in which the simulation runs,
    frames are rendered to the graphics device, and animations are exported
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:90f909597779f1385f2b4b5f3da6b1d15cefd54994c5657d103cdec24b9beb0f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# phenotypic-selection-simulation

## Summary

Simulates and animates directional selection on the phenotypic distribution of a genetically based polygenic trait using learnPopGen's phenotypic.selection function in R, showing the distribution shifting beyond its original range from standing genetic variation alone.

## When to use

Use when the goal is to demonstrate or teach how directional selection shifts a polygenic-trait phenotypic distribution over generations without new mutation, or to export that process as an animated GIF from R.

## When NOT to use

- The question concerns allele-frequency change under deterministic or stochastic processes (e.g. drift or the coalescent) rather than phenotypic trait distributions under natural selection — learnPopGen provides separate functions for those phenomena.
- The user is not comfortable in the R environment — the paper recommends guiding such students to the author's web interfaces, which run R on a remote server from any browser or smartphone.
- The simulation must include new mutation supplying genetic variation — this function specifically demonstrates the response using standing genetically based variation only.

## Inputs

- Simulated polygenic trait distribution (phenotypic values generated within learnPopGen from standing genetic variation)
- Directional-selection parameterization for the phenotypic.selection function (selection acting on the trait mean across generations)

## Outputs

- Sequence of phenotypic-distribution frames rendered in R's graphics device, one per generation
- Animated GIF file phenotypic_selection.gif showing the trait distribution's mean shifting beyond its original range

## How to apply

Install learnPopGen from its GitHub repository (github.com/liamrevell/learnPopGen) and load it in an interactive R session, since the package is designed for teaching population genetics, quantitative genetics, and evolutionary theory. Call the phenotypic.selection function on a simulated polygenic trait distribution, iterating the directional-selection response over generations so the phenotypic distribution is recomputed each generation from standing (genetically based) variation only, with no new mutation introduced. Render the returned sequence of phenotypic-distribution frames in R's graphics device to animate the mean shifting beyond its starting range. Export the frame sequence as an animated GIF (e.g. via the package-supported animation export pathway, which the paper describes as exporting plots and animations 'as flat or animated graphics, or as videos') and write it to phenotypic_selection.gif. The rationale is pedagogical: many learnPopGen functions 'employ compelling graphics and animations' precisely so students can watch evolutionary dynamics unfold.

## Related tools

- **learnPopGen** (Provides the phenotypic.selection function that simulates and animates directional selection on a polygenic-trait phenotypic distribution across generations) — github.com/liamrevell/learnPopGen
- **R** (Interactive scientific computing environment in which the simulation runs, frames are rendered to the graphics device, and animations are exported)

## Examples

```
devtools::install_github("liamrevell/learnPopGen"); library(learnPopGen); phenotypic.selection()
```

## Evaluation signals

- phenotypic_selection.gif exists and is a valid animated GIF containing multiple frames
- Successive frames show the phenotypic distribution's mean shifting beyond its original starting range (visual QC of the rendered animation)
- The shift is produced without introducing new mutation — the distribution each generation is recomputed from standing genetically based variation only
- The frame sequence renders correctly in R's graphics device before export, consistent with the package's design of animating 'changes in phenotypic trait values or distributions under natural selection'

## Limitations

- No package version number or changelog is stated for learnPopGen, so the exact function signature may vary with the GitHub source at time of use
- No R version requirement is stated in the article
- Web interface URLs for running equivalent analyses without R are referenced but not provided in the text

## Evidence

- [abstract] compute and animate the changes in phenotypic trait values or distributions under natural: "to visualize gene frequency changes through time under multiple deterministic and stochastic processes, to compute and animate the changes in phenotypic trait values or distributions under natural"
- [abstract] export plots and animations from R: "it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos"
- [discussion] compelling graphics and animations: "Many functions of the package employ compelling graphics and animations."
- [intro] package purpose statement: "I present a new R package called learnPopGen that I have developed with the expressed purpose of teaching (and/or learning about) population genetics, quantitative genetics, and evolutionary theory."
- [readme] README repository scope: "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
