---
name: dominance-regime-comparative-analysis
description: Use when the task is to reproduce or compare deterministic biallelic
  selection dynamics across dominance/fitness regimes (e.g., wAa>wAA>=waa overdominance
  vs. underdominance) and identify equilibria, instability, and fixation behavior
  from given genotype fitnesses.
when_to_use_negative:
- The analysis involves stochastic processes (e.g., genetic drift simulations) rather
  than the deterministic genotypic selection model handled by selection()
- The question concerns phenotypic trait change under natural selection or game-theory
  models, which are separate learnPopGen functions, not biallelic fitness regimes
- The user has no R environment and cannot install the package from GitHub; the article
  notes web interfaces exist for such users but this skill's workflow is R-based
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3056
tools:
- name: learnPopGen
  role: Provides the selection() function for numerical analysis and plotting of the
    biallelic genotypic selection model across dominance regimes; installed from GitHub
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Interactive computing environment in which the package is loaded, selection()
    is invoked with each fitness vector, and panels are exported with png()/pdf()
    graphics devices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:eb533477f3a894803e749447bc9754a11f02f045703753f8366689cdba58203e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3056
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# dominance-regime-comparative-analysis

## Summary

Numerically analyze a biallelic genotypic selection model in R with the learnPopGen selection() function across all dominance regimes (overdominance, complete dominance, recessive benefit, codominance, underdominance), producing allele-frequency trajectories, mean-fitness curves, delta-p landscapes, and cobweb plots that reproduce Figures 1 and 3 of the paper.

## When to use

Use when the task is to reproduce or compare deterministic biallelic selection dynamics across dominance/fitness regimes (e.g., wAa>wAA>=waa overdominance vs. underdominance) and identify equilibria, instability, and fixation behavior from given genotype fitnesses.

## When NOT to use

- The analysis involves stochastic processes (e.g., genetic drift simulations) rather than the deterministic genotypic selection model handled by selection()
- The question concerns phenotypic trait change under natural selection or game-theory models, which are separate learnPopGen functions, not biallelic fitness regimes
- The user has no R environment and cannot install the package from GitHub; the article notes web interfaces exist for such users but this skill's workflow is R-based

## Inputs

- Three-element genotype fitness vector w = c(wAA, wAa, waa) defining the dominance regime
- Initial allele frequency p0 and number of generations for the deterministic iteration
- Plot-type selector for selection(): one of show='p', 'fitness', 'deltap', 'cobweb'

## Outputs

- Allele-frequency trajectory through time (show='p')
- Mean fitness versus time plot (show='fitness')
- Delta-p versus p curve (show='deltap')
- Cobweb plot of the recursion (show='cobweb')
- Multi-panel/multi-page figure reproducing paper Figures 1 and 3, exported via png()/pdf()

## How to apply

Install learnPopGen from GitHub via devtools::install_github('liamrevell/learnPopGen'), then call selection() with a three-element fitness vector w=c(wAA,wAa,waa) ordered by genotype. Set the dominance regime by choosing relative fitnesses: overdominance wAa>wAA>=waa (e.g., w=c(0.9,1.0,0.7)) yields a stable polymorphic equilibrium with both alleles coexisting (Figure 1a), while underdominance (wAA>wAa<waa or wAA<wAa>waa) yields an unstable equilibrium where outcome depends on the initial allele frequency (Figure 3b). Generate the full panel set per regime by cycling the show argument through 'p' (allele-frequency trajectory through time), 'fitness' (mean fitness vs. time), 'deltap' (delta-p versus p curve), and 'cobweb' (cobweb plot). Repeat for complete-dominance (wAA=wAa>waa), recessive-benefit, and codominance (wAA>wAa>waa) vectors, then assemble trajectory, mean-fitness, landscape, delta-p, and cobweb panels into a single multi-page/panel figure and export with png()/pdf() devices.

## Related tools

- **learnPopGen** (Provides the selection() function for numerical analysis and plotting of the biallelic genotypic selection model across dominance regimes; installed from GitHub) — github.com/liamrevell/learnPopGen
- **R** (Interactive computing environment in which the package is loaded, selection() is invoked with each fitness vector, and panels are exported with png()/pdf() graphics devices)

## Examples

```
selection(p=0.01, w=c(0.9,1.0,0.7), show="p")
```

## Evaluation signals

- Under overdominance fitnesses such as w=c(0.9,1.0,0.7), the show='p' trajectory converges to an internal equilibrium with both alleles coexisting, matching Figure 1a
- Under underdominance, the show='p' trajectory exhibits an unstable equilibrium: trajectories starting above the threshold fix for A, those below fix for a, matching Figure 3b
- The show='fitness' panel shows mean fitness increasing monotonically over time under directional regimes, and the show='deltap' curve changes sign at the internal equilibrium
- All four plot types (p, fitness, deltap, cobweb) are produced for each of the five regimes (overdominance, complete dominance, recessive benefit, codominance, underdominance)
- Exported files render correctly from png()/pdf() devices as a multi-page/panel figure

## Limitations

- The selection() model is deterministic; stochastic dynamics such as drift are handled by other package functions and are not covered by this regime comparison
- Equilibria and fixation outcomes depend on the initial allele frequency under underdominance, so a single starting p0 is insufficient to characterize that regime
- No package version number, changelog, or R version requirement is stated in the article, so reproducibility across package versions is unverified

## Evidence

- [abstract] Package functions support simulations and numerical analyses of evolutionary phenomena: "Functions of the package can be used to conduct simulations and numerical analyses of a wide range of evolutionary phenomena"
- [intro] Package developed for teaching population genetics and evolutionary theory: "I present a new R package called learnPopGen that I have developed with the expressed purpose of teaching (and/or learning about) population genetics, quantitative genetics, and evolutionary theory."
- [discussion] Instructors can export plots and animations as flat graphics for lecture use: "instructors can export plots and animations as flat graphics or animations for use in lecture"
- [readme] README identifies the repository as source code, documentation, and R data: "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
