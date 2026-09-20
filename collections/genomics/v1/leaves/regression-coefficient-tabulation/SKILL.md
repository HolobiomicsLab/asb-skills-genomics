---
name: regression-coefficient-tabulation
description: Use when fitting simulated or real scRNA-seq counts under alternative
  model specifications (e.g., no ZI, intercept-only ZI, covariate-inclusive ZI) and
  you need a unified per-gene table of coefficient estimates and standard errors for
  comparison.
when_to_use_negative:
- When the goal is gene set testing or set-level enrichment—the adhoc procedure and
  per-gene coefficient tabulation are not recommended there because geneset testing
  requires a common gene-level null across genes
- When the analysis goal is hypothesis testing of a covariate via likelihood ratio
  test rather than tabulating coefficient estimates—use lr.twosigma / lr.twosigma_custom
  instead
- When random-effect variance components need to be formally tested—use test.vc.twosigma
  rather than comparing fixed-effect tables
edam_operation: http://edamontology.org/operation_3657
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: twosigma (R package)
  role: Fitting interface (twosigma / twosigma_custom) wrapping glmmTMB to fit zero-inflated
    negative binomial models with alternative ziformula specifications per gene
  repo: https://github.com/edvanburen/twosigma
- name: glmmTMB
  role: Underlying fitting engine; coefficient estimates and standard errors are extracted
    from its cond component of the summary
- name: R
  role: Language for running the fits, assembling the per-gene table, and writing
    the CSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/gepi.22361
    title: 'TWO‐SIGMA: A novel two‐component single cell model‐based association method
      for single‐cell RNA‐seq data'
  source_content_hash: sha256:66a6fe8965f2981650ad291ed7523d9db9de58886eb8248fca61e2abc58613dd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edvanburen/twosigma
  tools:
  - R
  - glmmTMB
  - twosigma (R package)
  edam_operation: http://edamontology.org/operation_3657
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# regression-coefficient-tabulation

## Summary

Tabulates per-gene fixed-effect coefficient estimates and standard errors from alternative model specifications—in this context, zero-inflated negative binomial fits from glmmTMB via the twosigma R package—into a single comparison table so conditional-mean estimates can be compared side by side.

## When to use

Use when fitting simulated or real scRNA-seq counts under alternative model specifications (e.g., no ZI, intercept-only ZI, covariate-inclusive ZI) and you need a unified per-gene table of coefficient estimates and standard errors for comparison.

## When NOT to use

- When the goal is gene set testing or set-level enrichment—the adhoc procedure and per-gene coefficient tabulation are not recommended there because geneset testing requires a common gene-level null across genes
- When the analysis goal is hypothesis testing of a covariate via likelihood ratio test rather than tabulating coefficient estimates—use lr.twosigma / lr.twosigma_custom instead
- When random-effect variance components need to be formally tested—use test.vc.twosigma rather than comparing fixed-effect tables

## Inputs

- Simulated count matrix of 10,000 independent genes (non-negative integer counts, e.g., from simulate_zero_inflated_nb_random_effect_data in the twosigma R package)
- Simulated covariates: binary disease status, age, and cellular detection rate (CDR)
- Per-gene model specification choices (zi_covar=0, zi_covar=1, or covariate-inclusive ziformula)

## Outputs

- A per-gene coefficient table written to zinb_specification_comparison.csv with columns: gene, model component, specification, coefficient, estimate, standard error
- Rows for all three ZI specifications per gene with finite coefficient estimates and standard errors

## How to apply

Fit each specification per gene using glmmTMB through the twosigma interface: a negative binomial GLM with no zero-inflation component (zi_covar=0 / ziformula=~0), a zero-inflated model with an intercept-only drop-out component (zi_covar=1 / ziformula=~1), and optionally a ZI model with the full covariate set (disease status, age, cellular detection rate) in the drop-out component. From each fit, extract the conditional-mean (cond) fixed-effect coefficient estimates and standard errors, mirroring the README's pattern of reading x$coefficients$cond from glmmTMB summary objects. Assemble all estimates into one table with columns identifying gene, model component, specification, coefficient, estimate, and standard error, and write to a CSV such as zinb_specification_comparison.csv. The rationale is that mean-model coefficients from the same data under different ZI specifications can then be inspected directly for sensitivity of conditional-mean inference to dropout-model choice.

## Related tools

- **twosigma (R package)** (Fitting interface (twosigma / twosigma_custom) wrapping glmmTMB to fit zero-inflated negative binomial models with alternative ziformula specifications per gene) — https://github.com/edvanburen/twosigma
- **glmmTMB** (Underlying fitting engine; coefficient estimates and standard errors are extracted from its cond component of the summary)
- **R** (Language for running the fits, assembling the per-gene table, and writing the CSV)

## Examples

```
fits <- twosigma(count_matrix, mean_covar=mean_covar_matrix, zi_covar=1, id=id, adhoc=FALSE, mean_re=FALSE, zi_re=FALSE)
```

## Evaluation signals

- Output CSV contains rows for all three ZI specifications (no-ZI, intercept-only ZI, covariate-inclusive ZI) for every gene
- All coefficient estimates and standard errors in the table are finite (no NA/NaN from failed fits)
- No random-effect (variance component) terms appear in any fitted model—ziformula and mean formula contain no (1|id) terms
- Extracted values match the cond (conditional-mean) component of each glmmTMB summary, not the zi component
- Standard errors correspond to the same specification/estimate pairs as the point estimates

## Limitations

- If a fit fails, glmmTMB objects may not summarize—code extracting coefficients must handle non-glmmTMB or character objects (the README's calc_* helpers return NA in that case)
- Intercept-free covariate matrices: mean_covar and zi_covar must be supplied without intercept columns; the intercept-only ZI specification is expressed via zi_covar=1
- Discussion of zero-inflation component performance is deferred to supplement (section 6, supplementary figure S10), so guidance on which ZI specification is preferred is not in the main text

## Evidence

- [methods] The twosigma function can fit the model with an intercept-only zero-inflation component and no random effects (zi_covar=1), complementing the no-ZI specification (zi_covar=0): [evidence span withheld — anchor status: not_found]
