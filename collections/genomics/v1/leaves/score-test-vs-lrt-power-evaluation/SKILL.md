---
name: score-test-vs-lrt-power-evaluation
description: Use when planning an ordinal-phenotype GWAS with OrdinalGWAS.jl and needing
  to decide whether score-test-only screening suffices or when top hits must be re-analyzed
  with the LRT to obtain p-values and effect sizes.
when_to_use_negative:
- When sample size is small (well below n=2500), since the score test's power advantage
  over the LRT is no longer guaranteed and LRT is preferred
- When final reported associations are needed — score test alone does not yield effect
  sizes; the LRT must be re-run on top hits
- When the phenotype is truly continuous or binary with no natural ordering, where
  linear or logistic regression baselines are the appropriate models
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0203
tools:
- name: OrdinalGWAS.jl
  role: Runs score-test GWAS across SNPs and LRT re-analysis on top hits for the ordered
    multinomial (proportional odds) model
  repo: https://github.com/OpenMendel/OrdinalGWAS.jl
- name: OrdinalMultinomialModels.jl
  role: Provides the underlying fit of ordered multinomial (proportional odds / ordered
    Probit) models used for null model fitting and testing
  repo: https://github.com/OpenMendel/OrdinalMultinomialModels.jl
- name: SnpArrays
  role: Reads and manipulates PLINK .bed/.bim/.fam genotype data consumed by the GWAS
    testing loop
  repo: https://github.com/OpenMendel/SnpArrays.jl
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/gepi.22276
    title: Ordered multinomial regression for genetic association analysis of ordinal
      phenotypes at Biobank scale
  source_content_hash: sha256:5e1777226d4c9cb3f3d945b5bb661d14984cec0950a1ad6327c6a5939db934f7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMendel/OrdinalGWAS.jl
  tools:
  - CSV.jl
  - DataFrames
  - OrdinalGWAS.jl
  - OrdinalMultinomialModels.jl
  - SnpArrays
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# score-test-vs-lrt-power-evaluation

## Summary

Evaluates whether the score test implemented in OrdinalGWAS.jl provides power comparable to the likelihood ratio test (LRT) for ordinal-trait GWAS at reasonable sample sizes (e.g., n=2500 or greater), justifying the score-test screen-then-LRT-refine workflow.

## When to use

Use when planning an ordinal-phenotype GWAS with OrdinalGWAS.jl and needing to decide whether score-test-only screening suffices or when top hits must be re-analyzed with the LRT to obtain p-values and effect sizes.

## When NOT to use

- When sample size is small (well below n=2500), since the score test's power advantage over the LRT is no longer guaranteed and LRT is preferred
- When final reported associations are needed — score test alone does not yield effect sizes; the LRT must be re-run on top hits
- When the phenotype is truly continuous or binary with no natural ordering, where linear or logistic regression baselines are the appropriate models

## Inputs

- Simulated ordinal phenotype matrix (J ordered categories, e.g., J=5 COPD-like GOLD stages) with genotype dosages and covariates at n=2500 per replicate
- PLINK/VCF/BGEN genotype files readable by SnpArrays
- Null model formula and covariate file for the proportional odds model

## Outputs

- Empirical power estimates for score test vs LRT per scenario and sample size
- Empirical type I error rates under gamma=0 null replicates
- score.pval.txt from score-test screening and LRT p-values/effect sizes for top hits
- Summary table (model, scenario, type I error, power) e.g., simulation_results.csv

## How to apply

Fit the proportional-odds null model (e.g., @formula(trait ~ sex)) without SNP effects in OrdinalGWAS.jl, then run the score test across all SNPs. The paper's justification is that 'when sample size (e.g., n = 2500 or greater) is reasonable, score test provides comparable power with LRT' (Supplementary Section S.6 / Supplementary Figure 9). At biobank scale, adopt the recommended two-step workflow: perform score-test GWAS over all SNPs (results in score.pval.txt), sort p-values and select top hits, then re-run the LRT only on those top hits to obtain final p-values and effect sizes, because the LRT is more expensive. For the comparison itself, repeat null (gamma=0) and alternative replicates in simulated ordinal data (e.g., J=5 categories in the COPD-like setting) and compare empirical power and type I error between score test and LRT at fixed nominal levels. Validate that score-test power converges to LRT power as n reaches ~2500 and that both are properly calibrated.

## Related tools

- **OrdinalGWAS.jl** (Runs score-test GWAS across SNPs and LRT re-analysis on top hits for the ordered multinomial (proportional odds) model) — https://github.com/OpenMendel/OrdinalGWAS.jl
- **OrdinalMultinomialModels.jl** (Provides the underlying fit of ordered multinomial (proportional odds / ordered Probit) models used for null model fitting and testing) — https://github.com/OpenMendel/OrdinalMultinomialModels.jl
- **SnpArrays** (Reads and manipulates PLINK .bed/.bim/.fam genotype data consumed by the GWAS testing loop) — https://github.com/OpenMendel/SnpArrays.jl

## Examples

```
julia> ordinalgwas(@formula(trait ~ sex), covfile="covariate.txt", plfile="snpplink"); # then sort score.pval.txt and re-run with LRT on top hits
```

## Evaluation signals

- At n=2500, empirical power of the score test should be comparable to the LRT, as reported in Supplementary Section S.6 / Supplementary Figure 9
- Empirical type I error of both tests should be near the nominal level across null (gamma=0) replicates
- Two-step workflow output should include score.pval.txt from step 1 and LRT results only for the selected top SNPs (e.g., top 10 by sorted score p-value)
- Score-test p-values for top hits should be concordant in ranking with LRT p-values on the same SNPs
- Aggregate results table should show per-scenario rows (model, scenario, type I error, power) with the ordered multinomial model matching or exceeding baselines

## Limitations

- The score test's power equivalence to the LRT holds only at reasonable sample sizes (n=2500 or greater in the paper's simulations); below that the LRT is preferable
- The score test alone yields p-values but no effect-size estimates; LRT re-run is required for effect sizes on top hits
- Link function choice (logit vs Probit) affects fit and should be selected via deviance (-2 x loglikelihood) at MLEs

## Evidence

- [methods] Two-step screening workflow with score test then LRT: [evidence span withheld — anchor status: not_found]
