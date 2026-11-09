---
name: single-cell-rna-seq-differential-expression-inference
description: Use when you have a raw (non-normalized) scRNA-seq count matrix, cell-level
  covariates such as a binary disease status (e.g. t2d_sim), age, and cellular detection
  rate, plus sample IDs, and need per-gene DE inference while controlling for within-sample
  correlation.
when_to_use_negative:
- 'Input data is log-transformed or normalized (e.g. TPM/log-CPM): TWO-SIGMA models
  observed raw counts directly and the README states ''No normalization is done'',
  so a hurdle model on log scale (e.g. MAST) is the wrong target here'
- The goal is gene-set (competitive) testing rather than per-gene DE inference — use
  the TWO-SIGMA-G twosigmag function instead, and note the ad hoc procedure is not
  recommended for gene set testing
- 'The covariate to test is not present in both model components (or the zero-inflation
  component has only an intercept): lr.twosigma''s assumptions fail and lr.twosigma_custom
  with explicit null/alternative formulas plus lr.df is required'
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: twosigma
  role: R package providing twosigma(), twosigma_custom(), lr.twosigma(), and lr.twosigma_custom()
    for fitting the ZINB mixed model and running the joint likelihood ratio test per
    gene
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Underlying GLMM fitting engine used by the twosigma package to fit the equation
    (2) zero-inflated negative binomial mixed model; fitted objects are returned as
    glmmTMB-class objects
- name: R
  role: Execution environment; package requires R >= 3.6.3 and minimum versions of
    multcomp (>= 1.4-13), pscl (>= 1.5.5), pbapply (>= 1.4.0)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:a77eaf32dd508708858e9e56a320b338a5a1f04d01ccde968c29744cbb784f62
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edvanburen/twosigma
  tools:
  - R
  - glmmTMB
  - twosigma
  - twosigma (R package)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# single-cell-rna-seq-differential-expression-inference

## Summary

Fit the TWO-SIGMA two-component zero-inflated negative binomial mixed model (logit link for drop-out, log link for conditional mean, sample-level random intercepts) to raw scRNA-seq counts per gene, then test a fixed-effect covariate jointly across both components via a likelihood ratio test to obtain per-gene differential expression statistics.

## When to use

Use when you have a raw (non-normalized) scRNA-seq count matrix, cell-level covariates such as a binary disease status (e.g. t2d_sim), age, and cellular detection rate, plus sample IDs, and need per-gene DE inference while controlling for within-sample correlation.

## When NOT to use

- Input data is log-transformed or normalized (e.g. TPM/log-CPM): TWO-SIGMA models observed raw counts directly and the README states 'No normalization is done', so a hurdle model on log scale (e.g. MAST) is the wrong target here
- The goal is gene-set (competitive) testing rather than per-gene DE inference — use the TWO-SIGMA-G twosigmag function instead, and note the ad hoc procedure is not recommended for gene set testing
- The covariate to test is not present in both model components (or the zero-inflation component has only an intercept): lr.twosigma's assumptions fail and lr.twosigma_custom with explicit null/alternative formulas plus lr.df is required

## Inputs

- count_matrix: matrix of non-negative integer scRNA-seq counts (raw, no normalization), one row per cell or per gene as expected by the function
- mean_covar: covariate matrix for the conditional-mean model WITHOUT an intercept term (e.g. model.matrix output), columns such as t2d_sim, age, CDR
- zi_covar: covariate matrix for the zero-inflation model WITHOUT an intercept term
- id: vector of individual-level sample IDs, one entry per cell
- covar_to_test: column name (string) or column position (integer) of the covariate to test, present in both mean_covar and zi_covar

## Outputs

- Per-gene fitted TWO-SIGMA model objects (glmmTMB class) with fixed-effect estimates, Z values, and Wald p-values in $coefficients$cond
- lr.fit object containing per-gene likelihood ratio statistics (LR_stat) and p-values (LR_p.val) for the joint null H0: alpha1 = beta1 = 0
- Per-gene results table (gene, mean-component coefficient, Z value, Wald p-value, LR statistic, LR p-value), e.g. written to twosigma_lr_results.csv

## How to apply

Load the count matrix, design covariates, and individual-level ID vector, then fit the TWO-SIGMA ZINB mixed model per gene with twosigma() (or twosigma_custom() with formulas like count ~ mean_covar + (1|id)), letting the default ad hoc procedure regress one-way-ANOVA Pearson residuals on the sample indicator to decide whether random intercepts are needed in the mean and zero-inflation components. The models are fit via the glmmTMB package for GLMM fitting, and no normalization is performed on the counts. To test a covariate such as t2d_sim, call lr.twosigma() with covar_to_test set to the covariate's column name, which tests the joint null H0: alpha1 = beta1 = 0 (covariate absent from both mean and zero-inflation components). Extract the likelihood ratio statistic and p-value from lr.fit$LR_stat and lr.fit$LR_p.val, and pull the conditional-mean component fixed-effect estimate, 'z value', and Wald p-value for t2d_sim from each fitted model via x$coefficients$cond. Assemble a per-gene table (gene, mean coefficient, Z, Wald p, LR stat, LR p) and write it to CSV; validate that each gene has a finite LR statistic and Wald and LR p-values in [0, 1].

## Related tools

- **twosigma** (R package providing twosigma(), twosigma_custom(), lr.twosigma(), and lr.twosigma_custom() for fitting the ZINB mixed model and running the joint likelihood ratio test per gene) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Underlying GLMM fitting engine used by the twosigma package to fit the equation (2) zero-inflated negative binomial mixed model; fitted objects are returned as glmmTMB-class objects)
- **R** (Execution environment; package requires R >= 3.6.3 and minimum versions of multcomp (>= 1.4-13), pscl (>= 1.5.5), pbapply (>= 1.4.0))

## Examples

```
lr.fit <- lr.twosigma(count_matrix, mean_covar, zi_covar, covar_to_test="t2d_sim", mean_re=TRUE, zi_re=TRUE, id=id); LR_stat <- lr.fit$LR_stat; LR_p.val <- lr.fit$LR_p.val
```

## Evaluation signals

- The output table contains exactly one row per gene, each with a finite LR statistic and both Wald and LR p-values in [0, 1]
- Input count matrix passed to twosigma/lr.twosigma contains only non-negative integers with no missing values, and mean_covar/zi_covar matrices contain no intercept column
- Extracted t2d_sim coefficients are pulled from the 'cond' component of glmmTMB summary output (x$coefficients$cond['t2d_sim',]) and non-converged/character entries are mapped to NA by the calc_* helper pattern
- Simulated-data validation reproduces the documented workflow: covar_to_test='t2d_sim' yields LR_stat and LR_p.val per gene with no NA rows for successfully fit genes
- lr.twosigma results on the same covariate are consistent with Wald inference: genes with large |Z| in the mean component tend to show large LR statistics under the joint null

## Limitations

- lr.twosigma assumes the tested variable is in BOTH components (mean and zero-inflation); the README notes users wishing to test in other cases must use lr.twosigma_custom and ensure their formulas form a valid likelihood ratio test
- If covar_to_test is given as an integer, there is no check that it corresponds to the same covariate in both mean_covar and zi_covar matrices
- For custom formulas, the user must supply lr.df (degrees of freedom, non-negative integer); errors here invalidate the LR p-value
- Ignoring random effects can lead to underestimated standard errors and inflated type-I errors; however the ad hoc random-effect screening is not recommended for gene set testing because a common gene-level null is then unclear
- No normalization is performed on counts, and no software version details are given in the article's discussion section

## Evidence
