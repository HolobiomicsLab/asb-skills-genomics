---
name: interactive-slider-action-button-ui-design
description: Use when building a browser-based interface for an R simulation function
  (e.g., selection.drift) so that changing argument values via sliders automatically
  recomputes and re-plots the analysis, per the learnPopGen web-interface pattern.
when_to_use_negative:
- The goal is a static figure or exported media (flat graphic, animation, or video)
  for lecture use — learnPopGen supports direct export from R without a web interface
- The audience is comfortable in an interactive R session — the article positions
  web interfaces for 'students less adept in the R environment'
- The target function has no numeric parameters that a user would vary, so sliders
  and a re-run action button would add no interactivity
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: learnPopGen
  role: Source of the population-genetics simulation functions (e.g., selection.drift)
    wrapped by the Shiny interface; installed from its GitHub repository
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: Computing environment in which the Shiny UI/server logic is written and the
    reactive plotting is implemented
- name: Shiny
  role: Framework supplying sliderInput widgets, the actionButton that invalidates
    the reactive context, and reactive re-plotting; launched locally via shiny::runApp
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:7deeb2b284e7372aabb1003c162b1dee368a362a023ef9c286f22a93bbab6591
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Shiny
  - learnPopGen
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# interactive-slider-action-button-ui-design

## Summary

Design Shiny web interfaces for learnPopGen population-genetics functions in which sliderInput widgets expose each numeric function argument and an actionButton triggers a fresh simulation run, with plots redrawn reactively when arguments change — mirroring the interface pattern the article illustrates (e.g., Figure 4) for selection.drift.

## When to use

Use when building a browser-based interface for an R simulation function (e.g., selection.drift) so that changing argument values via sliders automatically recomputes and re-plots the analysis, per the learnPopGen web-interface pattern.

## When NOT to use

- The goal is a static figure or exported media (flat graphic, animation, or video) for lecture use — learnPopGen supports direct export from R without a web interface
- The audience is comfortable in an interactive R session — the article positions web interfaces for 'students less adept in the R environment'
- The target function has no numeric parameters that a user would vary, so sliders and a re-run action button would add no interactivity

## Inputs

- R function from learnPopGen with numeric arguments to expose (e.g., selection.drift for a selection-with-drift simulation)
- Argument signature of the target function (names, ranges, defaults for sliderInput widgets)
- Shiny UI/server R script files (ui.R / server.R or app.R)

## Outputs

- Shiny app definition (fluidPage UI with sliderInput per numeric argument plus actionButton)
- Interactive reactive plot of the function output (e.g., drift trajectory of allele frequencies), re-plotted on argument change or button click
- Locally launchable app verified via shiny::runApp

## How to apply

Install R and the learnPopGen package from its GitHub repository (github.com/liamrevell/learnPopGen), then inspect the argument signature of the target function (e.g., selection.drift) to decide which quantities the interface should expose. Write a Shiny UI definition — a fluidPage with one sliderInput widget per numeric argument and an actionButton that triggers a new simulation run, following the interface pattern illustrated in Figure 4 of the article. In the server logic, make the action button invalidate the reactive context so that selection.drift is re-executed on click (drawing a new stochastic drift trajectory), and wire the plot output so it redraws reactively whenever argument values change. Note the design trade-off stated in the article: web-interface graphics cannot be animated, but they are frequently 'reactive' — when arguments change, the analysis is automatically recomputed and replotted. Verify by launching locally with shiny::runApp and confirming that moving a slider or clicking the action button recomputes and re-plots the selection-with-drift simulation.

## Related tools

- **learnPopGen** (Source of the population-genetics simulation functions (e.g., selection.drift) wrapped by the Shiny interface; installed from its GitHub repository) — github.com/liamrevell/learnPopGen
- **R** (Computing environment in which the Shiny UI/server logic is written and the reactive plotting is implemented)
- **Shiny** (Framework supplying sliderInput widgets, the actionButton that invalidates the reactive context, and reactive re-plotting; launched locally via shiny::runApp)

## Examples

```
ui <- fluidPage(sliderInput("p0","p0",min=0,max=1,value=0.5), actionButton("run","Simulate")); shiny::runApp()
```

## Evaluation signals

- The app launches locally (e.g., via shiny::runApp) without errors
- Moving any slider widget triggers automatic recomputation and re-plotting of the simulation output
- Clicking the actionButton invalidates the reactive context and re-executes selection.drift, producing a new stochastic simulation run (a visibly different trajectory)
- The UI exposes a slider for every numeric argument of the target function, matching its inspected argument signature
- Interface layout and behavior follow the article's illustrated pattern (Figure 4): sliders for numeric arguments plus a run button

## Limitations

- Web-interface graphics cannot be animated, unlike the package's in-R animations — only reactive re-plotting is possible
- The article's web interfaces run R on a remote server and are used from an Internet-connected browser or smartphone; specific web-interface URLs are not provided in the text
- No changelog or package version number is stated for learnPopGen, so interface code may drift from the current GitHub source

## Evidence

- [results] Although web-interface graphics cannot be animated, they are frequently 'reactive': when the arguments of the function change, the analysis is automatically recomputed and replotted.: [evidence span withheld — anchor status: not_found]
- [abstract] instructors may choose to guide students less adept in the R environment to one of various web interfaces that I have built for a number of the functions of the package: "instructors may choose to guide students less adept in the R environment to one of various web interfaces that I have built for a number of the functions of the package"
- [discussion] I have also built a number of web interfaces for functions of the package which use R on a remote server: "I have also built a number of web interfaces for functions of the package which use R on a remote server"
- [readme] This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.": "This repository contains source code, documentation, and R data for the project "learnPopGen: Population Genetic Simulations & Numerical Analysis.""
