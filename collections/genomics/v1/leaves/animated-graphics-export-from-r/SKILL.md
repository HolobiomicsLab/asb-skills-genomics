---
name: animated-graphics-export-from-r
description: Use when an R simulation returns a sequence of plot frames (e.g. a shifting
  phenotypic distribution under directional selection) that must be exported as an
  animated GIF or video file.
when_to_use_negative:
- The analysis goal is a single static figure (e.g. one equilibrium gene-frequency
  plot) rather than a time sequence of frames.
- The student or instructor cannot use R at all — the article suggests steering such
  users to the package's web interfaces instead of local animation export.
- The trait change depends on new mutation rather than standing genetically based
  variation, which is outside the phenotypic.selection demonstration's design.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3299
tools:
- name: learnPopGen
  role: Provides the simulation functions (e.g. phenotypic.selection) whose plot frames
    are animated, plus the export pathway for writing flat graphics, animated GIFs,
    or videos from R.
  repo: github.com/liamrevell/learnPopGen
- name: R
  role: The scientific computing environment in which the simulation runs, frames
    are rendered in the graphics device, and exports are produced.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.5412
    title: '<i>learnPopGen</i>: An R package for population genetic simulation and
      numerical analysis'
  source_content_hash: sha256:9a54e8a332b5d56a10432d6a804a380e6c5a465bf19980aafce737e1bf9c9732
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - learnPopGen
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# animated-graphics-export-from-r

## Summary

Render sequences of R graphics-device frames from learnPopGen simulations (e.g. phenotypic.selection, drift coalescence) as animated GIFs or videos, so evolutionary dynamics can be inspected frame-by-frame or embedded in teaching materials.

## When to use

Use when an R simulation returns a sequence of plot frames (e.g. a shifting phenotypic distribution under directional selection) that must be exported as an animated GIF or video file.

## When NOT to use

- The analysis goal is a single static figure (e.g. one equilibrium gene-frequency plot) rather than a time sequence of frames.
- The student or instructor cannot use R at all — the article suggests steering such users to the package's web interfaces instead of local animation export.
- The trait change depends on new mutation rather than standing genetically based variation, which is outside the phenotypic.selection demonstration's design.

## Inputs

- Frame sequence returned by a learnPopGen simulation function (e.g. phenotypic.selection over generations of directional selection on a polygenic trait)
- R graphics-device plot frames rendered from the simulation output

## Outputs

- phenotypic_selection.gif — an animated GIF with multiple frames showing the phenotypic distribution shifting beyond its original range
- Optionally flat graphics or video exports of the same plot sequence for lecture use

## How to apply

Install learnPopGen from github.com/liamrevell/learnPopGen and load it in an interactive R session. Call the relevant simulation function (e.g. phenotypic.selection on a simulated polygenic trait distribution), iterating over generations so the phenotypic distribution is recomputed each generation from standing genetic variation only, with no new mutation. Render each returned frame in R's graphics device, then use the package's export pathway to write the frame sequence as an animated GIF (e.g. phenotypic_selection.gif) or video; the paper notes this export is straightforward for both flat and animated graphics. Validate the output by confirming the GIF exists, contains multiple frames, and shows the expected dynamic (e.g. the distribution's mean shifting beyond its original range under directional selection).

## Related tools

- **learnPopGen** (Provides the simulation functions (e.g. phenotypic.selection) whose plot frames are animated, plus the export pathway for writing flat graphics, animated GIFs, or videos from R.) — github.com/liamrevell/learnPopGen
- **R** (The scientific computing environment in which the simulation runs, frames are rendered in the graphics device, and exports are produced.)

## Examples

```
library(learnPopGen); obj <- phenotypic.selection(...); # render frames and export as animated GIF: phenotypic_selection.gif
```

## Evaluation signals

- phenotypic_selection.gif exists at the expected path and is a valid animated GIF containing multiple frames.
- Successive frames show the phenotypic distribution's mean shifting progressively beyond the trait's original range under directional selection (visual QC).
- No new mutation was introduced: each generation's distribution is recomputed from standing genetically based variation only.
- Exported animation opens and plays correctly outside R (e.g. in a browser or image viewer) as a flat animated graphic.

## Limitations

- No package version number, R version requirement, or changelog is stated in the article, so reproducibility of the exact animation behavior across versions is unverified.
- Web interfaces for the package functions run R on a remote server, but their URLs are not provided in the article text; local animation export requires an interactive R session.
- Animation export is intended for teaching/demonstration of evolutionary phenomena, not for quantitative downstream analysis of the frame data.

## Evidence

- [abstract] Package functions support visualizing gene frequency changes, animating phenotypic trait changes under selection, game theory model analysis, and plotting coalescence under genetic drift: "to visualize gene frequency changes through time under multiple deterministic and stochastic processes, to compute and animate the changes in phenotypic trait values or distributions under natural"
- [abstract] Plots and animations can be exported as flat or animated graphics or videos: "it is straightforward to export plots and animations from R in the form of flat or animated graphics, or as videos"
- [discussion] Instructors export plots and animations for lecture use: "instructors can export plots and animations as flat graphics or animations for use in lecture"
- [discussion] Students use the package in an interactive R session: "Students can use the package within an interactive R session"
- [readme] README describes the repository contents: "This repository contains source code, documentation, and R data for the project"
