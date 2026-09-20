---
name: coalescent-genealogy-plotting
description: 'Use when teaching or demonstrating genetic drift and coalescence (Kingman,
  1982): you need to simulate allele lineages in a Wright–Fisher population and render
  the resulting coalescent genealogy as a plot or exported flat graphic.'
when_to_use_negative:
- The goal is visualizing allele/gene frequency changes through time under deterministic
  or stochastic processes rather than backward-time coalescent genealogies (use the
  corresponding learnPopGen frequency-visualization functions instead)
- The audience is students less adept in R and no exported graphic is required — the
  article suggests guiding such users to the package's web interfaces, which run R
  on a remote server
- The analysis concerns phenotypic trait changes under natural selection or game theory
  models, which are separate learnPopGen function families
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_3056
- http://edamontology.org/topic_0625
tools:
- name: learnPopGen
  role: Provides the coalescent.plot function that simulates coalescence of alleles
    in a Wright–Fisher population under genetic drift and renders the genealogy plot
    (Figure 2)
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Scientific computing environment in which learnPopGen is run interactively
    and from which the coalescent plot is exported as a flat graphic
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:b6487eb1bf64d4ac6a82d90dad405a8de4b5ebcf460562b6f9a309f9e2116b89
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_3056
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# coalescent-genealogy-plotting

## Summary

Simulate and plot the coalescence of allele lineages in a Wright–Fisher population experiencing genetic drift using the coalescent.plot function of the learnPopGen R package (Revell, 2019), producing a genealogy graphic in which lineages converge backward in time to a common ancestor.

## When to use

Use when teaching or demonstrating genetic drift and coalescence (Kingman, 1982): you need to simulate allele lineages in a Wright–Fisher population and render the resulting coalescent genealogy as a plot or exported flat graphic.

## When NOT to use

- The goal is visualizing allele/gene frequency changes through time under deterministic or stochastic processes rather than backward-time coalescent genealogies (use the corresponding learnPopGen frequency-visualization functions instead)
- The audience is students less adept in R and no exported graphic is required — the article suggests guiding such users to the package's web interfaces, which run R on a remote server
- The analysis concerns phenotypic trait changes under natural selection or game theory models, which are separate learnPopGen function families

## Inputs

- Coalescent simulation parameters accepted by coalescent.plot (function defaults: Wright–Fisher population of alleles experiencing genetic drift)
- Interactive R session with learnPopGen installed from github.com/liamrevell/learnPopGen

## Outputs

- On-screen coalescent genealogy plot of allele lineages converging to a common ancestor (visual form of Figure 2)
- Exported flat graphic file (e.g. coalescent_plot.png) suitable for lecture use

## How to apply

Install and load the learnPopGen package in R from its GitHub repository (github.com/liamrevell/learnPopGen). Call coalescent.plot with the function's default coalescent simulation settings to run a Wright–Fisher drift simulation and obtain the allele coalescent genealogy. Render the returned object as a plot whose visual form matches Figure 2 of the article — allele lineages converging backward in time to a common ancestor. Export the plot from R as a flat graphic file (e.g. coalescent_plot.png); the article notes it is straightforward to export plots and animations from R as flat or animated graphics. Validate that the exported file exists, is a valid PNG, and depicts lineages coalescing over generations under drift.

## Related tools

- **learnPopGen** (Provides the coalescent.plot function that simulates coalescence of alleles in a Wright–Fisher population under genetic drift and renders the genealogy plot (Figure 2)) — github.com/liamrevell/learnPopGen
- **R** (Scientific computing environment in which learnPopGen is run interactively and from which the coalescent plot is exported as a flat graphic)

## Examples

```
Rscript -e 'library(learnPopGen); coalescent.plot(); png("coalescent_plot.png"); coalescent.plot(); dev.off()'
```

## Evaluation signals

- coalescent_plot.png exists in the working directory and is a valid PNG graphic
- The plot depicts allele lineages converging backward in time to a single common ancestor, matching the visual form of Figure 2 of the article
- The simulation used coalescent.plot's default Wright–Fisher coalescent settings (no non-default drift parameters were introduced without justification)
- The genealogy is consistent with coalescence under genetic drift (Kingman, 1982) — lineages merge going backward rather than diverge going forward
- The exported graphic is reproducible from a fresh R session loading learnPopGen from its GitHub repository

## Limitations

- No package version number or changelog is stated for learnPopGen, and no R version requirement is documented, so reproducibility depends on the GitHub state of the source
- Web interface URLs for running functions without R are not provided in the article text
- The skill covers the default coalescent simulation settings; the article does not document custom parameter tuning for coalescent.plot

## Evidence

- [other] coalescent.plot illustrates genetic coalescence (Kingman, 1982) in a Wright–Fisher population of alleles experiencing genetic drift: [evidence span withheld — anchor status: not_found]
- [abstract] it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos: "it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos"
- [readme] This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.": "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
- [discussion] instructors can export plots and animations as flat graphics or animations for use in lecture: "instructors can export plots and animations as flat graphics or animations for use in lecture"
- [abstract] a new R package called learnPopGen that has been designed primarily for the purposes of teaching evolutionary biology, population genetics, and evolutionary theory: "a new R package called learnPopGen that has been designed primarily for the purposes of teaching evolutionary biology, population genetics, and evolutionary theory"
