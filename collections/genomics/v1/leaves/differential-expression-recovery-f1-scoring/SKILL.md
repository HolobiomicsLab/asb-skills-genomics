---
name: differential-expression-recovery-f1-scoring
description: Use when a normalization method must be quantitatively judged on simulated
  scRNA-seq data where the ground-truth normalized values and planted DE/cluster labels
  are known, and RMSE/F1 tables (3 settings x 5 scenarios) are needed.
when_to_use_negative:
- No simulated ground truth or planted DE/cluster labels exist (e.g. real scRNA-seq
  data), so RMSE against truth and F1 for planted-structure recovery cannot be computed
- The goal is downstream biological interpretation of a single dataset rather than
  benchmarking normalization methods against baselines
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3308
tools:
- name: scKWARN
  role: Method under test; its LocASN normalization output (Result$NormalizedData)
    is scored by RMSE and F1
  repo: https://github.com/cyhsuTN/scKWARN
- name: R
  role: Environment in which simulations, normalization, and RMSE/F1 aggregation are
    run
- name: scran
  role: Baseline normalization method (relative counts) whose RMSE/F1 scores are compared
- name: SCnorm
  role: Baseline normalization method scored against the simulated truth
- name: sctransform
  role: Baseline normalization method scored against the simulated truth
- name: PsiNorm
  role: Baseline normalization method scored against the simulated truth
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae008
    title: 'scKWARN: Kernel-weighted-average robust normalization for single-cell
      RNA-seq data'
  source_content_hash: sha256:af6f6c36f802699e115247856b2cbaefaaef4af9161e00a1cf58c076eca6082f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cyhsuTN/scKWARN
  tools:
  - PsiNorm
  - R
  - Rcpp
  - SCnorm
  - devtools
  - methods
  - scKWARN
  - scran
  - sctransform
  - stats
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# differential-expression-recovery-f1-scoring

## Summary

Benchmark scoring procedure for scKWARN evaluation: after normalizing simulated single-cell RNA-seq count matrices (3000 genes, three subpopulations) with scKWARN and baselines (RC, scran, SCnorm, sctransform, PsiNorm), compute RMSE of normalized values against simulated truth and F1 scores for recovery of the planted differential-expression/cluster structure.

## When to use

Use when a normalization method must be quantitatively judged on simulated scRNA-seq data where the ground-truth normalized values and planted DE/cluster labels are known, and RMSE/F1 tables (3 settings x 5 scenarios) are needed.

## When NOT to use

- No simulated ground truth or planted DE/cluster labels exist (e.g. real scRNA-seq data), so RMSE against truth and F1 for planted-structure recovery cannot be computed
- The goal is downstream biological interpretation of a single dataset rather than benchmarking normalization methods against baselines

## Inputs

- Simulated count matrices (3000 genes, three subpopulations) generated from negative binomial distributions with known ground-truth normalized values and planted DE/cluster labels, 100 replicates per scenario
- Normalized expression matrices from each method: scKWARN (LocASN Result$NormalizedData), scran relative counts, SCnorm, sctransform, and PsiNorm outputs

## Outputs

- Per-setting, per-scenario RMSE of normalized values against the simulated truth
- Per-setting, per-scenario F1 scores for recovery of the planted differential-expression/cluster structure
- rmse_table.csv and f1_table.csv, each with 15 rows (3 settings x 5 scenarios)

## How to apply

Generate simulated count matrices per the Supplementary protocol (gene means drawn from a gamma prior, counts from rnbinom, e.g. mu <- rgamma(G, shape = 2, rate = 2); rnbinom(G, size = 0.1, mu = mu)) across five scenarios varying library size, RNA composition, dropout, population balance, and DE level, with 100 replicates per scenario, in three settings (normal error with linear count-depth, negative binomial with linear count-depth, negative binomial without count-depth assumption). Normalize each replicate with scKWARN via LocASN (extracting Result$NormalizedData and Result$scalingFactor) and with scran (relative counts), SCnorm, sctransform, and PsiNorm. For each replicate compute the RMSE of normalized values against the simulated truth and the F1 score for recovering the planted differential-expression/cluster structure (log1p(Result$NormalizedData) feeds clustering). Aggregate metrics into two tables of 15 rows each (3 settings x 5 scenarios) and write rmse_table.csv and f1_table.csv. Correct application is judged by whether the method attains the lowest RMSE and highest F1 across all five scenarios in each setting, as scKWARN did against all five baselines.

## Related tools

- **scKWARN** (Method under test; its LocASN normalization output (Result$NormalizedData) is scored by RMSE and F1) — https://github.com/cyhsuTN/scKWARN
- **R** (Environment in which simulations, normalization, and RMSE/F1 aggregation are run)
- **scran** (Baseline normalization method (relative counts) whose RMSE/F1 scores are compared)
- **SCnorm** (Baseline normalization method scored against the simulated truth)
- **sctransform** (Baseline normalization method scored against the simulated truth)
- **PsiNorm** (Baseline normalization method scored against the simulated truth)

## Examples

```
Result <- LocASN(countmatrix = countsimdata); metrics <- score_rmse_f1(Result$NormalizedData, truth, planted_de_labels); log1p(Result$NormalizedData)
```

## Evaluation signals

- scKWARN's RMSE should be the lowest across all five scenarios in each of the three simulation settings (15-row rmse_table.csv)
- scKWARN's F1 score should be the highest across all five scenarios in each setting (15-row f1_table.csv), outperforming RC, scran, SCnorm, sctransform, and PsiNorm regardless of error distribution or count-depth assumptions
- Each metric table contains exactly 15 rows (3 settings x 5 scenarios) covering library size, RNA composition, dropout, population balance, and DE-level variations
- Normalized values used for scoring equal counts divided by the per-cell scaling factor (counts/scalingFactor), and clustering uses log1p(Result$NormalizedData)
- Metrics are computed per replicate (100 replicates per scenario) and aggregated per setting/scenario

## Limitations

- F1/RMSE evaluation is only meaningful on simulated data where the truth (normalized values and planted DE/cluster structure) is known; results depend on the simulation protocol (gamma prior on means, rnbinom with size = 0.1)
- Conclusions are relative to the specific baselines compared (RC, scran, SCnorm, sctransform, PsiNorm) and the five scenario axes varied; other confounders are not covered
- Existing methods may over- or under-correct due to technical biases beyond sequencing depth and restrictive model/distribution assumptions, which the scoring must capture but real datasets cannot verify without truth

## Evidence

- [supplementary] RMSE against simulated truth: [evidence span withheld — anchor status: not_found]
- [results] outperformed baselines: [evidence span withheld — anchor status: not_found]
- [readme] log1p for clustering: "# log1p(Result$NormalizedData) for clustering"
- [readme] simulated NB count matrix: "mu <- rgamma(G, shape = 2, rate = 2)
NB_cell <- function(j) rnbinom(G, size = 0.1, mu = mu)"
- [abstract] over/under-correction motivation: "these methods may lead to over or under-correction due to presence of technical biases beyond sequencing depth and the restrictive assumption on models and distributions"
