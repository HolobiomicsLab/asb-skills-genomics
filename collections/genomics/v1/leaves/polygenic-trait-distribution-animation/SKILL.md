---
name: polygenic-trait-distribution-animation
description: Use when teaching or demonstrating quantitative-genetic responses to
  selection on a polygenic trait — specifically when the goal is to render an animated
  sequence of phenotypic distributions that shift under directional selection from
  standing genetic variation only.
when_to_use_negative:
- The question concerns allele-frequency change at one or a few loci under deterministic
  or stochastic processes rather than the distribution of a quantitative polygenic
  trait — use the package's gene-frequency simulation functions instead.
- A static single-generation plot suffices and animation or frame-by-frame export
  is unnecessary overhead.
- The user cannot run R and only needs a conceptual demonstration — the article notes
  web interfaces exist for functions, but this animation workflow requires an R session.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3299
tools:
- name: learnPopGen
  role: Provides the phenotypic.selection function that computes and animates changes
    in phenotypic trait distributions under natural selection across generations
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Interactive computing environment in which the simulation is run, frames are
    rendered, and animations are exported
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:9cc0fe615c6e2001edfce29c259c6e1e359c85aa251856bbbc931c4aa4b46e8d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# polygenic-trait-distribution-animation

## Summary

Animates the shift of a genetically based polygenic trait's phenotypic distribution under directional selection using learnPopGen's phenotypic.selection function in R, showing that the mean can move far beyond its original range from standing variation alone, without new mutation.

## When to use

Use when teaching or demonstrating quantitative-genetic responses to selection on a polygenic trait — specifically when the goal is to render an animated sequence of phenotypic distributions that shift under directional selection from standing genetic variation only.

## When NOT to use

- The question concerns allele-frequency change at one or a few loci under deterministic or stochastic processes rather than the distribution of a quantitative polygenic trait — use the package's gene-frequency simulation functions instead.
- A static single-generation plot suffices and animation or frame-by-frame export is unnecessary overhead.
- The user cannot run R and only needs a conceptual demonstration — the article notes web interfaces exist for functions, but this animation workflow requires an R session.

## Inputs

- A simulated polygenic trait phenotypic distribution in R (starting trait values / standing genetic variation)
- Directional-selection parameters for phenotypic.selection (selection acting on the trait mean over generations)

## Outputs

- A sequence of per-generation phenotypic-distribution frames rendered in R's graphics device
- An animated GIF file (e.g. phenotypic_selection.gif) exported from R

## How to apply

Install learnPopGen from its GitHub repository (github.com/liamrevell/learnPopGen) and load it in an interactive R session. Call phenotypic.selection on a simulated polygenic trait distribution, iterating the directional-selection response over generations so the distribution is recomputed each generation from standing (genetically based) variation only, with no new mutation introduced. Render the returned sequence of distribution frames in R's graphics device, then export the frame sequence as an animated GIF via the animation export pathway the package supports (the paper notes it is straightforward to export animations from R in the form of animated graphics or videos). Write the result to phenotypic_selection.gif and inspect it visually. The rationale is pedagogical: learnPopGen was designed primarily for teaching evolutionary biology, population genetics, and evolutionary theory, and many of its functions employ compelling graphics and animations.

## Related tools

- **learnPopGen** (Provides the phenotypic.selection function that computes and animates changes in phenotypic trait distributions under natural selection across generations) — github.com/liamrevell/learnPopGen
- **R** (Interactive computing environment in which the simulation is run, frames are rendered, and animations are exported)

## Examples

```
library(learnPopGen); anim <- phenotypic.selection(p=0, ngen=100, ...); saveGIF(anim)  # write phenotypic_selection.gif
```

## Evaluation signals

- phenotypic_selection.gif exists on disk.
- The file is a valid animated GIF containing multiple frames.
- Successive frames show the phenotypic distribution's mean shifting beyond its original range under directional selection (visual QC of the rendered animation).
- The recomputed distributions derive from standing genetically based variation only — no new mutation was introduced in the simulation call.
- Export pathway used is one the package supports for flat or animated graphics or videos.

## Limitations

- No package version number or changelog is stated for learnPopGen, so reproducibility of exact animation behavior across versions is unverified.
- No R version requirement is stated in the article, so environment compatibility must be confirmed empirically.
- The demonstration relies on standing genetic variation: the shift beyond the original range is achieved without new mutation, so the animation is not a model of mutation-limited adaptation.

## Evidence

- [abstract] Package functions support visualizing gene frequency changes, animating phenotypic trait changes under selection, game theory model analysis, and plotting coalescence under genetic drift: "to visualize gene frequency changes through time under multiple deterministic and stochastic processes, to compute and animate the changes in phenotypic trait values or distributions under natural"
- [abstract] Plots and animations exportable from R: "it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos"
- [intro] Package purpose is teaching population genetics and evolutionary theory: "I present a new R package called learnPopGen that I have developed with the expressed purpose of teaching (and/or learning about) population genetics, quantitative genetics, and evolutionary theory."
- [discussion] Many functions employ compelling graphics and animations: "Many functions of the package employ compelling graphics and animations."
- [discussion] Students use the package in an interactive R session: "Students can use the package within an interactive R session"
- [readme] README describes repository contents: "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
