---
name: publication-figure-reproduction
description: Use when a source card supplies a per-figure workflow (steps, tools,
  expected numeric results) and the goal is to regenerate that figure's numbers and
  plot, e.g. reproducing slope Z-scores of accuracy-vs-CDI-rank regressions for scMAGIC,
  Seurat v4, scClassify and scSemiCluster.
when_to_use_negative:
- The goal is to reproduce figures requiring data or accession-level detail the paper
  defers to Materials and Methods and does not provide in the analysed section
- Only qualitative performance claims are needed (e.g. 'scMAGIC outperforms competitors')
  without re-deriving the quantitative regression statistics
- The figure depends on tools whose versions are unstated in the source (e.g. Seurat,
  ggplot2, sciBet), making exact bit-level reproduction unverifiable
edam_operation: http://edamontology.org/operation_0370
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMAGIC
  role: Annotated-cell-type classifier whose per-test accuracies across the 86 benchmark
    tests are regressed against CDI rank; installed from GitHub in R
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: Seurat
  role: Competing RBC method (Seurat v4) whose accuracy-vs-CDI-rank slope Z-score
    (0.85) is reproduced for the Figure 5B comparison
- name: scClassify
  role: Tree-based competing RBC method whose slope Z-score (1.2) is reproduced in
    the regression comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:a0c79450c53966f52808cc42a3497d311af11623e7215dab5254206382d0cc35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - Seurat
  - scClassify
  - scMAGIC
  edam_operation: http://edamontology.org/operation_0370
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# publication-figure-reproduction

## Summary

Reproduce a published figure by re-deriving its underlying quantitative analysis from the paper's methods — here, the Figure 5B regression of annotation accuracy on classification difficulty index (CDI) rank across scMAGIC's 86 benchmark tests versus 13 competing RBC methods.

## When to use

Use when a source card supplies a per-figure workflow (steps, tools, expected numeric results) and the goal is to regenerate that figure's numbers and plot, e.g. reproducing slope Z-scores of accuracy-vs-CDI-rank regressions for scMAGIC, Seurat v4, scClassify and scSemiCluster.

## When NOT to use

- The goal is to reproduce figures requiring data or accession-level detail the paper defers to Materials and Methods and does not provide in the analysed section
- Only qualitative performance claims are needed (e.g. 'scMAGIC outperforms competitors') without re-deriving the quantitative regression statistics
- The figure depends on tools whose versions are unstated in the source (e.g. Seurat, ggplot2, sciBet), making exact bit-level reproduction unverifiable

## Inputs

- Per-test annotation accuracy table for scMAGIC and 13 competing RBC methods across 86 benchmark tests
- Reference/query scRNA-seq expression pairs underlying each of the 86 benchmark tests
- Cell-type and batch label assignments per cell for silhouette-width computation

## Outputs

- Ranked CDI table (86 tests, each with F1-ASW batch score, classifiable-cell proportion, composite CDI score and rank)
- OLS regression summaries per method (slope, standard error, slope Z-score)
- Figure 5B-style plot of per-method slope Z-scores

## How to apply

Load the per-test annotation accuracies for scMAGIC and the 13 competing RBC methods across the 86 benchmark tests together with the reference/query expression pairs. Recompute each test's CDI from the F1-ASW batch-effect score (average-silhouette-width statistics of cell-type versus batch labels in the combined reference-query space) and the proportion of classifiable query cells, then rank all 86 tests by CDI. For each method, fit an ordinary least-squares model of annotation accuracy versus CDI rank with R's lm(), and convert each slope to a Z-score (slope divided by its standard error). Judge correctness by comparing fitted Z-scores to the reported values: scMAGIC 2.32, Seurat v4 0.85, scClassify 1.2, scSemiCluster 0.72. Finally render a Figure 5B-style plot of per-method slope Z-scores and write the CDI table and regression summaries to disk.

## Related tools

- **scMAGIC** (Annotated-cell-type classifier whose per-test accuracies across the 86 benchmark tests are regressed against CDI rank; installed from GitHub in R) — https://github.com/TianLab-Bioinfo/scMAGIC
- **Seurat** (Competing RBC method (Seurat v4) whose accuracy-vs-CDI-rank slope Z-score (0.85) is reproduced for the Figure 5B comparison)
- **scClassify** (Tree-based competing RBC method whose slope Z-score (1.2) is reproduced in the regression comparison)

## Examples

```
fit <- lm(accuracy ~ cdi_rank, data = cdi_table); z <- summary(fit)$coefficients[2,1]/summary(fit)$coefficients[2,2]
```

## Evaluation signals

- Fitted slope Z-score for scMAGIC ≈ 2.32, Seurat v4 ≈ 0.85, scClassify ≈ 1.2, scSemiCluster ≈ 0.72
- CDI table contains exactly 86 tests, each with an F1-ASW batch-effect component and a classifiable-cell proportion, fully ranked with no ties or missing ranks
- Regression summaries report slope, standard error and Z = slope/SE for every one of the compared methods
- Rendered plot reproduces the Figure 5B layout: one slope Z-score bar/point per method, ordered as in the publication
- All outputs (CDI table, regression summaries, plot) are written to disk with non-empty, parseable contents

## Limitations

- Dataset accession codes and detailed dataset information are deferred to Materials and Methods, so the reproduction depends on obtaining those inputs separately
- No version numbers are stated for Seurat, R, sciBet, scSemiCluster, ggplot2 or devtools, so exact numeric reproduction may drift across software versions
- No archived code release, version tag or Zenodo DOI is provided for scMAGIC beyond the GitHub repository link, pinning the exact code state is not possible

## Evidence

- [full_text] CDI regression workflow: [evidence span withheld — anchor status: not_found]
- [full_text] OLS fit and Z-score: [evidence span withheld — anchor status: not_found]
- [abstract] Benchmark scope: "scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests"
- [methods] Reference dataset used in benchmark tests: "The reference dataset includes 1727 mouse primary visual cortex cells profiled by SMART-seq."
- [full_text] Reported slope Z-scores: [evidence span withheld — anchor status: not_found]
- [readme] Tool installation: "devtools::install_github("Drizzle-Zhang/scMAGIC")"
