---
name: mutational-signature-contribution-estimation
description: Use when you have somatic mutation data (VCF/MAF, e.g., EXON1073 ESCC
  cohort) and want per-sample contributions to a predefined COSMIC signature, to test
  whether a specific mutation (e.g., TP53 R342*) elevates that signature.
when_to_use_negative:
- When signatures are unknown and must be extracted de novo from the mutation count
  matrix — use the denovoNMF 5-step framework (Dirichlet resampling, k-means initialization,
  silhouette-based K selection) instead of fixing P and running InverseNMF.
- When the input is already a normalized signature contribution/abundance matrix rather
  than raw per-sample somatic mutation records (MAF/VCF).
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0199
tools:
- name: RNMF
  role: R package providing InverseNMF to estimate sample contribution matrix S from
    a user-defined signature matrix P, and samPerMutSigs to compute per-sample/per-group
    median contributions from a MAF file
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Language in which the RNMF package is implemented and the group-comparison
    script is run
  repo: https://github.com/zhenzhang-li/RNMF
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor genome build used by RNMF to construct the SBS mutation-type
    count matrix from mutation positions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:c3c9f211f5a49fd8b69c416b9710bb54d79c32f6a48c5e4c8802135331b2426a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - Bioconductor
  - R
  - RNMF
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mutational-signature-contribution-estimation

## Summary

Estimate per-sample contributions to a fixed, user-defined mutational signature (e.g., COSMIC SBS1*, the age signature) using the RNMF InverseNMF approach, then stratify samples (e.g., by TP53 R342* status) and compare group median contributions to link a gene mutation to signature activity.

## When to use

Use when you have somatic mutation data (VCF/MAF, e.g., EXON1073 ESCC cohort) and want per-sample contributions to a predefined COSMIC signature, to test whether a specific mutation (e.g., TP53 R342*) elevates that signature.

## When NOT to use

- When signatures are unknown and must be extracted de novo from the mutation count matrix — use the denovoNMF 5-step framework (Dirichlet resampling, k-means initialization, silhouette-based K selection) instead of fixing P and running InverseNMF.
- When the input is already a normalized signature contribution/abundance matrix rather than raw per-sample somatic mutation records (MAF/VCF).

## Inputs

- Somatic mutation MAF or VCF file (e.g., EXON1073 ESCC exon-sequenced cohort)
- Fixed signature matrix P from COSMIC Mutational Signatures v3.2 (March 2021), e.g., SBS1* age signature profile
- Sample contribution matrix Sfile from InverseNMF (SBS.Smatrix.txt)
- Group annotation file assigning samples to mutation-status strata (e.g., TP53 R342* vs other vs wild-type)

## Outputs

- Per-sample signature contribution table with group annotations (e.g., sbs1_tp53_group_contributions.csv)
- Per-group median contribution statistics and median difference (e.g., 0.388 for TP53 R342* vs wild-type SBS1* contribution)
- samPerMutSigs plots of per-mutation-type signature contribution

## How to apply

Load the somatic mutation records from a MAF file with the RNMF R package, joining on 'Tumor_Sample_Barcode' and 'Hugo_Symbol'. Fix the signature matrix P to the user-defined COSMIC v3.2 profile (e.g., SBS1*, the age signature) and run InverseNMF to iteratively estimate the sample contribution matrix S via multiplicative updates S' = S P^T V / (P^T P S + gamma), iterating until convergence. Stratify samples into groups (e.g., TP53 R342*-mutated, other-TP53-mutant, TP53 wild-type) and compute per-sample and per-group median SBS1* contributions with the samPerMutSigs-style interface using Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1. Rationale: the CCA-based InverseNMF approach identifies signature contributions within a single tumor sample or small dataset, enabling direct association of driver mutations (like TP53 R342*) with signature activity; InverseNMF complements denovoNMF, which is for de novo signature extraction.

## Related tools

- **RNMF** (R package providing InverseNMF to estimate sample contribution matrix S from a user-defined signature matrix P, and samPerMutSigs to compute per-sample/per-group median contributions from a MAF file) — https://github.com/zhenzhang-li/RNMF
- **R** (Language in which the RNMF package is implemented and the group-comparison script is run) — https://github.com/zhenzhang-li/RNMF
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor genome build used by RNMF to construct the SBS mutation-type count matrix from mutation positions)

## Examples

```
samPerMutSigs(Sfile = Sfile, choose.Sigs = 'SBS1', file = maffile, sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1, plot = TRUE)
```

## Evaluation signals

- Reproduce the reported median SBS1* contribution increase for the TP53 R342* group versus wild-type of 0.388 (R342*) vs 0.305 (other TP53 mutations) vs 0.276 (wild-type) in the EXON1073 cohort.
- InverseNMF iterations converge (or reach the iteration cap) and the estimated S matrix columns are non-negative and normalized per sample.
- Per-sample contribution table contains all 1073 EXON1073 samples with group annotations covering the three TP53 strata (R342*, other-TP53-mutant, wild-type).
- samPerMutSigs runs accept the fixed filter parameters (Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1) without errors on the nonsilent MAF input.
- Cosine similarity between the fixed SBS1* profile used and the COSMIC v3.2 reference signature remains ~1 (signature P was not re-estimated).

## Limitations

- Analysis is limited by the inconsistent sequencing background of the pooled data; only coding regions were uniformly analyzed (EXON1073), which introduces limitations.
- The framework spends more time on de novo extraction due to the 'Re-updated the initial value' correction and 'build a solution space' processes and needs future optimization (less relevant when P is fixed, but affects the RNMF pipeline overall).
- The mechanism linking a mutation to a signature (e.g., TP53 R342* and the age signature) is correlational and needs verification by subsequent experiments.

## Evidence

- [methods] we also developed a reversible method named InverseNMF, which can identify mutational signatures within a small data set or a single tumor sample: "we also developed a reversible method named InverseNMF, which can identify mutational signatures within a small data set or a single tumor sample"
- [methods] somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file: "somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file"
- [methods] Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1: "Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1"
- [abstract] age signature is closely related to the frequent mutation of TP53, of which R342* is highlighted due to strongly linked to age signature: "age signature is closely related to the frequent mutation of TP53, of which R342* is highlighted due to strongly linked to age signature"
- [readme] samPerMutSigs(Sfile = Sfile, choose.Sigs = isig,
                file = maffile: "samPerMutSigs(Sfile = Sfile, choose.Sigs = isig,
                file = maffile, sample.id = 'Tumor_Sample_Barcode',"
