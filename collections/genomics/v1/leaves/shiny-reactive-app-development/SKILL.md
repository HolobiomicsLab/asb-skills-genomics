---
name: shiny-reactive-app-development
description: Use when wrapping an R simulation or analysis function (e.g. a population-genetic
  routine from learnPopGen) in a browser-based interface so that students can explore
  how changing argument values reactively recomputes and replots the result without
  knowing R.
when_to_use_negative:
- The goal is an animated graphic (e.g. an exported video or animation of trait change
  under selection) — web-interface graphics cannot be animated; export animations
  from R instead
- The target audience is comfortable in an interactive R session and can call the
  package functions directly, making a browser interface unnecessary
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: learnPopGen
  role: Package supplying the analysis function (selection.drift) whose arguments
    drive the reactive Shiny interface
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Computing environment in which the Shiny app, the package, and the reactive
    re-plotting logic are implemented and run
- name: PopGen.apps
  role: Repository of the existing Shiny web interfaces for learnPopGen functions,
    serving as the interface-pattern reference
  repo: liamrevell/PopGen.apps
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:ab1fe94cafb0b668daca056f5c005abf160cf7d4d076c22bb8962c3aae3a99bd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PopGen.apps
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# shiny-reactive-app-development

## Summary

Build a Shiny web interface for an R analysis function (here, learnPopGen's selection.drift) in which slider inputs for each numeric argument trigger automatic recomputation and re-plotting of the analysis output.

## When to use

Use when wrapping an R simulation or analysis function (e.g. a population-genetic routine from learnPopGen) in a browser-based interface so that students can explore how changing argument values reactively recomputes and replots the result without knowing R.

## When NOT to use

- The goal is an animated graphic (e.g. an exported video or animation of trait change under selection) — web-interface graphics cannot be animated; export animations from R instead
- The target audience is comfortable in an interactive R session and can call the package functions directly, making a browser interface unnecessary

## Inputs

- R function object whose argument signature defines the UI controls (e.g. learnPopGen::selection.drift with its numeric arguments)
- R package source installed from github.com/liamrevell/learnPopGen

## Outputs

- Shiny app (ui + server definition) exposing sliderInput widgets and an actionButton for the target function
- Reactively re-plotted simulation graphic (e.g. selection-with-drift allele-frequency trajectory) rendered in the browser

## How to apply

Inspect the argument signature of the target function (e.g. selection.drift from learnPopGen, installed from github.com/liamrevell/learnPopGen) to decide which quantities the interface should expose. Write a Shiny UI definition with a sliderInput widget for each numeric argument plus an actionButton that triggers a new simulation run, following the interface pattern illustrated in Figure 4 of the learnPopGen article. In the server logic, let the action button invalidate the reactive context so selection.drift is re-executed on click, and redraw the drift trajectory plot reactively whenever any argument value changes. Verify locally with shiny::runApp that moving a slider or clicking the action button recomputes and re-plots the selection-with-drift simulation; note that web-interface graphics cannot be animated, but they can be reactive.

## Related tools

- **learnPopGen** (Package supplying the analysis function (selection.drift) whose arguments drive the reactive Shiny interface) — github.com/liamrevell/learnPopGen
- **R** (Computing environment in which the Shiny app, the package, and the reactive re-plotting logic are implemented and run)
- **PopGen.apps** (Repository of the existing Shiny web interfaces for learnPopGen functions, serving as the interface-pattern reference) — liamrevell/PopGen.apps

## Examples

```
library(learnPopGen); args(selection.drift); shiny::runApp("selection_drift_app")
```

## Evaluation signals

- shiny::runApp launches the app locally without errors
- Moving a slider corresponding to a selection.drift argument causes the plot to be recomputed and redrawn reactively
- Clicking the actionButton invalidates the reactive context and starts a new simulation run with a redrawn drift trajectory
- Every numeric argument of selection.drift is exposed as a sliderInput widget in the UI
- The interface follows the pattern illustrated in Figure 4 of the article (sliders + action button rather than code entry)

## Limitations

- Web-interface graphics cannot be animated, only reactively re-plotted
- The article does not provide the web interface URLs, and no changelog or version number is documented for learnPopGen
- Interfaces are aimed at students less adept in R; they run R on a remote server and require an Internet-connected browser or smartphone

## Evidence

- [abstract] Web interfaces available for students less adept in R: "instructors may choose to guide students less adept in the R environment to one of various web interfaces"
- [discussion] Web interfaces run R on a remote server: "I have also built a number of web interfaces for functions of the package which use R on a remote server but can be run from any Internet‐connected browser or smartphone."
- [readme] learnPopGen README: "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
