---
name: inverse-nmf-deconvolution
description: Use when a non-negative mutation count matrix (SBS/DBS/ID from a VCF
  or MAF file) must be deconvolved against a FIXED reference signature profile to
  get per-sample contributions, and per-gene or per-mutation-group contribution differences
  are the endpoint.
when_to_use_negative:
- When signatures must be discovered de novo rather than tested against a fixed reference
  profile — use RNMF's denovoNMF 5-step framework instead
- When the goal is whole-cohort signature number selection (K) with silhouette coefficient
  and error gradient rather than testing a known signature's association with gene
  mutation status
tools:
- name: RNMF
  role: Provides InverseNMF for estimating S given fixed P, and samPerMutSigs for
    per-sample/per-mutation-group contribution statistics from the MAF file
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Language in which the RNMF package and its InverseNMF/samPerMutSigs interfaces
    are implemented and invoked
  repo: https://github.com/zhenzhang-li/RNMF
- name: Bioconductor (BSgenome.Hsapiens.UCSC.hg19/hg38)
  role: Genome reference libraries used by RNMF to build the SBS/DBS/ID mutation-type
    count matrix from MAF coordinates
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:4c80c4a4e0bd775dd108d7dcbbacc74daae22d4cfebc352eb1a9b0ba40285b45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - Bioconductor
  - Bioconductor (BSgenome.Hsapiens.UCSC.hg19/hg38)
  - R
  - RNMF
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# inverse-nmf-deconvolution

## Summary

Estimate the sample contribution matrix S for a fixed, user-defined mutational signature matrix P (e.g. COSMIC v3.2 SBS1* age signature) using the InverseNMF routine of the RNMF R package, then test whether gene mutation status (e.g. TP53 R342*) shifts per-sample signature contribution.

## When to use

Use when a non-negative mutation count matrix (SBS/DBS/ID from a VCF or MAF file) must be deconvolved against a FIXED reference signature profile to get per-sample contributions, and per-gene or per-mutation-group contribution differences are the endpoint.

## When NOT to use

- When signatures must be discovered de novo rather than tested against a fixed reference profile — use RNMF's denovoNMF 5-step framework instead
- When the goal is whole-cohort signature number selection (K) with silhouette coefficient and error gradient rather than testing a known signature's association with gene mutation status

## Inputs

- Somatic mutation MAF file (e.g. EXON1073 ESCC exon-sequenced cohort, 1073 samples, columns Tumor_Sample_Barcode, Hugo_Symbol, Chromosome, Start_position, Reference_Allele, Tumor_Seq_Allele2)
- Fixed reference signature matrix P (COSMIC Mutational Signatures v3.2, March 2021, e.g. SBS1* age signature profile)
- Genome build library for tri-nucleotide context (BSgenome.Hsapiens.UCSC.hg19 or hg38)
- Group/annotation file mapping samples to gene-mutation classes (e.g. TP53 R342* / other-TP53-mutant / wild-type)

## Outputs

- Per-sample contribution matrix S (SBS.Smatrix.txt) for the fixed signature
- Per-sample and per-group median signature contribution table with group annotations (e.g. sbs1_tp53_group_contributions.csv)
- Median contribution differences between mutation-status groups (e.g. 0.388 R342* vs 0.305 other vs 0.276 wild-type)

## How to apply

Import somatic variants from a MAF (or VCF) file with RNMF's SigsInput, joining on 'Tumor_Sample_Barcode', 'Chromosome', 'Start_position', 'Reference_Allele', 'Tumor_Seq_Allele2' with Hugo_Symbol. Fix P to the COSMIC v3.2 SBS1* (age) signature profile and run InverseNMF to iteratively estimate S via the multiplicative update S' = S P^T V / (P^T P S + γ), iterating until convergence. Then call samPerMutSigs with Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1 to compute per-sample and per-group median contributions for samples stratified by the gene status of interest (e.g. TP53 R342*-mutated vs other-TP53-mutant vs wild-type). Compare group medians; in the EXON1073 ESCC cohort this reproduced a median SBS1* increase of 0.388 for R342* vs 0.276 for wild-type, the correctness signal for this skill.

## Related tools

- **RNMF** (Provides InverseNMF for estimating S given fixed P, and samPerMutSigs for per-sample/per-mutation-group contribution statistics from the MAF file) — https://github.com/zhenzhang-li/RNMF
- **R** (Language in which the RNMF package and its InverseNMF/samPerMutSigs interfaces are implemented and invoked) — https://github.com/zhenzhang-li/RNMF
- **Bioconductor (BSgenome.Hsapiens.UCSC.hg19/hg38)** (Genome reference libraries used by RNMF to build the SBS/DBS/ID mutation-type count matrix from MAF coordinates)

## Examples

```
samPerMutSigs(Sfile = Sfile, choose.Sigs = 'SBS1', file = maffile, sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1, plot = TRUE)
```

## Evaluation signals

- Computed median SBS1* contribution increase for the TP53 R342* group versus wild-type reproduces the reported value of 0.388 (R342*) vs 0.276 (wild-type)
- InverseNMF iteration converges (or reaches its maximum iteration count) with non-negative S entries
- Per-sample contribution table carries group annotations for all three strata (R342*, other-TP53-mutant, wild-type) across the 1073-sample EXON1073 cohort
- samPerMutSigs runs with the documented parameters Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1 without subsetting/filter errors on the MAF

## Limitations

- Due to inconsistent sequencing background of the pooled data, only coding regions were uniformly analyzed, which introduces limitations
- The framework spends more time on de novo extraction because of the 'Re-updated the initial value' and 'build a solution space' correction processes
- No software version numbers or installation details are stated in the paper for the framework

## Evidence

- [methods] we also developed a reversible method named InverseNMF, which can identify mutational signatures within a small data set or a single tumor sample: "we also developed a reversible method named InverseNMF, which can identify mutational signatures within a small data set or a single tumor sample"
- [abstract] we found that age signature is closely related to the frequent mutation of TP53, of which R342* is highlighted due to strongly linked to age signature: "we found that age signature is closely related to the frequent mutation of TP53, of which R342* is highlighted due to strongly linked to age signature"
- [readme] samPerMutSigs(Sfile = Sfile, choose.Sigs = isig,
                file = maffile, sample.id = 'Tumor_Sample_Barcode',: "samPerMutSigs(Sfile = Sfile, choose.Sigs = isig,
                file = maffile, sample.id = 'Tumor_Sample_Barcode',"
- [methods] Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1: "Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1"
