---
name: classification-difficulty-index-computation
description: Use when benchmarking single-cell annotation methods across multiple
  reference/query test pairs and you need a principled ranking of test difficulty
  to show which methods remain stable as batch effects grow and fewer query cell types
  are covered by the reference.
when_to_use_negative:
- The analysis goal is a single reference/query pair rather than a panel of benchmark
  tests — CDI ranking is meaningless with one test.
- You need an absolute difficulty score for one method in isolation; the CDI regression
  comparison requires per-test accuracies from multiple methods on identical test
  sets.
- Reference and query share cell types and platform with no batch effects — F1-ASW
  batch-effect scoring and classifiability proportions will be uninformative.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMAGIC
  role: Method under evaluation; its two-round RBC yields the accuracy profile whose
    stability against CDI rank is compared to competitors
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: Seurat
  role: Competing RBC method (Seurat v4 integration-based annotation) whose accuracy-versus-CDI-rank
    slope (Z-score 0.85) is fitted and compared
- name: scClassify
  role: Competing tree-based RBC method whose accuracy-versus-CDI-rank slope (Z-score
    1.2) is fitted and compared
- name: scSemiCluster
  role: Competing semi-supervised RBC method whose accuracy-versus-CDI-rank slope
    (Z-score 0.72) is fitted and compared
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:7150ec3a5ab5793d77bfd257d569c3052e722be4666698aacd47a401b9796a03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - Seurat
  - scClassify
  - scMAGIC
  - scSemiCluster
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# classification-difficulty-index-computation

## Summary

Computes a classification difficulty index (CDI) that ranks reference-based classification (RBC) benchmark tests by difficulty, combining an F1-ASW batch-effect score with the proportion of classifiable query cells, and then quantifies how each method's accuracy degrades with CDI rank via OLS slope Z-scores.

## When to use

Use when benchmarking single-cell annotation methods across multiple reference/query test pairs and you need a principled ranking of test difficulty to show which methods remain stable as batch effects grow and fewer query cell types are covered by the reference.

## When NOT to use

- The analysis goal is a single reference/query pair rather than a panel of benchmark tests — CDI ranking is meaningless with one test.
- You need an absolute difficulty score for one method in isolation; the CDI regression comparison requires per-test accuracies from multiple methods on identical test sets.
- Reference and query share cell types and platform with no batch effects — F1-ASW batch-effect scoring and classifiability proportions will be uninformative.

## Inputs

- Per-test annotation accuracy matrix for scMAGIC and 13 competing RBC methods across 86 benchmark tests
- Combined reference-query expression matrices (R data objects such as .Rdata/RDS, e.g. Tasic.Rdata, MouseNeocortex.Rdata) with cell-type and batch labels per benchmark test
- Per-test counts/proportions of query cells that remain classifiable by each method

## Outputs

- CDI score and CDI rank per benchmark test (86-row CDI table)
- Per-method OLS regression summaries of accuracy versus CDI rank (slope, standard error, slope Z-score)
- Figure 5B-style plot of per-method slope Z-scores

## How to apply

First load the per-test annotation accuracy results for scMAGIC and the 13 competing RBC methods across the 86 benchmark tests, together with the reference/query expression pairs underlying each test. For each test compute the F1-ASW batch-effect score from average-silhouette-width (ASW) statistics of cell-type versus batch labels in the combined reference-query space, and compute the proportion of query cells that remain classifiable; combine these two components into a single CDI score and rank all 86 tests by CDI. Then, for each annotation method, fit an ordinary least-squares model of annotation accuracy versus CDI rank with R's lm(), convert each fitted slope to a Z-score (slope divided by its standard error), and compare against the reported values (scMAGIC 2.32, Seurat v4 0.85, scClassify 1.2, scSemiCluster 0.72). A shallow slope Z-score indicates stable performance under increasing classification difficulty; a high Z-score indicates accuracy that drops steeply as CDI rank increases.

## Related tools

- **scMAGIC** (Method under evaluation; its two-round RBC yields the accuracy profile whose stability against CDI rank is compared to competitors) — https://github.com/TianLab-Bioinfo/scMAGIC
- **Seurat** (Competing RBC method (Seurat v4 integration-based annotation) whose accuracy-versus-CDI-rank slope (Z-score 0.85) is fitted and compared)
- **scClassify** (Competing tree-based RBC method whose accuracy-versus-CDI-rank slope (Z-score 1.2) is fitted and compared)
- **scSemiCluster** (Competing semi-supervised RBC method whose accuracy-versus-CDI-rank slope (Z-score 0.72) is fitted and compared)

## Examples

```
fit <- lm(accuracy ~ CDI_rank, data = cdi_results); s <- coef(summary(fit))["CDI_rank",]; slope_z <- s["Estimate"] / s["Std. Error"]
```

## Evaluation signals

- All 86 benchmark tests receive a defined CDI score and a unique rank; the CDI table written to disk has exactly 86 rows.
- The F1-ASW batch-effect score falls in [0, 1] and the classifiable-query-cell proportion falls in [0, 1] for every test.
- Fitted slope Z-scores are finite and reproducible; reported reference values are scMAGIC 2.32, Seurat v4 0.85, scClassify 1.2, scSemiCluster 0.72.
- Z-score equals slope divided by its standard error from the lm() summary for every method (internal consistency check).
- Regression summaries, the CDI table, and the Figure 5B-style Z-score plot are all rendered and written to disk.

## Limitations

- The CDI ranking is specific to the 86 real-dataset benchmark tests and the reference/query pairs used in the scMAGIC study; it does not generalize to arbitrary new benchmarks without recomputation.
- Tool versions were not pinned in the article (no version numbers given for Seurat, R, sciBet, or scSemiCluster), so reproducing the exact slope Z-scores may be version-sensitive.
- Methods that wrongly assign all unclassifiable query cells (e.g. sciBet and scSemiCluster in some settings) distort the classifiable-cell-proportion component of the CDI.

## Evidence

- [abstract] scMAGIC significantly outperforms 13 competing RBC methods with optimal parameter settings across 86 benchmark tests: "scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests"
- [intro] We have designed three situations that users may encounter when conducting RBC and have prepared a total of 86 benchmark tests using real datasets for these situations.: "We have designed three situations that users may encounter when conducting RBC and have prepared a total of 86 benchmark tests using real datasets for these situations."
- [methods] Implementation of scMAGIC in R: "devtools::install_github("Drizzle-Zhang/scMAGIC")"
- [discussion] Competing methods fail on unclassifiable query cells: "sciBet, a statistical-metrics-based, and scSemiCluster, a semi-supervised-learning-based method, wrongly assign all unclassifiable query cells"
- [readme] scMAGIC is an R package for two-round reference-based classification: [evidence span withheld — anchor status: not_found]
