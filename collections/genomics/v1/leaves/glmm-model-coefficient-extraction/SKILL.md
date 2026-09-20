---
name: glmm-model-coefficient-extraction
description: Use when fitted twosigma/glmmTMB model objects are in hand and per-gene
  conditional-mean fixed-effect coefficients, Z values, or Wald p-values (e.g., for
  t2d_sim) must be pulled into a summary table.
when_to_use_negative:
- Input is a raw count or expression matrix rather than fitted model objects — this
  skill operates on the glmmTMB fit list from twosigma()/twosigma_custom(); fit first.
- 'The goal is a joint likelihood-ratio test of a covariate in both model components
  (H0: alpha1 = beta1 = 0) — use lr.twosigma or lr.twosigma_custom (supplying lr.df
  for custom formulas) instead of mean-component Wald statistics alone.'
- Set-level geneset testing or custom contrast matrices are needed — the README notes
  contrast functionality is ongoing and routes users to twosigmag with index_test=list(c(1,2))
  and all_as_ref=TRUE.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: twosigma
  role: Provides twosigma()/twosigma_custom() whose returned fits$fit list of glmmTMB
    objects is the source of the extracted conditional-mean coefficients and Wald
    statistics
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Fits the two-component ZINB mixed model per gene; summary(x)$coefficients$cond
    on glmmTMB objects exposes the 'Estimate' and 'z value' columns extracted here
- name: R
  role: Language for the calc_* extractor helpers (calc_logFC, calc_Z, calc_p.vals,
    calc_Stouffer) applied via lapply/unlist over fits$fit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:47b9089ebeda59fe3f3df59cae09457db9e94d9ddecc9c2f69880b8a0fb66145
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edvanburen/twosigma
  tools:
  - R
  - glmmTMB
  - twosigma
  - twosigma (R package)
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# glmm-model-coefficient-extraction

## Summary

Extracts per-gene fixed-effect coefficient estimates, Z values, and Wald p-values for a covariate (e.g., t2d_sim) from the conditional-mean component of glmmTMB-fitted TWO-SIGMA zero-inflated negative binomial mixed models, guarding failed fits as NA. This yields Wald-based gene-level association statistics to pair with lr.twosigma likelihood-ratio results.

## When to use

Use when fitted twosigma/glmmTMB model objects are in hand and per-gene conditional-mean fixed-effect coefficients, Z values, or Wald p-values (e.g., for t2d_sim) must be pulled into a summary table.

## When NOT to use

- Input is a raw count or expression matrix rather than fitted model objects — this skill operates on the glmmTMB fit list from twosigma()/twosigma_custom(); fit first.
- The goal is a joint likelihood-ratio test of a covariate in both model components (H0: alpha1 = beta1 = 0) — use lr.twosigma or lr.twosigma_custom (supplying lr.df for custom formulas) instead of mean-component Wald statistics alone.
- Set-level geneset testing or custom contrast matrices are needed — the README notes contrast functionality is ongoing and routes users to twosigmag with index_test=list(c(1,2)) and all_as_ref=TRUE.

## Inputs

- List of fitted glmmTMB model objects returned by twosigma()/twosigma_custom() in fits$fit, one per gene
- Name (or integer column position) of the fixed-effect covariate to extract, present in the conditional-mean design (e.g., binary disease status t2d_sim alongside age and cellular detection rate)
- Non-negative integer count matrix, mean_covar/zi_covar matrices without intercepts, and individual-level id vector, if fitting must be performed before extraction

## Outputs

- Per-gene numeric vectors of conditional-mean coefficient estimates ('Estimate'), Z values ('z value'), and Wald p-values keyed to the tested covariate row
- Optional per-gene Stouffer statistic combining the mean- and zero-inflation-component Z values
- Per-gene results table (gene, mean-component coefficient, Z value, Wald p-value, LR statistic, LR p-value), e.g. written to twosigma_lr_results.csv

## How to apply

Fit each gene's two-component ZINB mixed model (logit link for drop-out, log link for conditional mean, random intercepts per component) with twosigma()/twosigma_custom(), which employ glmmTMB, then iterate over the returned fits$fit list with lapply. For each element, convert glmmTMB objects via summary() and index the conditional-mean table with $coefficients$cond, selecting the row named for the tested covariate (e.g., 't2d_sim') and the columns 'Estimate' and 'z value' (column 4 holds the Wald p-value). Apply the README's guard: if the element is a character string rather than a glmmTMB object (a failed fit), return NA instead of aborting. Unlist results into per-gene vectors (logFC, Zstats, p.val_Zstat) and optionally compute Stouffer's combination (z_cond + z_zi)/sqrt(2) of the mean- and zero-inflation-component Z values. Merge these Wald statistics with lr.twosigma outputs (LR_stat, LR_p.val) into one row per gene and write twosigma_lr_results.csv; the mixed-model basis matters because ignoring random effects can underestimate standard errors and inflate type-I errors.

## Related tools

- **twosigma** (Provides twosigma()/twosigma_custom() whose returned fits$fit list of glmmTMB objects is the source of the extracted conditional-mean coefficients and Wald statistics) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Fits the two-component ZINB mixed model per gene; summary(x)$coefficients$cond on glmmTMB objects exposes the 'Estimate' and 'z value' columns extracted here)
- **R** (Language for the calc_* extractor helpers (calc_logFC, calc_Z, calc_p.vals, calc_Stouffer) applied via lapply/unlist over fits$fit)

## Examples

```
logFC<-unlist(lapply(fits$fit,calc_logFC)); Zstats<-unlist(lapply(fits$fit,calc_Z))
```

## Evaluation signals

- Output contains exactly one row per simulated gene with a finite LR statistic and Wald and LR p-values in [0, 1].
- Elements of fits$fit that are character strings (failed fits) map to NA rather than crashing the lapply loop or being silently dropped.
- Row indexing uses exactly the tested covariate name ('t2d_sim') against summary(x)$coefficients$cond, with 'Estimate', 'z value', and column 4 (Wald p-value) as columns.
- Extracted estimates and Z values match the values printed by running summary() directly on the underlying glmmTMB object.
- If Stouffer's statistic is computed, it equals the sum of the mean- and ZI-component Z values divided by sqrt(2).

## Limitations

- Wald statistics test only the conditional-mean component; they do not address the joint null H0: alpha1 = beta1 = 0 that lr.twosigma evaluates via the likelihood ratio.
- lr.twosigma assumes the tested variable is in both model components, and an integer covar_to_test position is not checked for consistency between mean_covar and zi_covar.
- Fits that fail return character strings, so Wald statistics are unavailable (NA) for those genes; and excluding random effects on the assumption of independence yields underestimated standard errors and inflated type-I error (article's table 3 example).

## Evidence

- [readme] if(class(x)=="glmmTMB"){x<-summary(x)}
    if(is.character(x)){return(NA)}else{
      x$coefficients$cond['t2d_sim','z value']: "if(class(x)=="glmmTMB"){x<-summary(x)}
    if(is.character(x)){return(NA)}else{
      x$coefficients$cond['t2d_sim','z value']"
