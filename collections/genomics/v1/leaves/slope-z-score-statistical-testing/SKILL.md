---
name: slope-z-score-statistical-testing
description: Use when you have per-test accuracy scores for several methods over a
  ranked difficulty axis and need a single normalized statistic comparing how stable
  each method's performance is as difficulty increases.
when_to_use_negative:
- You need an absolute performance ranking of methods rather than a stability-under-difficulty
  comparison — average accuracy or win-rate metrics are more appropriate
- The difficulty axis is not on a comparable scale across tests (CDI must be computed
  and ranked consistently, otherwise slope Z-scores are not comparable between methods)
- Only a handful of tests are available — the slope standard error will be too large
  for a meaningful Z-score
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: scMAGIC
  role: Method under evaluation whose accuracy-versus-CDI-rank slope Z-score (2.32)
    is compared against competitors
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: Seurat
  role: Competing RBC method (Seurat v4 integration-based) whose slope Z-score (0.85)
    is estimated in the regression
- name: scClassify
  role: Competing tree-based RBC method whose slope Z-score (1.2) is estimated in
    the regression
- name: R (lm function)
  role: Used to fit the ordinary least-squares model of annotation accuracy versus
    CDI rank and obtain slope standard errors
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:9e5189fedf6f8d00d0f354adcd8144f7de2fc59f321981eeef66ad5b2ff70bb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - R (lm function)
  - Seurat
  - scClassify
  - scMAGIC
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# slope-z-score-statistical-testing

## Summary

Fits an OLS regression of annotation accuracy versus classification difficulty index (CDI) rank with R's lm(), then converts the fitted slope to a Z-score (slope / standard error) to quantify how steeply each method's accuracy degrades with increasing classification difficulty across the 86 benchmark tests.

## When to use

Use when you have per-test accuracy scores for several methods over a ranked difficulty axis and need a single normalized statistic comparing how stable each method's performance is as difficulty increases.

## When NOT to use

- You need an absolute performance ranking of methods rather than a stability-under-difficulty comparison — average accuracy or win-rate metrics are more appropriate
- The difficulty axis is not on a comparable scale across tests (CDI must be computed and ranked consistently, otherwise slope Z-scores are not comparable between methods)
- Only a handful of tests are available — the slope standard error will be too large for a meaningful Z-score

## Inputs

- Per-test annotation accuracy results for scMAGIC and the 13 competing RBC methods across the 86 benchmark tests
- Reference/query expression pairs underlying each benchmark test (used to compute the CDI components: F1-ASW batch-effect score and proportion of classifiable query cells)
- CDI ranks for all 86 tests (computed from F1-ASW and classifiable-cell proportion)

## Outputs

- Per-method OLS regression summaries (slope, standard error, slope Z-score) from lm()
- Figure 5B-style plot of per-method slope Z-scores
- CDI table, regression summaries, and plot written to disk

## How to apply

First rank the benchmark tests by a composite difficulty score — here the CDI, combining the F1-ASW batch-effect score (average silhouette width of cell-type versus batch labels in the combined reference-query space) with the proportion of classifiable query cells. For each annotation method, fit an ordinary least-squares linear model of per-test annotation accuracy versus CDI rank using R's lm(). Convert each fitted slope to a Z-score by dividing by its standard error; larger Z-scores indicate steeper accuracy loss and thus less stable performance. In the scMAGIC benchmark, scMAGIC's slope Z-score was 2.32 versus 0.85 (Seurat v4), 1.2 (scClassify) and 0.72 (scSemiCluster), and a low Z-score was interpreted as evidence of stable performance — verify directionality of interpretation before reusing. Render a per-method slope Z-score bar plot (Figure 5B style) and write the CDI table and regression summaries to disk.

## Related tools

- **scMAGIC** (Method under evaluation whose accuracy-versus-CDI-rank slope Z-score (2.32) is compared against competitors) — https://github.com/TianLab-Bioinfo/scMAGIC
- **Seurat** (Competing RBC method (Seurat v4 integration-based) whose slope Z-score (0.85) is estimated in the regression)
- **scClassify** (Competing tree-based RBC method whose slope Z-score (1.2) is estimated in the regression)
- **R (lm function)** (Used to fit the ordinary least-squares model of annotation accuracy versus CDI rank and obtain slope standard errors)

## Examples

```
fit <- lm(accuracy ~ cdi_rank, data = res_scMAGIC); z <- coef(summary(fit))['cdi_rank','Estimate'] / coef(summary(fit))['cdi_rank','Std. Error']
```

## Evaluation signals

- The fitted slope Z-scores reproduce the reported values: scMAGIC 2.32, Seurat v4 0.85, scClassify 1.2, scSemiCluster 0.72
- All 86 benchmark tests are ranked by CDI before regression; the CDI table on disk contains exactly 86 ranked rows
- Each regression summary reports a slope with a finite standard error so the Z-score (slope / SE) is well defined
- The interpretation invariant holds: a method whose accuracy decreases only slightly with CDI rank yields a small slope Z-score relative to less stable methods
- Figure 5B-style output contains one slope Z-score per annotation method covering scMAGIC and the 13 competing RBC methods

## Limitations

- The CDI is composite (F1-ASW batch-effect score plus proportion of classifiable query cells), so slope Z-scores are conditional on how the difficulty index is defined and cannot be compared to benchmarks using other difficulty measures
- The analysis presumes the 86 benchmark tests span the difficulty range of interest; methods' relative stability may differ when query cell types are not fully covered by the reference or when batch effects are structured differently
- No version numbers were stated for R, Seurat, or ggplot2 in this section, so exact reproducibility of the regression requires additional environment details

## Evidence

- [results] slope Z-score: [evidence span withheld — anchor status: not_found]
- [methods] regression model: [evidence span withheld — anchor status: not_found]
- [methods] CDI computation: [evidence span withheld — anchor status: not_found]
- [abstract] benchmark scope: "scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests"
