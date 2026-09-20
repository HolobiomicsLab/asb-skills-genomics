---
name: selection-with-drift-simulation-wrapping
description: Use when a learner-facing interface is needed for the selection.drift
  stochastic simulation and students are 'less adept in the R environment', or when
  arguments of a learnPopGen function must drive automatic re-plotting.
when_to_use_negative:
- The target audience is comfortable in an interactive R session and can call selection.drift
  directly, so a browser-based interface adds no value
- The goal is a static exported graphic or animation (flat graphic, animated graphic,
  or video) for lecture slides rather than a live parameter-driven interface
- The function to expose is deterministic or its web interface cannot meaningfully
  re-render — the article notes web-interface graphics cannot be animated, only reactive
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: learnPopGen
  role: Provides the selection.drift simulation whose arguments are wrapped by the
    Shiny interface
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Computing environment in which the package runs and the Shiny UI/server are
    implemented and run via shiny::runApp
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:687998e5b840df8244c7fea06c16d080dda3e7580fe9ef70b2b73c705c38f256
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# selection-with-drift-simulation-wrapping

## Summary

Build a reactive Shiny web interface around the learnPopGen selection.drift function, exposing its numeric arguments as slider widgets and an action button so that changing parameters automatically recomputes and replots the selection-with-drift simulation for students who cannot use R directly.

## When to use

Use when a learner-facing interface is needed for the selection.drift stochastic simulation and students are 'less adept in the R environment', or when arguments of a learnPopGen function must drive automatic re-plotting.

## When NOT to use

- The target audience is comfortable in an interactive R session and can call selection.drift directly, so a browser-based interface adds no value
- The goal is a static exported graphic or animation (flat graphic, animated graphic, or video) for lecture slides rather than a live parameter-driven interface
- The function to expose is deterministic or its web interface cannot meaningfully re-render — the article notes web-interface graphics cannot be animated, only reactive

## Inputs

- Argument signature of the learnPopGen selection.drift function (numeric arguments governing selection fitnesses, population size, initial allele frequency, and number of generations)
- learnPopGen package installed from github.com/liamrevell/learnPopGen

## Outputs

- Shiny application (ui + server) with sliderInput widgets and an actionButton
- Reactively redrawn selection-with-drift trajectory plot recomputed on argument change or button click

## How to apply

Install R and the learnPopGen package from its GitHub repository (github.com/liamrevell/learnPopGen), then inspect the argument signature of selection.drift to decide which quantities (e.g. initial allele frequency, fitness coefficients, effective population size, number of generations) the interface should expose. Write a Shiny ui definition with sliderInput widgets for each numeric argument of selection.drift, mirroring the interface pattern illustrated in Figure 4 of the article, plus an actionButton that triggers a fresh stochastic run (since drift is stochastic, a new run on button click samples a new trajectory rather than re-plotting the same one). Implement server logic inside a reactive context so the action button invalidates the reactive expression, selection.drift is re-executed on click, and the drift trajectory plot is redrawn whenever any slider value changes. Because drift is a stochastic process, each button click should yield a visibly different trajectory, while slider-only changes should re-plot the current run's output. Verify the app by launching it with shiny::runApp and confirming that moving a slider or clicking the action button recomputes and re-plots the selection-with-drift simulation.

## Related tools

- **learnPopGen** (Provides the selection.drift simulation whose arguments are wrapped by the Shiny interface) — github.com/liamrevell/learnPopGen
- **R** (Computing environment in which the package runs and the Shiny UI/server are implemented and run via shiny::runApp)

## Examples

```
library(shiny); shiny::runApp('selection_drift_app')
```

## Evaluation signals

- App launches locally via shiny::runApp() without errors
- Moving any sliderInput for a numeric selection.drift argument triggers automatic recomputation and re-plotting
- Clicking the actionButton invalidates the reactive context and yields a visibly different stochastic drift trajectory
- Interface exposes widgets corresponding to each numeric argument in selection.drift's argument signature
- Rendered plot matches output produced by calling selection.drift directly with the same argument values

## Limitations

- Web-interface graphics cannot be animated; they can only be reactive (recomputed and replotted when arguments change)
- The article does not provide URLs for the pre-built web interfaces, so a custom app must be built and hosted locally or on a remote server
- No package version number or changelog is provided for learnPopGen, so reproducibility of the interface depends on the GitHub repository state

## Evidence

- [other] Reactive web interface pattern for argument changes: [evidence span withheld — anchor status: not_found]
- [abstract] Motivation: web interfaces for students less adept in R: "instructors may choose to guide students less adept in the R environment to one of various web interfaces that I have built for a number of the functions of the package"
- [discussion] Remote-server architecture of the web interfaces: "I have also built a number of web interfaces for functions of the package which use R on a remote server"
- [readme] Repository is the installation source: "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
