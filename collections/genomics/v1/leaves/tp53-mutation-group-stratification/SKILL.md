---
name: tp53-mutation-group-stratification
description: Use when MAF-format somatic mutation data with fixed-signature sample
  contributions (e.g. SBS1* from InverseNMF) must be grouped by gene hotspot status
  such as TP53 R342* to test mutation-specific signature associations.
when_to_use_negative:
- When the aim is de novo extraction of mutational signatures rather than testing
  a known signature (SBS1*) against predefined gene-mutation groups — use denovoNMF
  instead of InverseNMF-based grouping
- When the cohort lacks sufficient TP53 R342* mutated samples to compute a stable
  group median, since the contrast depends on per-group medians
- When mutations of interest are not representable in the MAF fields joined on Tumor_Sample_Barcode
  and Hugo_Symbol (e.g. non-coding or structural variants)
edam_operation: http://edamontology.org/operation_3793
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0199
tools:
- name: RNMF
  role: R package providing MAF import, InverseNMF fixed-signature contribution estimation,
    and samPerMutSigs-style per-mutation-group contribution statistics
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Language in which the RNMF pipeline and group-median comparisons are executed
  repo: https://github.com/zhenzhang-li/RNMF
- name: Bioconductor
  role: Genome-build dependency libraries (e.g. BSgenome.Hsapiens.UCSC.hg19/hg38)
    supporting mutation-type matrix construction from MAF coordinates
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:56ac663222ac20997cdf6383fe84bd5239b8cc2a6eaf54c2a5c810964700cd94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - Bioconductor
  - R
  - RNMF
  edam_operation: http://edamontology.org/operation_3793
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# tp53-mutation-group-stratification

## Summary

Stratify ESCC exon-sequenced samples into TP53 R342*-mutated, other-TP53-mutant, and TP53 wild-type groups, then compare per-sample SBS1* (age signature) contribution medians to test whether the R342* hotspot is specifically linked to age-signature activity.

## When to use

Use when MAF-format somatic mutation data with fixed-signature sample contributions (e.g. SBS1* from InverseNMF) must be grouped by gene hotspot status such as TP53 R342* to test mutation-specific signature associations.

## When NOT to use

- When the aim is de novo extraction of mutational signatures rather than testing a known signature (SBS1*) against predefined gene-mutation groups — use denovoNMF instead of InverseNMF-based grouping
- When the cohort lacks sufficient TP53 R342* mutated samples to compute a stable group median, since the contrast depends on per-group medians
- When mutations of interest are not representable in the MAF fields joined on Tumor_Sample_Barcode and Hugo_Symbol (e.g. non-coding or structural variants)

## Inputs

- Somatic mutation data in MAF format (e.g. EXON1073 ESCC exon-sequenced cohort) with Tumor_Sample_Barcode, Hugo_Symbol, and TP53 protein-change annotations
- COSMIC Mutational Signatures v3.2 (March 2021) SBS1* reference profile used as fixed P
- Per-sample contribution matrix S estimated by InverseNMF (SBS.Smatrix.txt)
- Group annotation file mapping samples to TP53 status (R342*, other-TP53-mutant, wild-type)

## Outputs

- Per-sample SBS1* contribution table annotated with TP53 group membership
- Group-level median SBS1* contribution statistics (e.g. sbs1_tp53_group_contributions.csv) with median differences R342* vs other mutations (0.388 vs 0.305) vs wild-type (0.276)

## How to apply

Load somatic mutation records from a MAF file with RNMF, joining on 'Tumor_Sample_Barcode' and 'Hugo_Symbol'. Fix the signature matrix P to the COSMIC v3.2 SBS1* age-signature profile and use InverseNMF with the multiplicative update S' = S P^T V / (P^T P S + γ) to estimate per-sample contributions S. Partition the EXON1073 samples into three TP53 groups (R342*-mutated, other-TP53-mutant, wild-type) and compute per-group median SBS1* contributions via the samPerMutSigs-style interface (choose.Sigs = SBS1*, Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1). The rationale is that a hotspot-specific elevation in age-signature contribution reveals which gene mutation drives the mutational process rather than the gene as a whole. Write a per-sample contribution table with group annotations and summary statistics, and validate by checking that the R342* vs wild-type median increase reproduces the reported 0.388.

## Related tools

- **RNMF** (R package providing MAF import, InverseNMF fixed-signature contribution estimation, and samPerMutSigs-style per-mutation-group contribution statistics) — https://github.com/zhenzhang-li/RNMF
- **R** (Language in which the RNMF pipeline and group-median comparisons are executed) — https://github.com/zhenzhang-li/RNMF
- **Bioconductor** (Genome-build dependency libraries (e.g. BSgenome.Hsapiens.UCSC.hg19/hg38) supporting mutation-type matrix construction from MAF coordinates)

## Examples

```
samPerMutSigs(Sfile = Sfile, choose.Sigs = 'SBS1', file = maffile, sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1, plot = TRUE)
```

## Evaluation signals

- Computed median SBS1* contribution increase for the TP53 R342* group versus wild-type reproduces the reported 0.388 (R342* 0.388 vs other mutations 0.305 vs wild-type 0.276)
- All 1073 EXON1073 exon-sequenced samples are assigned to exactly one of the three TP53 groups with no missing Hugo_Symbol joins
- Per-sample S values come from a converged InverseNMF run with P fixed to the COSMIC v3.2 SBS1* profile (no drift of the signature during updates)
- samPerMutSigs parameters match the article settings: Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1
- Output CSV contains per-sample contributions plus group annotations and group summary statistics (sbs1_tp53_group_contributions.csv)

## Limitations

- Analysis was limited to coding regions because of the inconsistent sequencing background of the pooled data, which introduces constraints on variant context
- The framework spends more time on de novo signature extraction than previously published software due to the initial-value correction and solution-space processes, needing future optimization
- Only the age signature SBS1* association was validated for R342*; the mechanism is correlational and other TP53 hotspots may behave differently in other cohorts

## Evidence

- [methods] InverseNMF estimation of sample contribution matrix S from user-defined P: "we also developed a reversible method named InverseNMF, which can identify mutational signatures within a small data set or a single tumor sample"
- [abstract] Age signature linked to TP53 R342*: "we found that age signature is closely related to the frequent mutation of TP53, of which R342* is highlighted due to strongly linked to age signature"
- [methods] samPerMutSigs interface parameters: "Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1"
- [readme] MAF import and join keys in README example: "sample.id = 'Tumor_Sample_Barcode', chr = 'Chromosome', pos = 'Start_position', ref = 'Reference_Allele', alt = 'Tumor_Seq_Allele2'"
- [methods] EXON1073 dataset naming: "exon region of 1073 samples renames as EXON1073"
