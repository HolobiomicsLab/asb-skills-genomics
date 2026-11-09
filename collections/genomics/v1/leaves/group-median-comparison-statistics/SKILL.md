---
name: group-median-comparison-statistics
description: Use when samples can be stratified by a specific gene mutation or wild-type
  status and you need to test whether a mutational signature's sample-contribution
  matrix S differs by group using median comparisons.
when_to_use_negative:
- When the goal is de novo mutational signature extraction rather than testing a known/fixed
  signature against a predefined sample grouping (use denovoNMF instead)
- When no biological grouping variable (e.g., a specific hotspot mutation vs wild-type
  status) can be defined for the samples, so group median comparison is uninformative
- When the dataset is not exon/coding-region mutation data consistent with the coding-region-only
  analysis background used in the article
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0203
tools:
- name: RNMF
  role: R package providing InverseNMF to fix signature matrix P and estimate sample
    contributions S, and samPerMutSigs to compute per-sample signature contributions
    filtered by Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1 for group median comparison
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Language in which RNMF is implemented and the stratification, per-group median
    computation, and median-difference statistics are scripted
- name: Bioconductor
  role: Dependency layer supplying genome libraries (e.g., BSgenome.Hsapiens.UCSC.hg19/hg38)
    needed by RNMF when importing MAF mutation records
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:594592c6c4101f8b386d553520233d0abe982e4dc213807c06ffb6c273ba1761
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - Bioconductor
  - R
  - RNMF
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# group-median-comparison-statistics

## Summary

Stratify samples by mutation status (e.g., TP53 R342* vs other TP53-mutant vs TP53 wild-type in the EXON1073 ESCC cohort), compute per-group median sample contributions to a fixed mutational signature (SBS1*) via RNMF's samPerMutSigs interface, and compare medians to attribute signature activity to a specific mutation.

## When to use

Use when samples can be stratified by a specific gene mutation or wild-type status and you need to test whether a mutational signature's sample-contribution matrix S differs by group using median comparisons.

## When NOT to use

- When the goal is de novo mutational signature extraction rather than testing a known/fixed signature against a predefined sample grouping (use denovoNMF instead)
- When no biological grouping variable (e.g., a specific hotspot mutation vs wild-type status) can be defined for the samples, so group median comparison is uninformative
- When the dataset is not exon/coding-region mutation data consistent with the coding-region-only analysis background used in the article

## Inputs

- MAF-formatted somatic mutation file for the 1073-sample Asian ESCC exon cohort (EXON1073), parsed with RNMF using Tumor_Sample_Barcode and Hugo_Symbol
- User-defined COSMIC v3.2 SBS1* (age) signature matrix P (fixed profile)
- Sample contribution matrix Sfile produced by InverseNMF
- Group annotation file assigning samples to R342*, other-TP53-mutant, and TP53 wild-type strata

## Outputs

- Per-sample SBS1* contribution table annotated with TP53 group membership
- Group-level median SBS1* contribution statistics (e.g., R342* = 0.388, other TP53 mutations = 0.305, wild-type = 0.276)
- sbs1_tp53_group_contributions.csv with per-sample contributions and group summary statistics

## How to apply

First estimate per-sample signature contributions by fixing the signature matrix P to the user-defined COSMIC v3.2 SBS1* (age) profile and running InverseNMF to obtain the abundance matrix S. Then use RNMF's samPerMutSigs interface with Mutfreq = 0.04, threshold = 0.06, and Qvalue = 0.1 to compute per-sample SBS1* contributions for each sample in the EXON1073 cohort. Stratify the 1073 samples into R342*-mutated, other-TP53-mutant, and TP53 wild-type groups, join mutation records on 'Tumor_Sample_Barcode' and 'Hugo_Symbol', and compute the group median SBS1* contribution for each stratum. The decision point is the median difference between the focal-mutation group (R342*, median 0.388) and comparison groups (other mutations 0.305, wild-type 0.276), which attributes age-signature activity specifically to the R342* hotspot.

## Related tools

- **RNMF** (R package providing InverseNMF to fix signature matrix P and estimate sample contributions S, and samPerMutSigs to compute per-sample signature contributions filtered by Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1 for group median comparison) — https://github.com/zhenzhang-li/RNMF
- **R** (Language in which RNMF is implemented and the stratification, per-group median computation, and median-difference statistics are scripted)
- **Bioconductor** (Dependency layer supplying genome libraries (e.g., BSgenome.Hsapiens.UCSC.hg19/hg38) needed by RNMF when importing MAF mutation records)

## Examples

```
samPerMutSigs(Sfile = Sfile, choose.Sigs = 'SBS1', file = maffile, sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1, plot = TRUE)
```

## Evaluation signals

- The median SBS1* contribution increase for the TP53 R342* group versus wild-type should reproduce the reported value of 0.388
- The ordering of group medians should hold: R342* (0.388) > other TP53 mutations (0.305) > wild-type (0.276)
- The samPerMutSigs output must reflect the stated filter parameters Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1
- Per-sample contribution table should cover all 1073 EXON1073 samples joined on Tumor_Sample_Barcode and Hugo_Symbol, and group labels should partition samples exactly into R342*, other-TP53-mutant, and wild-type
- Written CSV (sbs1_tp53_group_contributions.csv) should contain both per-sample annotations and group summary statistics without missing medians

## Limitations

- The framework is slower than previously published software for de novo signature extraction, due to the 'Re-updated the initial value' correction process and 'build a solution space' process; needs future optimization
- Analysis limited by inconsistent sequencing background of data; only coding regions uniformly analyzed, which introduces limitations
- Relatively small number of individuals studied, a shortcoming for model learning

## Evidence

- [abstract] we found that age signature is closely related to the frequent mutation of TP53, of which R342* is highlighted due to strongly linked to age signature: "we found that age signature is closely related to the frequent mutation of TP53, of which R342* is highlighted due to strongly linked to age signature"
- [methods] we also developed a reversible method named InverseNMF, which can identify mutational signatures within a small data set or a single tumor sample: "we also developed a reversible method named InverseNMF, which can identify mutational signatures within a small data set or a single tumor sample"
- [readme] samPerMutSigs(Sfile = Sfile, choose.Sigs = isig, file = maffile: "samPerMutSigs(Sfile = Sfile, choose.Sigs = isig, 
                file = maffile, sample.id = 'Tumor_Sample_Barcode',
                Hugo = 'Hugo_Symbol', Mutfreq = 0.04, 
                threshold"
