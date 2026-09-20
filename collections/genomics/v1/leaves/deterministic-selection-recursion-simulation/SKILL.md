---
name: deterministic-selection-recursion-simulation
description: Use when reproducing Figures 1 and 3 of Revell (2019) or analyzing how
  allele-frequency trajectories and equilibria (e.g., stable coexistence under wAa>wAA≥waa)
  depend on genotype fitness vectors.
when_to_use_negative:
- The question involves stochastic processes (genetic drift, coalescence) rather than
  deterministic genotypic selection — learnPopGen handles those via separate functions
- Analysis targets quantitative-trait or game-theory dynamics, which selection() does
  not model
- The user is not working in R and only needs a quick demonstration — the paper notes
  web interfaces built for students 'less adept in the R environment'
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: learnPopGen
  role: Provides the selection() function that performs the deterministic biallelic
    genotypic-selection numerical analysis and produces the trajectory, mean-fitness,
    delta-p, cobweb, and landscape plots
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Interactive scientific computing environment in which the selection() recursion
    is run and from which plots are exported via png()/pdf() devices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:9b2150cc4837c389b0328b98bb6531272f8cdceff0a7bd6e0bffed82c471fe09
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# deterministic-selection-recursion-simulation

## Summary

Reproduces the learnPopGen selection() function's deterministic biallelic genotypic-selection numerical analysis in R, generating allele-frequency trajectories, mean-fitness curves, delta-p, cobweb, and fitness-landscape plots across dominance regimes (overdominance, underdominance, complete dominance, recessive benefit, codominance).

## When to use

Use when reproducing Figures 1 and 3 of Revell (2019) or analyzing how allele-frequency trajectories and equilibria (e.g., stable coexistence under wAa>wAA≥waa) depend on genotype fitness vectors.

## When NOT to use

- The question involves stochastic processes (genetic drift, coalescence) rather than deterministic genotypic selection — learnPopGen handles those via separate functions
- Analysis targets quantitative-trait or game-theory dynamics, which selection() does not model
- The user is not working in R and only needs a quick demonstration — the paper notes web interfaces built for students 'less adept in the R environment'

## Inputs

- Three-element genotype fitness vector w = (wAA, wAa, waa) per dominance regime
- Initial allele frequency p0 for the deterministic recursion (critical for underdominance outcomes)
- Plot-type selector string: "p", "fitness", "deltap", or "cobweb"

## Outputs

- Allele-frequency trajectory through time (p vs t plot)
- Mean fitness versus time plot
- Delta-p versus p recursion curve
- Cobweb plot of the selection recursion
- Multi-panel figure reproducing Figures 1 and 3, exported as png()/pdf() graphics

## How to apply

Install learnPopGen from github.com/liamrevell/learnPopGen via devtools::install_github, then call selection() with a three-element fitness vector w = (wAA, wAa, waa) and show="p" to plot allele frequency through time. Sweep the dominance regimes: overdominance (wAa > wAA >= waa, e.g. w=c(0.9,1.0,0.7)) yields stable polymorphism; underdominance yields an unstable internal equilibrium whose fixation outcome depends on the initial allele frequency; complete dominance (wAA=wAa>waa), recessive benefit, and codominance (wAA>wAa>waa) each get their own w vector. For each regime, re-run selection() with show="fitness" for mean fitness versus time and show="deltap" and show="cobweb" for the delta-p versus p recursion curve and cobweb plot. Assemble trajectory, mean-fitness, landscape, delta-p, and cobweb panels into a multi-page/panel figure and export with png()/pdf() graphics devices, as R exports 'flat or animated graphics'.

## Related tools

- **learnPopGen** (Provides the selection() function that performs the deterministic biallelic genotypic-selection numerical analysis and produces the trajectory, mean-fitness, delta-p, cobweb, and landscape plots) — github.com/liamrevell/learnPopGen
- **R** (Interactive scientific computing environment in which the selection() recursion is run and from which plots are exported via png()/pdf() devices)

## Examples

```
devtools::install_github("liamrevell/learnPopGen"); library(learnPopGen); selection(w=c(0.9,1.0,0.7), show="p"); selection(w=c(0.9,1.0,0.7), show="fitness")
```

## Evaluation signals

- Under overdominance fitnesses with wAa>wAA≥waa (e.g. w=c(0.9,1.0,0.7)), the trajectory converges to a stable polymorphic equilibrium with both alleles present (Figure 1a behavior)
- Under underdominance, an unstable internal equilibrium appears and the final state (fixation of A or a) depends on the initial frequency (Figure 3b behavior)
- All four plot types (p, fitness, deltap, cobweb) render without error for each of the five fitness regimes
- Exported multi-panel figure opens correctly from png()/pdf() devices and matches the panel layout of Figures 1 and 3

## Limitations

- The selection() model is purely deterministic — it does not include genetic drift or other stochastic processes
- No package version number or R version requirement is stated in the article; no changelog is provided for the package
- Equilibrium predictions are specific to biallelic genotypic selection with constant fitness; the analysis does not cover frequency-dependent selection

## Evidence

- [results] The selection function shows that coexistence of both alleles at equilibrium is expected only under overdominance for fitness (wAa>wAA≥waa, Figure 1a), while underdominance produces an unstable equilibrium with fixation depending on initial frequency: [evidence span withheld — anchor status: not_found]
- [abstract] Package purpose and scope: "a new R package called learnPopGen that has been designed primarily for the purposes of teaching evolutionary biology, population genetics, and evolutionary theory"
- [abstract] Export capability: "it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos"
- [readme] README description of repository contents: "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
- [discussion] Web interface alternative for non-R users: "instructors may choose to guide students less adept in the R environment to one of various web interfaces that I have built for a number of the functions"
