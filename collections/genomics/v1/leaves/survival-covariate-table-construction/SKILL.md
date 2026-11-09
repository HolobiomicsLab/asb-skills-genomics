---
name: survival-covariate-table-construction
description: Use when a per-sample binary mutation status (e.g. TP53 R248Q from a
  MAF) must be combined with survival group annotations (e.g. G2/G4) into one covariate
  table for multivariate Cox regression of prognosis.
when_to_use_negative:
- When only aggregate gene-by-signature association statistics (samFisherSigs/samPerMutSigs
  outputs with Mutfreq=0.04, threshold=0.06, Qvalue=0.1) are needed rather than per-sample
  prognostic modeling
- When no survival/group annotation exists for the samples, since the Cox covariates
  require the Groupfile labels
- When the question is de novo mutational signature extraction (denovoNMF) rather
  than testing a specific mutation's prognostic independence
tools:
- name: RNMF
  role: Source of the example maffile and Groupfile and of per-sample mutation extraction
    keyed on Tumor_Sample_Barcode / Hugo_Symbol
  repo: https://github.com/zhenzhang-li/RNMF
- name: R survival package
  role: Fits the multivariate Cox proportional hazards model on the merged covariate
    table (version 2.40–1)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:22792419b9165037be5b2d673674bc7e77c501647cf87e6cbb0e4de09e69dc70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - R
  - R survival package
  - RNMF
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# survival-covariate-table-construction

## Summary

Constructs a per-sample covariate table that merges a gene hotspot mutation indicator (e.g. TP53 R248Q extracted from an RNMF MAF file keyed on Tumor_Sample_Barcode/Hugo_Symbol) with survival group labels from a Groupfile, so a multivariate Cox proportional hazards model can test whether the mutation is an independent prognosticator.

## When to use

Use when a per-sample binary mutation status (e.g. TP53 R248Q from a MAF) must be combined with survival group annotations (e.g. G2/G4) into one covariate table for multivariate Cox regression of prognosis.

## When NOT to use

- When only aggregate gene-by-signature association statistics (samFisherSigs/samPerMutSigs outputs with Mutfreq=0.04, threshold=0.06, Qvalue=0.1) are needed rather than per-sample prognostic modeling
- When no survival/group annotation exists for the samples, since the Cox covariates require the Groupfile labels
- When the question is de novo mutational signature extraction (denovoNMF) rather than testing a specific mutation's prognostic independence

## Inputs

- MAF file of nonsilent somatic mutations (e.g. maffile = 'Regular.ESCC.exon.WES.maf-input-nonsilent.txt')
- Groupfile with per-sample survival group labels (Groupfile.txt, G2/G4)
- Column key definitions: sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol'

## Outputs

- Per-sample covariate table merging TP53 R248Q indicator with survival group label
- cox_regression_results.csv with coefficients, hazard ratios, confidence intervals, and p-values per covariate

## How to apply

Load the example MAF file (maffile, e.g. 'Regular.ESCC.exon.WES.maf-input-nonsilent.txt') and Groupfile (Groupfile.txt) from the RNMF package, keyed on sample.id = 'Tumor_Sample_Barcode' and Hugo = 'Hugo_Symbol', and extract per-sample TP53 hotspot mutation status including R248Q. Merge the R248Q mutation indicator with the survival group labels (G2/G4) into a single per-sample covariate table. Fit a multivariate Cox proportional hazards model with the R survival package (2.40–1) using R248Q status and group annotation as covariates. Extract regression coefficients, hazard ratios with confidence intervals, and p-values per covariate, and write them to cox_regression_results.csv. Decision point: the mutation qualifies as an independent prognosticator when its HR > 1 (poorer prognosis) with p < 0.05.

## Related tools

- **RNMF** (Source of the example maffile and Groupfile and of per-sample mutation extraction keyed on Tumor_Sample_Barcode / Hugo_Symbol) — https://github.com/zhenzhang-li/RNMF
- **R survival package** (Fits the multivariate Cox proportional hazards model on the merged covariate table (version 2.40–1))

## Examples

```
cov <- merge(tp53_r248q_status, read.delim('Groupfile.txt'), by.x='Tumor_Sample_Barcode', by.y='sample.id'); fit <- coxph(Surv(time, status) ~ R248Q + group, data=cov); write.csv(summary(fit)$coefficients, 'cox_regression_results.csv')
```

## Evaluation signals

- The covariate table has exactly one row per unique Tumor_Sample_Barcode present in both the MAF and the Groupfile
- The R248Q column is a per-sample binary indicator derived from Hugo_Symbol == 'TP53' rows in the maffile
- cox_regression_results.csv contains coefficient, HR, confidence interval, and p-value for each covariate (R248Q status and group)
- Validation passes if the R248Q hazard ratio indicates poorer prognosis (HR > 1) with p-value below the significance threshold of 0.05

## Limitations

- Relatively small number of individuals studied is a shortcoming for model learning
- Incomplete clinical information (treatment methods, medication information, extended follow-up time) limits further prognostic exploration
- Due to inconsistent sequencing background of the data, only coding regions were uniformly analyzed, which introduces limitations

## Evidence

- [methods] somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file: "somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file"
- [readme] maffile and Groupfile column keys in README: "maffile = 'Regular.ESCC.exon.WES.maf-input-nonsilent.txt'

Groupfile = 'Groupfile.txt'"
- [readme] README keying of MAF columns: "sample.id = 'Tumor_Sample_Barcode', chr = 'Chromosome', pos = 'Start_position', ref = 'Reference_Allele', alt = 'Tumor_Seq_Allele2', Hugo = 'Hugo_Symbol'"
- [results] task finding on TP53 R248Q: [evidence span withheld — anchor status: not_found]
