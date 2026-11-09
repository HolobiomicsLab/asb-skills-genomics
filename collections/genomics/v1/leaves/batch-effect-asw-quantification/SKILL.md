---
name: batch-effect-asw-quantification
description: Use when you have paired reference/query scRNA-seq data (e.g. the 86
  benchmark tests comparing scMAGIC to 13 RBC methods) and need to quantify batch
  effects (F1-ASW) plus cell-type coverage difficulty (CDI) to stratify tests.
when_to_use_negative:
- You need to correct or remove batch effects before classification — this skill only
  quantifies and stratifies difficulty via F1-ASW/CDI; batch-effect elimination is
  handled separately by scMAGIC's second-round RBC using verified query cells as a
  new reference
- You have a single dataset with no paired reference/query structure, since silhouette
  statistics across cell-type versus batch labels are undefined in that setting
- Your goal is raw annotation accuracy alone rather than relating accuracy to classification
  difficulty — a direct accuracy comparison across benchmark tests suffices
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scMAGIC
  role: Method under evaluation; its accuracy-versus-CDI-rank slope (Z-score 2.32)
    is the stability benchmark against the 13 competing RBC methods
  repo: https://github.com/TianLab-Bioinfo/scMAGIC
- name: Seurat
  role: Competing RBC method (Seurat v4) whose slope Z-score (0.85) is compared; also
    provides integration of reference and query data into the same space used for
    silhouette computation
- name: scClassify
  role: Competing tree-based RBC method whose accuracy-versus-CDI-rank slope Z-score
    (1.2) is compared against scMAGIC
- name: scSemiCluster
  role: Competing semi-supervised RBC method whose slope Z-score (0.72) is compared;
    also integrates reference and query data into the same space
  repo: https://github.com/xuebaliang/scSemiCluster
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab1275
    title: 'scMAGIC: accurately annotating single cells using two rounds of reference-based
      classification'
  source_content_hash: sha256:7f99fb78623a8222836916aa43d320821eef5107a8ffdd7ebc1ff1070fed5cfd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TianLab-Bioinfo/scMAGIC
  tools:
  - Seurat
  - scClassify
  - scMAGIC
  - scSemiCluster
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# batch-effect-asw-quantification

## Summary

Quantifies batch effect severity between reference and query scRNA-seq datasets using an F1-ASW score built from average-silhouette-width statistics of cell-type versus batch labels, and combines it with the proportion of classifiable query cells into a classification difficulty index (CDI) for ranking benchmark tests.

## When to use

Use when you have paired reference/query scRNA-seq data (e.g. the 86 benchmark tests comparing scMAGIC to 13 RBC methods) and need to quantify batch effects (F1-ASW) plus cell-type coverage difficulty (CDI) to stratify tests.

## When NOT to use

- You need to correct or remove batch effects before classification — this skill only quantifies and stratifies difficulty via F1-ASW/CDI; batch-effect elimination is handled separately by scMAGIC's second-round RBC using verified query cells as a new reference
- You have a single dataset with no paired reference/query structure, since silhouette statistics across cell-type versus batch labels are undefined in that setting
- Your goal is raw annotation accuracy alone rather than relating accuracy to classification difficulty — a direct accuracy comparison across benchmark tests suffices

## Inputs

- Per-test annotation accuracy results for scMAGIC and the 13 competing RBC methods across the 86 benchmark tests
- Reference/query expression-matrix pairs underlying each benchmark test (e.g. RDS files such as Tasic.Rdata reference and Drop-seq target data)
- Cell-type labels and batch labels for the combined reference-query space of each test
- Per-test proportion of query cells that remain classifiable

## Outputs

- CDI score and CDI rank table for all 86 benchmark tests (written to disk)
- OLS regression summaries (slope, standard error, slope Z-score) per annotation method
- Figure 5B-style plot of per-method slope Z-scores comparing scMAGIC (2.32), Seurat v4 (0.85), scClassify (1.2) and scSemiCluster (0.72)

## How to apply

For each benchmark test, combine the reference and query expression data into a shared space and compute average-silhouette-width statistics using both the cell-type labels and the batch labels; convert these into an F1-ASW batch-effect score. In parallel, compute the proportion of query cells that remain classifiable by the RBC method, and combine the F1-ASW score with this proportion into a single CDI score per test. Rank all 86 tests by CDI, then for each annotation method fit an OLS model of annotation accuracy versus CDI rank with R's lm() and convert each slope to a Z-score (slope divided by its standard error). The rationale is that slope Z-scores reveal how steeply a method's accuracy degrades with classification difficulty — scMAGIC degrades far more slowly (Z-score 2.32) than Seurat v4 (0.85), scClassify (1.2) or scSemiCluster (0.72), consistent with its second-round RBC eliminating reference-query batch effects. Finally, render a Figure 5B-style plot of per-method slope Z-scores and write the CDI table and regression summaries to disk.

## Related tools

- **scMAGIC** (Method under evaluation; its accuracy-versus-CDI-rank slope (Z-score 2.32) is the stability benchmark against the 13 competing RBC methods) — https://github.com/TianLab-Bioinfo/scMAGIC
- **Seurat** (Competing RBC method (Seurat v4) whose slope Z-score (0.85) is compared; also provides integration of reference and query data into the same space used for silhouette computation)
- **scClassify** (Competing tree-based RBC method whose accuracy-versus-CDI-rank slope Z-score (1.2) is compared against scMAGIC)
- **scSemiCluster** (Competing semi-supervised RBC method whose slope Z-score (0.72) is compared; also integrates reference and query data into the same space) — https://github.com/xuebaliang/scSemiCluster

## Examples

```
fit <- lm(accuracy ~ CDI_rank, data = res); slope_z <- coef(summary(fit))["CDI_rank", "Estimate"] / coef(summary(fit))["CDI_rank", "Std. Error"]
```

## Evaluation signals

- Each of the 86 benchmark tests receives a finite CDI score and the tests are fully rank-ordered by CDI with no missing ranks
- The fitted scMAGIC slope Z-score reproduces the reported value of 2.32, with competing methods near Seurat v4 (0.85), scClassify (1.2) and scSemiCluster (0.72)
- F1-ASW batch-effect score is computable for every test from average-silhouette-width statistics of cell-type versus batch labels in the combined reference-query space
- The classifiable-proportion component of CDI lies in [0, 1] for every test
- Each per-method regression summary reports a slope together with its standard error, and the Z-score equals slope divided by standard error

## Limitations

- CDI conflates two components — the F1-ASW batch-effect score and the proportion of classifiable query cells — so high CDI can reflect either strong batch effects or query cell types not covered by the reference
- The slope Z-score comparison is grounded in the 86 real-data benchmark tests prepared for three user situations; results depend on those specific reference/query pairs (e.g. Tasic SMART-seq reference vs Campbell Drop-seq target, MCA reference vs mouse neocortex query)
- Methods that integrate reference and query into the same space (Seurat v4, scSemiCluster) are evaluated under the same CDI ranking, but the article notes no version numbers were stated for these tools

## Evidence

- [abstract] scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests: "scMAGIC significantly outperforms 13 competing RBC methods with their optimal parameter settings across 86 benchmark tests"
- [methods] F1-ASW computed in combined reference-query space: [evidence span withheld — anchor status: not_found]
- [intro] Second-round RBC eliminates batch effects via verified query cells as new reference: "those query cells whose cell identifies are confidently validated in the first-round RBC are used as the new reference to again classify query single cells"
- [discussion] Integration of reference and query into same space underlies competing methods: "integrating the reference and the query data into the same space"
- [readme] scMAGIC is a classification tool using well-annotated RNA-seq references: "is a novel single-cell classification tool using well-annotated bulk or single-cell RNA-seq data as the reference"
