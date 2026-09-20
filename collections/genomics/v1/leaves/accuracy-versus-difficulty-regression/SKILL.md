---
name: accuracy-versus-difficulty-regression
description: Use when you have per-test annotation accuracies for multiple RBC methods
  across many benchmark tests and need to compare performance stability as classification
  difficulty (batch effects, incomplete reference coverage) increases.
when_to_use_negative:
- When only a single benchmark test or a handful of tests exists — slope estimation
  over CDI rank requires a ranked series of many tests
- When the goal is overall accuracy comparison rather than stability of accuracy under
  increasing classification difficulty
- When query/reference pairs lack batch labels or cell-type labels needed to compute
  the F1-ASW batch-effect component of the CDI
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: scMAGIC
  role: Method under evaluation whose accuracy-versus-CDI-rank slope Z-score (2.32)
    is compared against competitors
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: Seurat
  role: Competing RBC method (Seurat v4 integration-based, slope Z-score 0.85) included
    in the regression comparison
- name: scClassify
  role: Competing tree-based RBC method (slope Z-score 1.2) included in the regression
    comparison
- name: scSemiCluster
  role: Competing semi-supervised RBC method (slope Z-score 0.72) included in the
    regression comparison
- name: SciBet
  role: Competing statistical-metric-based RBC method among the 13 methods evaluated
    across the 86 benchmark tests
  repo: https://github.com/PaulingLiu/scibet
- name: SingleR
  role: Competing statistical-metric-based RBC method among the 13 methods evaluated
    across the 86 benchmark tests
  repo: https://github.com/dviraran/SingleR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:5d03279cf0bb510a1da3332ad3fc5bba6da2edbd3f96519f1652a6e8307ca738
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - SciBet
  - Seurat
  - SingleR
  - scClassify
  - scMAGIC
  - scSemiCluster
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# accuracy-versus-difficulty-regression

## Summary

Quantifies how a cell-annotation method's classification accuracy degrades with increasing classification difficulty by regressing per-benchmark-test accuracy on CDI rank with R's lm() and comparing slope Z-scores across methods (e.g. scMAGIC 2.32 vs Seurat v4 0.85, scClassify 1.2, scSemiCluster 0.72 over 86 benchmark tests).

## When to use

Use when you have per-test annotation accuracies for multiple RBC methods across many benchmark tests and need to compare performance stability as classification difficulty (batch effects, incomplete reference coverage) increases.

## When NOT to use

- When only a single benchmark test or a handful of tests exists — slope estimation over CDI rank requires a ranked series of many tests
- When the goal is overall accuracy comparison rather than stability of accuracy under increasing classification difficulty
- When query/reference pairs lack batch labels or cell-type labels needed to compute the F1-ASW batch-effect component of the CDI

## Inputs

- Per-test annotation accuracy results for scMAGIC and the 13 competing RBC methods across the 86 benchmark tests
- Reference/query expression pairs underlying each benchmark test (e.g. Tasic SMART-seq reference, Campbell Drop-seq query, MCA reference / MouseNeocortex query)
- Cell-type and batch label assignments for reference and query cells (for silhouette-width computation)

## Outputs

- CDI table ranking all 86 benchmark tests by classification difficulty
- Per-method OLS regression summaries (slope, standard error, Z-score) from lm()
- Figure 5B-style plot of per-method slope Z-scores written to disk

## How to apply

First compute a Classification Difficulty Index (CDI) per benchmark test by combining the F1-ASW batch-effect score (average silhouette width of cell-type versus batch labels in the combined reference-query space) with the proportion of query cells that remain classifiable, then rank all 86 tests by CDI. For each annotation method, fit an ordinary least-squares linear model of annotation accuracy versus CDI rank using R's lm(). Convert each fitted slope to a Z-score (slope divided by its standard error) and compare against reported values (scMAGIC 2.32, Seurat v4 0.85, scClassify 1.2, scSemiCluster 0.72): a smaller slope magnitude with a smaller Z-score indicates accuracy that is more stable under increasing difficulty. Render a Figure 5B-style plot of per-method slope Z-scores and write the CDI table, regression summaries, and plot to disk.

## Related tools

- **scMAGIC** (Method under evaluation whose accuracy-versus-CDI-rank slope Z-score (2.32) is compared against competitors) — https://github.com/TianLab-Bioinfo/scMAGIC
- **Seurat** (Competing RBC method (Seurat v4 integration-based, slope Z-score 0.85) included in the regression comparison)
- **scClassify** (Competing tree-based RBC method (slope Z-score 1.2) included in the regression comparison)
- **scSemiCluster** (Competing semi-supervised RBC method (slope Z-score 0.72) included in the regression comparison)
- **SciBet** (Competing statistical-metric-based RBC method among the 13 methods evaluated across the 86 benchmark tests) — https://github.com/PaulingLiu/scibet
- **SingleR** (Competing statistical-metric-based RBC method among the 13 methods evaluated across the 86 benchmark tests) — https://github.com/dviraran/SingleR

## Examples

```
fit <- lm(accuracy ~ cdi_rank, data = per_test_results); slope_z <- coef(summary(fit))['cdi_rank', 'Estimate'] / coef(summary(fit))['cdi_rank', 'Std. Error']
```

## Evaluation signals

- The regression slope Z-score for scMAGIC should match the reported value of 2.32, with Seurat v4 at 0.85, scClassify at 1.2, and scSemiCluster at 0.72
- All 86 benchmark tests must be ranked by a single combined CDI score before fitting the lm() models
- Each fitted slope divided by its standard error must yield the reported Z-score for every method
- The rendered plot should be a Figure 5B-style bar/point display of per-method slope Z-scores
- The CDI table, regression summaries, and plot files should all be present on disk after execution

## Limitations

- The CDI depends on silhouette-width statistics of cell-type versus batch labels; tests lacking batch labels cannot yield an F1-ASW component
- Slope Z-scores reflect relative stability over the 86 specific benchmark tests; absolute Z-score magnitudes are not directly transferable to other benchmark panels
- Accuracy still decreases for scMAGIC as CDI rank increases — the method is more stable than competitors but not difficulty-invariant
- Version numbers for R, Seurat, and other tools are not stated in the article, complicating exact reproduction

## Evidence

- [results] scMAGIC's accuracy decreases only slightly as CDI rank increases, with slope Z-score 2.32 versus 0.85 (Seurat v4), 1.2 (scClassify) and 0.72 (scSemiCluster): [evidence span withheld — anchor status: not_found]
- [methods] For each annotation method, fit an ordinary least-squares linear model of annotation accuracy versus CDI rank using R's lm(): [evidence span withheld — anchor status: not_found]
- [methods] compute the F1-ASW batch-effect score from average-silhouette-width statistics of cell-type versus batch labels: [evidence span withheld — anchor status: not_found]
- [abstract] scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests: "scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests"
