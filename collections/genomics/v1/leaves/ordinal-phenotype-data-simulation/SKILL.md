---
name: ordinal-phenotype-data-simulation
description: Use when you need simulated ordinal-trait GWAS replicates — J ordered
  categories, genotype dosage, covariates at n≈2500 — to benchmark empirical type
  I error and power of association tests before real-cohort analyses.
when_to_use_negative:
- The trait is continuous/quantitative rather than taking ordered discrete values
  — linear regression is the appropriate model and the ordinal simulation framing
  does not apply.
- Phenotype categories are unordered (nominal) — the ordered multinomial machinery
  assumes ordered discrete categories.
- The goal is association testing on real cohort data (e.g., COPDGene or UK Biobank)
  rather than synthetic type I error/power benchmarking.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
tools:
- name: OrdinalGWAS.jl
  role: Provides ordinalgwas (proportional odds, logit link) with test=:score and
    test=:LRT used to analyze simulated ordinal traits, and ships the simtrait-style
    simulation script plus hapmap3 PLINK example data and covariate.txt in its data
    folder
  repo: https://github.com/OpenMendel/OrdinalGWAS.jl
- name: OrdinalMultinomialModels.jl
  role: Fits the ordered multinomial (proportional odds / ordered probit) models that
    the simulated-data type I error and power benchmarks evaluate
  repo: https://github.com/OpenMendel/OrdinalMultinomialModels.jl.git
- name: SnpArrays.jl
  role: Reads and manipulates the compressed PLINK .bed/.bim/.fam genotype matrices
    that provide the genotype dosage backbone of the simulation
  repo: https://github.com/OpenMendel/SnpArrays.jl.git
- name: Julia
  role: Runtime language (v1.6 or later required by OrdinalGWAS.jl) in which the simulation
    script and ordinalgwas calls are executed
- name: CSV.jl
  role: Writes replicate-level and per-SNP results to CSV outputs such as simulation_results.csv
    and score_lrt_power_comparison.csv
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/gepi.22276
    title: Ordered multinomial regression for genetic association analysis of ordinal
      phenotypes at Biobank scale
  source_content_hash: sha256:c2c45d7ec5a194ff4910dae2479364be371014700cd258183225781be1e95063
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMendel/OrdinalGWAS.jl
  tools:
  - CSV.jl
  - DataFrames
  - Julia
  - OrdinalGWAS.jl
  - OrdinalMultinomialModels.jl
  - SnpArrays
  - SnpArrays.jl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# ordinal-phenotype-data-simulation

## Summary

Generates synthetic ordinal-trait GWAS datasets (J ordered phenotype categories, SNP dosage, covariates, configurable SNP effect gamma) to estimate empirical type I error and power for ordered multinomial regression against linear, dichotomized-logistic, and multinomial baselines.

## When to use

Use when you need simulated ordinal-trait GWAS replicates — J ordered categories, genotype dosage, covariates at n≈2500 — to benchmark empirical type I error and power of association tests before real-cohort analyses.

## When NOT to use

- The trait is continuous/quantitative rather than taking ordered discrete values — linear regression is the appropriate model and the ordinal simulation framing does not apply.
- Phenotype categories are unordered (nominal) — the ordered multinomial machinery assumes ordered discrete categories.
- The goal is association testing on real cohort data (e.g., COPDGene or UK Biobank) rather than synthetic type I error/power benchmarking.

## Inputs

- PLINK .bed/.bim/.fam genotype files (e.g., hapmap3 example data from the OrdinalGWAS.jl data folder) read via SnpArrays
- Covariate file covariate.txt for the null model (e.g., @formula(trait ~ sex))
- Simulation script (simtrait.jl-style) with parameters: number of ordered categories J, SNP effect size gamma, sample size n
- Scenario grid: null replicates (gamma=0) and alternative replicates (non-zero SNP effect), including intercept configurations

## Outputs

- Simulated ordinal phenotype vectors with J discrete ordered categories per replicate (J=5 in the COPD-like setting)
- Per-SNP test outputs: fitted null model ordinalgwas.null.txt, score.pval.txt (score-test p-values), and lrt.pval.txt (p-values plus effect sizes and standard errors)
- Summary CSVs: simulation_results.csv keyed by (model, scenario, type I error, power) and score_lrt_power_comparison.csv

## How to apply

Simulate ordinal phenotypes with the package's simtrait-style script (simtrait.jl), combining the hapmap3 PLINK genotype files and covariate.txt from OrdinalGWAS.jl's data folder with a chosen number of ordered categories J (COPD-like setting: J=5, excluding the unclassifiable category), sample size n=2500, and SNP effect gamma — running null replicates at gamma=0 and alternative replicates at non-zero effects. Analyze each replicate with the proportional-odds logit-link null model in ordinalgwas (saving ordinalgwas.null.txt), scoring all SNPs with test=:score and re-running test=:LRT for p-values with effect sizes and standard errors; fit the three baselines on identical data: linear regression on the raw ordinal trait, logistic regression on a dichotomized version, and multinomial regression that ignores ordinality. Let the package impute missing genotypes according to minor allele frequencies, and choose the link function by goodness of fit (deviance, −2×loglikelihood at the MLEs) when comparing link families. Aggregate replicate outcomes into a summary table (model, scenario, type I error, power) saved as simulation_results.csv, plus score_lrt_power_comparison.csv for the score-vs-LRT arm. Validate that the ordered multinomial model's empirical type I error is near nominal, its power matches or exceeds the baselines (logistic regression can show over 30% reduced power when intercepts are disproportionately non-uniform), and score-test power is comparable to LRT at n≥2500 as reported in Supplementary Section S.6 / Supplementary Figure 9.

## Related tools

- **OrdinalGWAS.jl** (Provides ordinalgwas (proportional odds, logit link) with test=:score and test=:LRT used to analyze simulated ordinal traits, and ships the simtrait-style simulation script plus hapmap3 PLINK example data and covariate.txt in its data folder) — https://github.com/OpenMendel/OrdinalGWAS.jl
- **OrdinalMultinomialModels.jl** (Fits the ordered multinomial (proportional odds / ordered probit) models that the simulated-data type I error and power benchmarks evaluate) — https://github.com/OpenMendel/OrdinalMultinomialModels.jl.git
- **SnpArrays.jl** (Reads and manipulates the compressed PLINK .bed/.bim/.fam genotype matrices that provide the genotype dosage backbone of the simulation) — https://github.com/OpenMendel/SnpArrays.jl.git
- **Julia** (Runtime language (v1.6 or later required by OrdinalGWAS.jl) in which the simulation script and ordinalgwas calls are executed)
- **CSV.jl** (Writes replicate-level and per-SNP results to CSV outputs such as simulation_results.csv and score_lrt_power_comparison.csv)

## Examples

```
using OrdinalGWAS; ordinalgwas(@formula(trait ~ sex), "hapmap3", test=:score)
```

## Evaluation signals

- Empirical type I error of the ordered multinomial model across gamma=0 null replicates falls near the nominal significance level.
- Power of the ordered multinomial model matches or exceeds linear, dichotomized-logistic, and multinomial baselines in each simulated scenario; logistic regression shows over 30% reduced power when intercept values are disproportionately non-uniform.
- Score-test power is comparable to LRT power at n=2500 on identical simulated replicates, reproducing the Supplementary Figure 9 benchmark.
- Simulated phenotype vectors contain exactly J discrete ordered levels (J=5 in the COPD-like setting, excluding the unclassifiable category) matched across all four models within each replicate.
- Output schema checks: score.pval.txt holds per-SNP p-values, lrt.pval.txt adds effect sizes and standard errors, and the summary CSV is keyed by model, scenario, type I error, and power.

## Limitations

- Score-vs-LRT power comparability is sample-size dependent — the paper's claim applies at n=2500 or greater; smaller simulated cohorts may not reproduce it.
- Simulated scenarios may not capture real proportional-odds violations, which the authors note are difficult to verify because existing tests are liberal.
- Simulation assets are demonstration-scale (hapmap3 example data, covariate.txt) with no stated software version or UK Biobank accession, so exact biobank-scale replication of the simulation study is not documented.

## Evidence

- [intro] The paper's core evaluation is type I error and power of ordered multinomial regression versus baselines in genetically plausible scenarios: "we systematically investigate the performance (type I error and power) of the ordered multinomial model"
- [methods] Score test provides power comparable to LRT at reasonable sample size (n = 2500 or greater): "When sample size (e.g., n = 2500 or greater) is reasonable, score test provides comparable power with LRT"
- [methods] The COPD-like simulation setting uses J = 5 ordered categories, excluding the unclassifiable category: "For example, in the COPD example J = 5 (excluding the unclassifiable category)."
- [methods] The package's data folder supplies the PLINK/VCF example datasets used by simulation workflows: "The `data` folder of the package contains the example data sets for use with PLINK and VCF Files."
- [readme] Ordinal phenotype target: ordered discrete values such as disease status stages: "It is useful when the phenotype takes ordered discrete values, e.g., disease status (undiagnosed, pre-disease, mild, moderate, severe)."
