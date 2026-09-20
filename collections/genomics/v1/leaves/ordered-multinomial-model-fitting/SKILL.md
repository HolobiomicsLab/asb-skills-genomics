---
name: ordered-multinomial-model-fitting
description: Use when the phenotype is an ordered categorical trait (e.g., disease
  status undiagnosed/pre-disease/mild/moderate/severe or GOLD stages) coded 1-J with
  covariates, and a null model without SNP effects is needed for GWAS score testing.
when_to_use_negative:
- The trait is continuous or binary — linear regression or logistic regression baselines
  are the appropriate models, not ordered multinomial regression
- Categories have no meaningful ordering — use multinomial regression that ignores
  ordinality instead
- The proportional odds assumption is heavily violated and a category-specific (non-proportional)
  model is required; the article notes existing tests for this violation are liberal
  and verification is difficult
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_2269
tools:
- name: OrdinalGWAS.jl
  role: High-level ordinalgwas function that fits the null ordered multinomial (proportional
    odds) model from a formula, covariate file and PLINK/VCF/BGEN genotype input,
    and can serialize the fitted null model
  repo: https://github.com/OpenMendel/OrdinalGWAS.jl
- name: OrdinalMultinomialModels.jl
  role: Underlying Julia utilities for fitting ordered multinomial models, including
    proportional odds (ordered logit) and ordered probit as special cases
  repo: https://github.com/OpenMendel/OrdinalMultinomialModels.jl
- name: SnpArrays.jl
  role: Reads and manipulates the compressed PLINK .bed biallelic SNP data supplied
    to the null-model fit and subsequent score-test scan
  repo: https://github.com/OpenMendel/SnpArrays.jl
- name: Julia 1.6
  role: Runtime language; OrdinalGWAS.jl requires Julia v1.6 or later and installation
    via the Julia package manager
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1002/gepi.22276
    title: Ordered multinomial regression for genetic association analysis of ordinal
      phenotypes at Biobank scale
  source_content_hash: sha256:c7dd1de6751e70154fc90bc27911e43f55342fe8c1718fd336beb083363f94e3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMendel/OrdinalGWAS.jl
  tools:
  - CSV.jl
  - DataFrames
  - Julia 1.6
  - OpenMendel
  - OrdinalGWAS.jl
  - OrdinalMultinomialModels.jl
  - SnpArrays
  - SnpArrays.jl
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# ordered-multinomial-model-fitting

## Summary

Fit an ordered multinomial (proportional odds, e.g. LogitLink) null model to an ordinal trait and covariates using OrdinalMultinomialModels.jl/OrdinalGWAS.jl, the first step of the ordinalgwas workflow before single-SNP score-test scanning.

## When to use

Use when the phenotype is an ordered categorical trait (e.g., disease status undiagnosed/pre-disease/mild/moderate/severe or GOLD stages) coded 1-J with covariates, and a null model without SNP effects is needed for GWAS score testing.

## When NOT to use

- The trait is continuous or binary — linear regression or logistic regression baselines are the appropriate models, not ordered multinomial regression
- Categories have no meaningful ordering — use multinomial regression that ignores ordinality instead
- The proportional odds assumption is heavily violated and a category-specific (non-proportional) model is required; the article notes existing tests for this violation are liberal and verification is difficult

## Inputs

- Covariate file (e.g. covariate.txt) containing the ordinal trait coded 1-J and covariates such as sex
- PLINK binary genotype files (.bed/.bim/.fam), e.g. hapmap3 with 324 samples at 13,928 SNPs
- Null model formula, e.g. @formula(trait ~ sex)
- Link function choice (default LogitLink())

## Outputs

- ordinalgwas.null.txt with fitted intercepts (intercept1|2, intercept2|3, intercept3|4), covariate coefficients, standard errors, t values and p-values
- Serialized null model (fittednullmodel.jls) for reuse across parallel jobs

## How to apply

Specify the null model as a first-argument formula without SNP effects, e.g. `@formula(trait ~ sex)`, passing the covariate file (ordinal trait coded 1-4 in the package example) and the PLINK prefix (.bed/.bim/.fam, e.g. hapmap3 with 324 samples and 13,928 SNPs). Use the default LogitLink() proportional odds model unless goodness-of-fit (deviance, -2 x loglikelihood at MLEs) favors another link such as probit. Missing genotypes are imputed according to minor allele frequencies during the scan that follows. Save the fitted null model (intercept1|2, intercept2|3, intercept3|4 and covariate coefficients with SEs, t values, p-values) to ordinalgwas.null.txt, or serialize it to fittednullmodel.jls for reuse across parallel per-chromosome GWAS jobs.

## Related tools

- **OrdinalGWAS.jl** (High-level ordinalgwas function that fits the null ordered multinomial (proportional odds) model from a formula, covariate file and PLINK/VCF/BGEN genotype input, and can serialize the fitted null model) — https://github.com/OpenMendel/OrdinalGWAS.jl
- **OrdinalMultinomialModels.jl** (Underlying Julia utilities for fitting ordered multinomial models, including proportional odds (ordered logit) and ordered probit as special cases) — https://github.com/OpenMendel/OrdinalMultinomialModels.jl
- **SnpArrays.jl** (Reads and manipulates the compressed PLINK .bed biallelic SNP data supplied to the null-model fit and subsequent score-test scan) — https://github.com/OpenMendel/SnpArrays.jl
- **Julia 1.6** (Runtime language; OrdinalGWAS.jl requires Julia v1.6 or later and installation via the Julia package manager)

## Examples

```
using Pkg; pkg"add OrdinalGWAS"; using OrdinalGWAS, SnpArrays; ordinalgwas(@formula(trait ~ sex), "covariate.txt", "hapmap3")
```

## Evaluation signals

- ordinalgwas.null.txt exists and contains one intercept row per adjacent category pair (intercept1|2 through intercept(J-1)|J) plus covariate coefficient rows with estimate, SE, t value and p-value columns
- Fitted coefficients reproduce the documented example values, e.g. intercept1|2 estimate -1.48564 and sex estimate 0.424656 for @formula(trait ~ sex) with LogitLink() on the hapmap3 example data
- Link-function choice is justified by comparing deviance (-2 x loglikelihood at MLEs) across candidate links, selecting the better fit
- The null model formula contains no SNP term — SNP effects enter only in the subsequent score-test/LRT scan steps
- Serialized fittednullmodel.jls reloads and yields identical coefficient estimates when reused across parallel per-chromosome jobs

## Limitations

- No software version is stated for OrdinalGWAS.jl in the paper; only 'Julia v1.6 or later' is documented
- Verification of proportional odds assumption violations is difficult because existing tests are liberal, so a low p-value may reflect assumption violation that cannot be confirmed
- PGEN format support is planned but not yet available in OrdinalGWAS.jl; currently PLINK, VCF (dosage and genotype) and BGEN formats are supported

## Evidence

- [methods] The first argument specifies the null model without SNP effects, e.g., `@formula(trait ~ sex)`.: "The first argument specifies the null model without SNP effects, e.g., `@formula(trait ~ sex)`."
- [methods] we choose the link function based on data according to goodness of fit measures such as the deviance (−2 × loglikelihood) at respective MLEs.: "we choose the link function based on data according to goodness of fit measures such as the deviance (−2 × loglikelihood) at respective MLEs."
- [methods] Step 1: fit null model and save result to file `fittednullmodel.jls`: "Step 1: fit null model and save result to file `fittednullmodel.jls`"
- [methods] OrdinalGWAS.jl is a Julia package for performing genome-wide association studies (GWAS) for ordered categorical phenotypes using proportional odds model: "OrdinalGWAS.jl is a Julia package for performing genome-wide association studies (GWAS) for ordered categorical phenotypes using proportional odds model"
- [methods] This package requires Julia v1.6 or later. The package has been registered and can be installed using Julia package manager.: "This package requires Julia v1.6 or later. The package has been registered and can be installed using Julia package manager."
- [methods] `ordinalgwas` imputes missing genotypes according to minor allele frequencies.: "`ordinalgwas` imputes missing genotypes according to minor allele frequencies."
- [readme] The package currently supports PLINK, VCF (both dosage and genotype data) file formats, and BGEN file formats. We plan to add PGEN support in the future.: "The package currently supports [PLINK](https://zzz.bwh.harvard.edu/plink/), [VCF](https://en.wikipedia.org/wiki/Variant_Call_Format) (both dosage and genotype data) file formats, and"
