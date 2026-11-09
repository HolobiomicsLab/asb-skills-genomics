---
name: hotspot-mutation-prognostic-evaluation
description: Use when a MAF file plus per-sample survival group labels (e.g., G2/G4
  in an RNMF Groupfile) are available and the goal is to test whether a named hotspot
  mutation independently predicts survival outcome.
when_to_use_negative:
- Input is already a gene-level or signature-level abundance table (e.g., geneCumulativeContributionAbundance.txt)
  rather than a per-mutation MAF — use RNMF's genePerMutSigs/samPerMutSigs association
  functions instead
- No survival/group outcome labels are available, making any Cox model unidentifiable;
  RNMF's Fisher-style association tests (samFisherSigs, Mutfreq=0.04, threshold=0.06,
  Qvalue=0.1) would be the appropriate alternative
edam_operation: http://edamontology.org/operation_3561
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: RNMF
  role: Provides the example MAF file (maffile) and Groupfile (Gfile) and the sample.id='Tumor_Sample_Barcode'
    / Hugo='Hugo_Symbol' keying conventions from which per-sample TP53 hotspot mutation
    status is extracted
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Implementation language for data merging and Cox model fitting
  repo: https://github.com/zhenzhang-li/RNMF
- name: R survival package
  role: Fits the multivariate Cox proportional hazards model with R248Q status and
    group annotation as covariates (survival 2.40–1)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:ffea7ec653c93644f034d4cc3e9602c43df635f029883fef48c31ebaa2baa010
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - R
  - R survival package
  - RNMF
  edam_operation: http://edamontology.org/operation_3561
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# hotspot-mutation-prognostic-evaluation

## Summary

Assesses whether a specific hotspot mutation (e.g., TP53 R248Q in ESCC) is an independent prognosticator by extracting per-sample hotspot status from an RNMF-style MAF/Groupfile setup and fitting a multivariate Cox proportional hazards model with the R survival package.

## When to use

Use when a MAF file plus per-sample survival group labels (e.g., G2/G4 in an RNMF Groupfile) are available and the goal is to test whether a named hotspot mutation independently predicts survival outcome.

## When NOT to use

- Input is already a gene-level or signature-level abundance table (e.g., geneCumulativeContributionAbundance.txt) rather than a per-mutation MAF — use RNMF's genePerMutSigs/samPerMutSigs association functions instead
- No survival/group outcome labels are available, making any Cox model unidentifiable; RNMF's Fisher-style association tests (samFisherSigs, Mutfreq=0.04, threshold=0.06, Qvalue=0.1) would be the appropriate alternative

## Inputs

- Mutation Annotation Format (MAF) file (e.g., RNMF example maffile such as 'Regular.ESCC.exon.WES.maf-input-nonsilent.txt') with columns Tumor_Sample_Barcode, Hugo_Symbol, and mutation allele/protein-change fields
- Groupfile (Groupfile.txt) mapping sample.id to survival group labels (G2/G4)

## Outputs

- cox_regression_results.csv containing per-covariate regression coefficients, hazard ratios with confidence intervals, and p-values
- Binary per-sample hotspot mutation status indicator (e.g., TP53 R248Q) merged with survival group labels

## How to apply

Load the example MAF file (maffile) and Groupfile (Gfile) from the RNMF package, keyed on sample.id='Tumor_Sample_Barcode' and Hugo='Hugo_Symbol', and derive a per-sample binary indicator for the hotspot mutation of interest (e.g., TP53 R248Q). Merge this indicator with the survival group labels (G2/G4) from the Groupfile into a single per-sample covariate table. Fit a multivariate Cox proportional hazards model using the R survival package (2.40–1) with the R248Q status and group annotation as covariates. Extract regression coefficients, hazard ratios with 95% confidence intervals, and p-values per covariate and write them to cox_regression_results.csv. Judge success by validation criteria: the hotspot's HR should exceed 1 (poorer prognosis) with p < 0.05, establishing it as an independent prognostic factor alongside the group covariate.

## Related tools

- **RNMF** (Provides the example MAF file (maffile) and Groupfile (Gfile) and the sample.id='Tumor_Sample_Barcode' / Hugo='Hugo_Symbol' keying conventions from which per-sample TP53 hotspot mutation status is extracted) — https://github.com/zhenzhang-li/RNMF
- **R** (Implementation language for data merging and Cox model fitting) — https://github.com/zhenzhang-li/RNMF
- **R survival package** (Fits the multivariate Cox proportional hazards model with R248Q status and group annotation as covariates (survival 2.40–1))

## Examples

```
library(survival); fit <- coxph(Surv(time, status) ~ TP53_R248Q + group, data=cov); write.csv(summary(fit)$coefficients, 'cox_regression_results.csv')
```

## Evaluation signals

- The R248Q covariate's hazard ratio is > 1, indicating association with poorer survival
- The R248Q covariate's p-value from the Cox model is below the paper's significance threshold of 0.05
- cox_regression_results.csv contains coefficients, HRs with confidence intervals, and p-values for every covariate entered in the model (R248Q status and group annotation)
- The per-sample covariate table row count matches the number of samples with both MAF mutation records and Groupfile survival labels, confirming a correct merge on Tumor_Sample_Barcode

## Limitations

- Due to the inconsistent sequencing background of the data, only coding regions were uniformly analyzed, which introduces limitations
- Complete clinical information including treatment methods, medication information, and extended follow-up time is lacking, which limits clinical interpretation
- The number of individuals studied in the paper is relatively small, which is one of the shortcomings of model learning

## Evidence

- [intro] used the R language to design a simple and convenient package RNMF: "used the R language to design a simple and convenient package RNMF"
- [methods] somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file: "somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file"
- [readme] sample.id = 'Tumor_Sample_Barcode', ... Hugo = 'Hugo_Symbol': "sample.id = 'Tumor_Sample_Barcode', chr = 'Chromosome', pos = 'Start_position', ref = 'Reference_Allele', alt = 'Tumor_Seq_Allele2', Hugo = 'Hugo_Symbol'"
- [discussion] Due to the inconsistent sequencing background of the data, we uniformly analyzed the data of coding regions: "Due to the inconsistent sequencing background of the data, we uniformly analyzed the data of coding regions"
