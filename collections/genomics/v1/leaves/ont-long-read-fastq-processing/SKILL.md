---
name: ont-long-read-fastq-processing
description: Use when FASTQ files from ONT sequencing of 10x Genomics-derived amplicons
  or cDNA libraries must be converted into per-cell transcript classifications (e.g.
  CAR+ vs wildtype CD28) that short-read scRNA-seq fails to fully resolve.
when_to_use_negative:
- The input is short-read (Illumina) scRNA-seq data, which the paper notes is suboptimal
  for long fragments and cannot easily resolve structural variants such as transgenes
  like CARs
- The analysis goal is whole-genome or unrestricted transcriptome profiling rather
  than targeted detection of specific loci close to the 5' end of highly expressed
  genes shorter than 4 kB
- The cDNA library is not 5'-anchored 10x Genomics chemistry, so cell barcodes and
  UMIs cannot be extracted from the long reads
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: nanoranger
  role: Core long-read single-cell processing pipeline run in 5p10XGEX mode to extract
    cell barcodes/UMIs and assign reads to CAR vs wildtype CD28 transcript classes
  repo: https://github.com/mehdiborji/nanoranger
- name: nanoranger.R
  role: 'R package for downstream processing: UMI collapse with starcode (Levenshtein
    distance 3) and extraction of CAR sequences from nanoranger amplicon output'
  repo: https://github.com/liviuspenter/nanoranger.R
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform producing the FASTQ input from targeted amplicons
    of the 5'-anchored 10x Genomics cDNA library
- name: 10x Genomics
  role: Source chemistry providing the 5'-anchored whole-transcriptome cDNA libraries
    with cell barcodes and UMIs recovered from the long reads
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:aff641ee411493c5f217f1e7cf5a681c5a0e14c089a745bbcef99b0c5d8a9b35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - 10x Genomics
  - Oxford Nanopore Technologies (ONT)
  - nanoranger
  - nanoranger.R
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ont-long-read-fastq-processing

## Summary

Process Oxford Nanopore long-read FASTQ data from targeted amplicons or whole-transcriptome 10x Genomics 5'-anchored cDNA libraries with the nanoranger pipeline to detect cell lineage-defining features such as chimeric antigen receptor (CAR) transcripts and distinguish them from wildtype (e.g. CD28) transcripts at single-cell resolution.

## When to use

Use when FASTQ files from ONT sequencing of 10x Genomics-derived amplicons or cDNA libraries must be converted into per-cell transcript classifications (e.g. CAR+ vs wildtype CD28) that short-read scRNA-seq fails to fully resolve.

## When NOT to use

- The input is short-read (Illumina) scRNA-seq data, which the paper notes is suboptimal for long fragments and cannot easily resolve structural variants such as transgenes like CARs
- The analysis goal is whole-genome or unrestricted transcriptome profiling rather than targeted detection of specific loci close to the 5' end of highly expressed genes shorter than 4 kB
- The cDNA library is not 5'-anchored 10x Genomics chemistry, so cell barcodes and UMIs cannot be extracted from the long reads

## Inputs

- ONT long-read FASTQ file from targeted amplicon sequencing of a 10x Genomics 5'-anchored cDNA library (e.g. 97_6_CAR.fastq.gz)
- Custom reference combining the chimeric antigen receptor construct and the endogenous target locus (e.g. CAR_CD28 reference)

## Outputs

- Per-cell quantification table classifying each cell barcode as CAR+ or wildtype CD28 (car_wt_quantification.csv)
- UMI-collapsed per-cell transcript counts assigned to CAR vs wildtype transcript classes

## How to apply

Start from an ONT-sequenced FASTQ such as 97_6_CAR.fastq.gz, produced by sequencing targeted amplicons amplified from an intermediate 10x Genomics 5'-anchored whole-transcriptome cDNA library using a CD28-specific primer. Run nanoranger in 5p10XGEX mode with a CAR_CD28 reference so long reads align to both the chimeric antigen receptor construct and the endogenous CD28 locus. The pipeline extracts 10x cell barcodes and UMIs from the long reads and assigns each read to the CAR or wildtype CD28 transcript class; downstream (nanoranger.R) collapses UMI counts per cell barcode using starcode at Levenshtein distance 3, since 10x UMIs lack a correction whitelist unlike cell barcodes. Classify each cell as CAR+ or wildtype CD28 by its dominant transcript class, summarize the overall CAR+ fraction, and export the per-cell table as car_wt_quantification.csv.

## Related tools

- **nanoranger** (Core long-read single-cell processing pipeline run in 5p10XGEX mode to extract cell barcodes/UMIs and assign reads to CAR vs wildtype CD28 transcript classes) — https://github.com/mehdiborji/nanoranger
- **nanoranger.R** (R package for downstream processing: UMI collapse with starcode (Levenshtein distance 3) and extraction of CAR sequences from nanoranger amplicon output) — https://github.com/liviuspenter/nanoranger.R
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform producing the FASTQ input from targeted amplicons of the 5'-anchored 10x Genomics cDNA library)
- **10x Genomics** (Source chemistry providing the 5'-anchored whole-transcriptome cDNA libraries with cell barcodes and UMIs recovered from the long reads)

## Examples

```
nanoranger --mode 5p10XGEX --fastq 97_6_CAR.fastq.gz --reference CAR_CD28 --output car_wt_quantification.csv
```

## Evaluation signals

- The computed fraction of CAR+ cells exceeds 0.75, consistent with the reported >75% CAR+ T cell detection
- Nanoranger-based detection recovers essentially all CAR+ T cells found by unenriched Illumina scRNA-seq plus an additional ~15% of cells
- Reads are assigned to both the CAR construct and the endogenous CD28 locus in the CAR_CD28 reference, demonstrating correct dual alignment
- UMI collapsing (starcode, Levenshtein distance <= 3) removes sequencing-error-derived UMIs before per-cell classification

## Limitations

- Detection depends on transcript representation: the ceiling for genotyping is determined by the low, incomplete representation of many transcripts and cannot be merely overcome through amplicon optimization or deeper sequencing
- Optimal nanoranger targets are restricted to loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB
- Short-read single-cell data lack sufficient resolution as a complete comparison baseline due to incomplete capture of long or lowly expressed transcripts

## Evidence

- [abstract] nanoranger long-read pipeline overview: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [intro] ONT targeted amplification of 5'-anchored 10x libraries: "we provide a long-read based pipeline for the ONT platform using limited primer sets to amplify target genes from 5′-anchored 10x Genomics scRNA-seq whole-transcriptome cDNA libraries"
- [methods] input data definition: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification"
- [readme] UMI collapsing strategy: [evidence span withheld — anchor status: external_doc]
- [abstract] optimal target definition: "we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB"
