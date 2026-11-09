---
name: reactive-replot-visualization
description: 'Use when wrapping a learnPopGen simulation function (e.g. selection.drift)
  in a Shiny web interface, and the goal is reactive behavior: argument changes trigger
  automatic recomputation and re-plotting of the simulation graphic.'
when_to_use_negative:
- The target graphic needs true animation — the article notes web-interface graphics
  cannot be animated; use the package's in-R animations exported as animated graphics
  or videos instead
- The audience can work directly in an interactive R session with learnPopGen rather
  than needing a browser-based interface
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: learnPopGen
  role: Supplies the simulation function (selection.drift) whose arguments drive the
    reactive interface
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Computing environment in which the Shiny UI/server are written and the app
    is run locally
- name: Shiny
  role: Framework for the reactive UI (sliderInput, actionButton) and server logic
    that invalidates the reactive context and re-executes selection.drift
- name: PopGen.apps
  role: Repository hosting the existing web interfaces built for learnPopGen functions
  repo: liamrevell/PopGen.apps
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:e6302ed03d85cd5e6a483e9b314c503f9a6a48b9961acca092a152f4838effcf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PopGen.apps
  - R
  - Shiny
  - learnPopGen
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# reactive-replot-visualization

## Summary

Build a Shiny web interface around a learnPopGen simulation function (e.g. selection.drift) in which sliderInput widgets expose each numeric argument and an actionButton triggers a new simulation, so changing arguments automatically recomputes and replots the drift trajectory.

## When to use

Use when wrapping a learnPopGen simulation function (e.g. selection.drift) in a Shiny web interface, and the goal is reactive behavior: argument changes trigger automatic recomputation and re-plotting of the simulation graphic.

## When NOT to use

- The target graphic needs true animation — the article notes web-interface graphics cannot be animated; use the package's in-R animations exported as animated graphics or videos instead
- The audience can work directly in an interactive R session with learnPopGen rather than needing a browser-based interface

## Inputs

- Argument signature of a learnPopGen simulation function (e.g. selection.drift) with its numeric parameters
- LearnPopGen package installed from github.com/liamrevell/learnPopGen

## Outputs

- Shiny app (ui + server) with sliderInput widgets and an actionButton
- Reactive drift trajectory plot that updates when arguments change or the action button is clicked

## How to apply

First install R and learnPopGen from its GitHub repository (github.com/liamrevell/learnPopGen) and inspect the argument signature of the target function (e.g. selection.drift) to decide which quantities the interface should expose. Write a Shiny UI definition with a sliderInput widget for each numeric argument of the function, plus an actionButton that triggers a new simulation run, following the interface pattern illustrated in Figure 4 of the article. Implement server logic so that the action button invalidates the reactive context, selection.drift is re-executed on click, and the drift trajectory plot is redrawn reactively whenever argument values change. Verify locally via shiny::runApp that moving a slider or clicking the action button recomputes and re-plots the selection-with-drift simulation. The rationale is that web-interface graphics cannot be animated, but they can be reactive — recomputation happens automatically as arguments change.

## Related tools

- **learnPopGen** (Supplies the simulation function (selection.drift) whose arguments drive the reactive interface) — github.com/liamrevell/learnPopGen
- **R** (Computing environment in which the Shiny UI/server are written and the app is run locally)
- **Shiny** (Framework for the reactive UI (sliderInput, actionButton) and server logic that invalidates the reactive context and re-executes selection.drift)
- **PopGen.apps** (Repository hosting the existing web interfaces built for learnPopGen functions) — liamrevell/PopGen.apps

## Examples

```
Rscript -e "shiny::runApp('selection-drift-app')"
```

## Evaluation signals

- App launches locally with shiny::runApp without errors
- Moving any sliderInput (changing an argument value) automatically recomputes and replots the drift trajectory
- Clicking the actionButton re-executes selection.drift and produces a new simulation run
- Every numeric argument of selection.drift is exposed as a sliderInput, mirroring the interface pattern of Figure 4 of the article

## Limitations

- Web-interface graphics cannot be animated, unlike plots/animations exported from R as flat or animated graphics or videos
- No changelog, package version number, or R version requirement is stated for learnPopGen, so reproducibility of the app environment is not documented
- Web interface URLs are not provided in the article text; existing interfaces run R on a remote server but URLs must be found elsewhere

## Evidence

- [abstract] instructors may choose to guide students less adept in the R environment to one of various web interfaces that I have built for a number of the functions: "instructors may choose to guide students less adept in the R environment to one of various web interfaces that I have built for a number of the functions"
- [full_text] when the arguments of the function change, the analysis is automatically recomputed and replotted: "when the arguments of the function change, the analysis is automatically recomputed and replotted"
- [readme] This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.": "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
