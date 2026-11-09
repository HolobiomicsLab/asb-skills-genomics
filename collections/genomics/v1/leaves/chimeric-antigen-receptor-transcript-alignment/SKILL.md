---
name: chimeric-antigen-receptor-transcript-alignment
description: Use when scRNA-seq libraries incompletely capture CAR transgenes and
  you need per-cell CAR vs wildtype classification from targeted long-read amplicon
  FASTQ (e.g. ONT reads of 10x 5' cDNA).
when_to_use_negative:
- When the question requires full-transcriptome gene expression rather than targeted
  detection of a specific transgene/isoform — nanoranger is a targeted, limited-primer
  approach
- When the target locus is far from the 5' end, lowly expressed, or longer than 4
  kB — these are suboptimal nanoranger targets
- When input is bulk (non-single-cell) sequencing without 10x cell barcodes and UMIs,
  since per-cell classification depends on barcode/UMI extraction
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: nanoranger
  role: Aligns targeted long reads to the CAR_CD28 reference, extracts 10x cell barcodes/UMIs,
    and assigns reads to CAR or wildtype CD28 transcript classes in 5p10XGEX mode
  repo: https://github.com/mehdiborji/nanoranger
- name: nanoranger.R
  role: 'Downstream R processing of nanoranger output: UMI collapsing with starcode
    (Levenshtein distance 3), knee plot cell calling, CAR sequence extraction from
    amplicons'
  repo: https://github.com/liviuspenter/nanoranger.R
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform used to sequence targeted amplicons from the
    10x 5'-anchored cDNA library
- name: 10x Genomics
  role: Source of the 5'-anchored whole-transcriptome scRNA-seq cDNA library from
    which CAR amplicons are amplified
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:f64d0250bb803660f3debcb8591a7bcffe8a8545c3615794ce51b03412e9c0fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - 10x Genomics
  - Oxford Nanopore Technologies (ONT)
  - nanoranger
  - nanoranger.R
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# chimeric-antigen-receptor-transcript-alignment

## Summary

Targeted long-read detection and quantification of CD19 CAR transcripts from 10x Genomics 5'-anchored single-cell cDNA libraries using nanoranger, distinguishing the chimeric construct from wildtype CD28 transcripts via a CD28-specific primer and a combined CAR_CD28 reference.

## When to use

Use when scRNA-seq libraries incompletely capture CAR transgenes and you need per-cell CAR vs wildtype classification from targeted long-read amplicon FASTQ (e.g. ONT reads of 10x 5' cDNA).

## When NOT to use

- When the question requires full-transcriptome gene expression rather than targeted detection of a specific transgene/isoform — nanoranger is a targeted, limited-primer approach
- When the target locus is far from the 5' end, lowly expressed, or longer than 4 kB — these are suboptimal nanoranger targets
- When input is bulk (non-single-cell) sequencing without 10x cell barcodes and UMIs, since per-cell classification depends on barcode/UMI extraction

## Inputs

- Targeted amplicon FASTQ from long-read sequencing of an intermediate 10x Genomics 5'-anchored whole-transcriptome cDNA library (e.g. 97_6_CAR.fastq.gz)
- CAR_CD28 combined reference (chimeric antigen receptor construct plus endogenous CD28 locus)
- 10x Genomics cell barcode structure (CB/UMI in reads)

## Outputs

- Per-cell CAR vs wildtype CD28 classification table (car_wt_quantification.csv)
- Overall fraction of CAR+ cells per sample
- UMI-collapsed per-cell barcode counts per transcript class

## How to apply

Sequence targeted amplicons (e.g. sample 97_6_CAR.fastq.gz) amplified with a CD28-specific primer from an intermediate 10x Genomics 5'-anchored whole-transcriptome cDNA library on an ONT device. Run nanoranger in 5p10XGEX mode with a CAR_CD28 reference so reads align to both the chimeric antigen receptor construct and the endogenous CD28 locus. nanoranger extracts cell barcodes and UMIs from the long reads and assigns each read to the CAR or wildtype CD28 transcript class; downstream (nanoranger.R) collapses UMIs with starcode within a Levenshtein distance of 3 since 10x UMIs lack a whitelist. Collapse UMI counts per cell barcode, classify each cell as CAR+ or wildtype CD28 by dominant transcript class, and export the per-cell table (car_wt_quantification.csv). Targeting works best for loci near the 5' end of highly expressed genes with transcripts shorter than 4 kB, and enrichment prior to long-read sequencing augments detection of structural transcriptomic variants like transgenes.

## Related tools

- **nanoranger** (Aligns targeted long reads to the CAR_CD28 reference, extracts 10x cell barcodes/UMIs, and assigns reads to CAR or wildtype CD28 transcript classes in 5p10XGEX mode) — https://github.com/mehdiborji/nanoranger
- **nanoranger.R** (Downstream R processing of nanoranger output: UMI collapsing with starcode (Levenshtein distance 3), knee plot cell calling, CAR sequence extraction from amplicons) — https://github.com/liviuspenter/nanoranger.R
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform used to sequence targeted amplicons from the 10x 5'-anchored cDNA library)
- **10x Genomics** (Source of the 5'-anchored whole-transcriptome scRNA-seq cDNA library from which CAR amplicons are amplified)

## Examples

```
nanoranger 5p10XGEX --fastq 97_6_CAR.fastq.gz --ref CAR_CD28.fa --out car_wt_quantification
```

## Evaluation signals

- Computed fraction of CAR+ cells exceeds 0.75, consistent with the reported >75% CAR+ T cell detection
- Targeted nanoranger recovers almost all CAR+ T cells found by unenriched Illumina scRNA-seq plus an additional ~15%
- Cells classify consistently as CAR+ or wildtype CD28 based on the dominant transcript class after UMI collapsing
- Reads align unambiguously to either the CAR construct or the endogenous CD28 locus in the combined CAR_CD28 reference
- Cell barcode counts survive the expected knee-plot cell-calling cutoff in nanoranger.R

## Limitations

- Standard short-read scRNA-seq libraries incompletely capture CAR transcripts, so Illumina-based detection is an underestimate rather than ground truth
- Nanoranger target detection depends on low, incomplete transcript representation; the genotyping ceiling cannot be overcome merely by amplicon optimization or deeper sequencing
- UMIs in 10x libraries lack a whitelist, so sequencing-error-driven UMI inflation must be corrected by starcode collapsing at Levenshtein distance 3

## Evidence

- [results] Targeted nanoranger detection identified almost all CAR+ T cells found by unenriched Illumina scRNA-seq plus an additional 15% (workflow card finding): [evidence span withheld — anchor status: not_found]
- [abstract] Nanoranger detects chimeric antigen receptor sequences from intermediate single-cell cDNA libraries: "starts from intermediate single-cell cDNA libraries to detect cell lineage-defining features, including single-nucleotide variants, fusion genes, isoforms, sequences of chimeric antigen and TCRs"
- [methods] Methods: input data from 10x cDNA libraries or targeted amplicons sequenced on ONT/PacBio devices: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification"
- [discussion] Enrichment for genes of interest prior to long-read sequencing augments detection of structural variants: "enriching for genes of interest augments the inherent ability of long-read sequencing to detect structural transcriptomic variants"
- [readme] README: nanoranger.R collapses UMIs within Levenshtein distance 3 using starcode because 10x UMIs have no whitelist: "By using [starcode](https://github.com/gui11aume/starcode), UMIs that arise due to sequencing error are collapsed
in groups within a Levenshtein distance of 3."
- [abstract] Optimal nanoranger targets are 5'-proximal loci of highly expressed genes with transcripts shorter than 4 kB: "we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB"
