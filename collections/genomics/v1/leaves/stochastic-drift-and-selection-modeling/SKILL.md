---
name: stochastic-drift-and-selection-modeling
description: Use when the goal is to simulate or demonstrate allele-frequency trajectories
  at a biallelic locus under both deterministic natural selection and stochastic genetic
  drift, e.g. for teaching population genetics.
when_to_use_negative:
- When only deterministic processes are needed (selection or drift alone) — learnPopGen
  provides separate functions for individual deterministic and stochastic processes
  rather than the combined drift.selection model
- When the locus is multiallelic — drift.selection is defined for a biallelic locus
- When the user cannot run R and only needs a quick demonstration — a browser-based
  web interface for the function may be more appropriate than a local R session
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3056
- http://edamontology.org/topic_0622
tools:
- name: learnPopGen
  role: R package whose drift.selection function simulates joint selection and genetic
    drift at a biallelic locus and provides built-in plotting of the resulting trajectories
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Interactive scientific computing environment in which the learnPopGen package
    is loaded and drift.selection is invoked
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:a78ae4a964e533d36faaebc70150f4ba516b30cd7377a072852c46f08796110f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3056
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# stochastic-drift-and-selection-modeling

## Summary

Simulates the simultaneous effects of natural selection and genetic drift at a biallelic locus using the drift.selection function of the learnPopGen R package, which performs Wright–Fisher-style binomial sampling of allele frequencies each generation modulated by a deterministic selection step defined by genotype fitnesses.

## When to use

Use when the goal is to simulate or demonstrate allele-frequency trajectories at a biallelic locus under both deterministic natural selection and stochastic genetic drift, e.g. for teaching population genetics.

## When NOT to use

- When only deterministic processes are needed (selection or drift alone) — learnPopGen provides separate functions for individual deterministic and stochastic processes rather than the combined drift.selection model
- When the locus is multiallelic — drift.selection is defined for a biallelic locus
- When the user cannot run R and only needs a quick demonstration — a browser-based web interface for the function may be more appropriate than a local R session

## Inputs

- Initial allele frequency (p0) at the biallelic locus
- Genotype fitness vector (w) defining the deterministic selection step
- Effective population size (Ne)
- Number of generations (ngen)
- Number of replicate populations (nrep)

## Outputs

- Allele-frequency trajectory object per replicate (generation-by-generation frequencies)
- drift_selection_trajectories.csv with a frequency row per sampled generation per replicate
- drift_selection_plot.png showing replicate trajectories through time

## How to apply

Install/load learnPopGen from its GitHub source repository (github.com/liamrevell/learnPopGen) in an interactive R session. Call drift.selection, supplying the initial allele frequency (p0), the genotype fitness vector (w), the effective population size (Ne), the number of generations (ngen), and the number of replicate populations (nrep) as arguments. The function performs Wright–Fisher-style binomial sampling of allele frequencies each generation, modulated by the deterministic selection step defined by the genotype fitnesses, and returns an allele-frequency trajectory object for each replicate. Write the generation-by-generation frequencies to a CSV (e.g. drift_selection_trajectories.csv) and plot the replicate trajectories through time with the function's built-in plotting method, saving the figure (e.g. drift_selection_plot.png). For students less adept in R, Revell has also built web interfaces for a number of package functions that run R on a remote server from any browser or smartphone.

## Related tools

- **learnPopGen** (R package whose drift.selection function simulates joint selection and genetic drift at a biallelic locus and provides built-in plotting of the resulting trajectories) — github.com/liamrevell/learnPopGen
- **R** (Interactive scientific computing environment in which the learnPopGen package is loaded and drift.selection is invoked)

## Examples

```
library(learnPopGen); traj <- drift.selection(p0=0.5, w=c(1,0.9,0.8), Ne=100, ngen=200, nrep=10); plot(traj)
```

## Evaluation signals

- CSV validation: the output file contains a frequency row per sampled generation per replicate, and all frequency values lie in [0, 1]
- Trajectories show stochastic fluctuation around the selection-driven deterministic trend
- The returned object provides one allele-frequency trajectory per requested replicate population (nrep)
- Plot rendered via the function's built-in plotting method shows replicate trajectories through time across the requested number of generations

## Limitations

- No changelog found for the package and no package version number is stated
- Web interface URLs are not provided in the text, though browser/smartphone interfaces are said to exist for a number of functions
- No R version requirement is stated for running the package

## Evidence

- [other] drift.selection function, which simulates the simultaneous effects of selection and genetic drift at a biallelic locus: [evidence span withheld — anchor status: not_found]
- [abstract] a new R package called learnPopGen that has been designed primarily for the purposes of teaching evolutionary biology, population genetics: "a new R package called learnPopGen that has been designed primarily for the purposes of teaching evolutionary biology, population genetics"
- [abstract] to visualize gene frequency changes through time under multiple deterministic and stochastic processes: "to visualize gene frequency changes through time under multiple deterministic and stochastic processes"
- [discussion] Students can use the package within an interactive R session: "Students can use the package within an interactive R session"
- [readme] This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.": "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
