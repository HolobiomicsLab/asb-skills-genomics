---
name: r-package-function-execution
description: Use when the goal is to run a named function of an R package (e.g. learnPopGen's
  coalescent.plot, drift.selection, phenotypic.selection) in an R session to simulate
  an evolutionary process and export the resulting plot or animation as a file.
when_to_use_negative:
- The analysis target is not an evolutionary/population-genetic simulation (e.g. bioinformatics
  sequence processing or general statistical modeling) — the package is scoped to
  population genetics, quantitative genetics, and evolutionary theory.
- The practitioner cannot run R at all and lacks an Internet-connected browser — the
  article notes the package's web interfaces are intended for 'students less adept
  in the R environment', though the article does not provide the URLs.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0091
tools:
- name: learnPopGen
  role: Primary R package supplying the population-genetic simulation functions (coalescent.plot,
    drift.selection, phenotypic.selection, selection.drift) and their plotting/animation
    methods
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Scientific computing environment in which package functions are executed,
    plots are rendered, and exports (flat graphics, animated GIFs, videos) are produced
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:908ebdb8e908da3f6f01a28f06a5e5d291bec78e584b4d4b217ad9b01ef34229
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# r-package-function-execution

## Summary

Executes population-genetic simulation and numerical-analysis functions from the learnPopGen R package in an interactive R session, generating plots, animations, and allele-frequency trajectory objects. Grounded in Revell (2019, Ecology and Evolution), which presents the package for teaching population genetics, quantitative genetics, and evolutionary theory.

## When to use

Use when the goal is to run a named function of an R package (e.g. learnPopGen's coalescent.plot, drift.selection, phenotypic.selection) in an R session to simulate an evolutionary process and export the resulting plot or animation as a file.

## When NOT to use

- The analysis target is not an evolutionary/population-genetic simulation (e.g. bioinformatics sequence processing or general statistical modeling) — the package is scoped to population genetics, quantitative genetics, and evolutionary theory.
- The practitioner cannot run R at all and lacks an Internet-connected browser — the article notes the package's web interfaces are intended for 'students less adept in the R environment', though the article does not provide the URLs.

## Inputs

- R session with learnPopGen installed from github.com/liamrevell/learnPopGen
- Function arguments: initial allele frequency, genotype fitness vector, effective population size, number of generations, number of replicate populations (for drift.selection)
- Default coalescent simulation settings (for coalescent.plot)
- Simulated genetically based polygenic trait distribution (for phenotypic.selection)

## Outputs

- Allele-frequency trajectory objects per replicate (exportable as e.g. drift_selection_trajectories.csv)
- Coalescent genealogy plot (e.g. coalescent_plot.png)
- Selection-plus-drift trajectory plot (e.g. drift_selection_plot.png)
- Animated GIF of phenotypic distribution under directional selection (e.g. phenotypic_selection.gif)
- Shiny web interface with reactive re-plotting of selection.drift trajectories

## How to apply

Install learnPopGen from its source repository (github.com/liamrevell/learnPopGen) and load it within an interactive R session, since the article notes students use the package this way. Inspect the target function's argument signature before calling it (e.g. drift.selection takes initial allele frequency, genotype fitness vector, effective population size, number of generations, and number of replicate populations). Call the function with explicit or default arguments — for instance coalescent.plot with its default Wright–Fisher coalescent settings — letting the function perform Wright–Fisher-style binomial sampling modulated by deterministic selection or drift. Extract returned objects (e.g. generation-by-generation allele-frequency trajectories) and write them to flat files, then render plots/animations via the package's plotting methods. Export graphics using R's standard export pathway as flat graphics, animated GIFs, or videos, as the article states this is straightforward for the package's plots. Validate by checking output files exist, are valid PNG/GIF formats, and depict the expected biological pattern (e.g. allele lineages coalescing to a common ancestor, trajectories fluctuating stochastically around the selection-driven trend with frequencies in [0, 1]).

## Related tools

- **learnPopGen** (Primary R package supplying the population-genetic simulation functions (coalescent.plot, drift.selection, phenotypic.selection, selection.drift) and their plotting/animation methods) — github.com/liamrevell/learnPopGen
- **R** (Scientific computing environment in which package functions are executed, plots are rendered, and exports (flat graphics, animated GIFs, videos) are produced)

## Examples

```
library(learnPopGen); coalescent.plot()
```

## Evaluation signals

- Exported plot file (e.g. coalescent_plot.png) exists and is a valid PNG depicting allele lineages coalescing backward in time to a common ancestor under drift, matching the visual form of Figure 2.
- CSV of drift-selection trajectories contains a frequency row per sampled generation per replicate, with all values in [0, 1].
- Trajectories show stochastic fluctuation around the selection-driven deterministic trend, consistent with Wright–Fisher binomial sampling plus selection.
- Exported GIF (e.g. phenotypic_selection.gif) is a valid animated GIF with multiple frames whose successive frames show the phenotypic distribution's mean shifting beyond its original range under directional selection.
- If a Shiny interface is built, launching via shiny::runApp succeeds and moving a slider or clicking the action button recomputes and re-plots the selection.drift simulation.

## Limitations

- No changelog, package version number, R version requirement, DOI/formal citation within the document, or web-interface URLs are provided, so reproducibility of a specific package state cannot be pinned down from the article alone.
- Web-interface graphics cannot be animated (though they are reactive to argument changes).
- The source code was previously available under the alternative moniker 'PopGen', so older references or installations may use that name.
- phenotypic.selection demonstrates response from standing genetically based variation only, with no new mutation introduced, so it should not be used to infer mutation-limited adaptation.

## Evidence

- [abstract] abstract claim that the package supports coalescence plotting and export: "to visualize gene frequency changes through time under multiple deterministic and stochastic processes, to compute and animate the changes in phenotypic trait values or distributions under natural"
- [abstract] export capability statement: "it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos"
- [discussion] interactive R session usage: "Students can use the package within an interactive R session"
- [readme] repository README description of contents: "This repository contains source code, documentation, and R data for the project"
