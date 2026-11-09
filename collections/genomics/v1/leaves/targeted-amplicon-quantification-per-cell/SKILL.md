---
name: targeted-amplicon-quantification-per-cell
description: Use when you have FASTQ files from targeted amplification of a transgene/wildtype
  locus from a 10x Genomics 5' cDNA library sequenced on ONT, and need per-cell CAR+
  vs wildtype transcript classification with UMI collapsing.
when_to_use_negative:
- You only need standard whole-transcriptome gene-expression counts from unenriched
  10x libraries without transgene/wildtype discrimination — nanoranger's targeted
  5p10XGEX mode is for locus-level genotyping, not full GEX.
- The target locus is far from the 5' end or the transcript exceeds ~4 kB, since optimal
  nanoranger targets are loci close to the 5' end of highly expressed genes with transcript
  lengths shorter than 4 kB.
- Input data are short Illumina reads without an intermediate single-cell cDNA library
  — barcode/UMI extraction from long reads is the basis of the workflow.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0632
tools:
- name: nanoranger
  role: 'Long-read pipeline run in 5p10XGEX mode: aligns amplicon reads to the CAR_CD28
    reference, extracts 10x cell barcodes and UMIs, and assigns reads to CAR or wildtype
    CD28 transcript classes'
  repo: https://github.com/mehdiborji/nanoranger
- name: nanoranger.R
  role: Downstream R package that collapses sequencing-error UMIs (starcode, Levenshtein
    distance 3) and reads out CAR sequences from amplicons for per-cell classification
  repo: https://github.com/liviuspenter/nanoranger.R
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform used to sequence the targeted amplicons from
    the intermediate cDNA library
- name: 10x Genomics
  role: Source of the 5'-anchored whole-transcriptome cDNA library whose cell barcodes
    and UMIs are carried into the targeted amplicons
- name: minimap2
  role: Long-read aligner suitable for mapping noisy ONT amplicon reads to the CAR_CD28
    reference
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:b6c783bdcf564fff20746c39bb738dbea837549dc05fea46141c33afe111b7bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - 10x Genomics
  - Oxford Nanopore Technologies (ONT)
  - minimap2
  - nanoranger
  - nanoranger.R
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# targeted-amplicon-quantification-per-cell

## Summary

Quantifies targeted long-read amplicons per single cell by running nanoranger in 5p10XGEX mode on ONT-sequenced amplicons from intermediate 10x Genomics 5'-anchored cDNA libraries, assigning each read to a transgene (e.g. CD19 CAR) versus its wildtype counterpart (endogenous CD28) and collapsing UMIs per cell barcode.

## When to use

Use when you have FASTQ files from targeted amplification of a transgene/wildtype locus from a 10x Genomics 5' cDNA library sequenced on ONT, and need per-cell CAR+ vs wildtype transcript classification with UMI collapsing.

## When NOT to use

- You only need standard whole-transcriptome gene-expression counts from unenriched 10x libraries without transgene/wildtype discrimination — nanoranger's targeted 5p10XGEX mode is for locus-level genotyping, not full GEX.
- The target locus is far from the 5' end or the transcript exceeds ~4 kB, since optimal nanoranger targets are loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB.
- Input data are short Illumina reads without an intermediate single-cell cDNA library — barcode/UMI extraction from long reads is the basis of the workflow.

## Inputs

- FASTQ of targeted amplicons (e.g. 97_6_CAR.fastq.gz) from an intermediate 10x Genomics 5'-anchored whole-transcriptome cDNA library sequenced on ONT
- Combined chimeric-antigen-receptor + endogenous locus reference (CAR_CD28) for alignment
- CD28-specific primer-designed amplicon spanning both CAR and wildtype CD28 transcripts

## Outputs

- Per-cell quantification table (car_wt_quantification.csv) with cell barcode, UMI counts per transcript class, and CAR+/wildtype CD28 classification
- Overall fraction of CAR+ cells (>0.75 expected for this dataset)

## How to apply

Start from an intermediate 10x Genomics 5'-anchored whole-transcriptome cDNA library, amplify the target locus with a locus-specific primer (here a CD28-specific primer that detects both the CAR construct and endogenous CD28), and sequence amplicons on an Oxford Nanopore device to obtain FASTQ (e.g. 97_6_CAR.fastq.gz). Run nanoranger in 5p10XGEX mode with a combined CAR_CD28 reference so reads align to both the chimeric antigen receptor construct and the endogenous CD28 locus; nanoranger extracts the 10x cell barcodes and UMIs from the long reads and assigns each read to the CAR or wildtype CD28 transcript class. Collapse UMI counts per cell barcode — the downstream nanoranger.R package collapses UMIs that arise from sequencing error within a Levenshtein distance of 3 using starcode, since 10x UMIs lack a whitelist — and classify each cell as CAR+ or wildtype CD28 based on the dominant transcript class. Export the per-cell quantification (car_wt_quantification.csv) and compute the overall fraction of CAR+ cells. Rationale: short-read scRNA-seq incompletely captures the CAR transgene, while targeted long-read enrichment plus cell-barcode/UMI extraction recovers these cells; this workflow detected all CAR+ cells found by unenriched Illumina scRNA-seq plus ~15% more, exceeding 75% CAR+ T cell detection.

## Related tools

- **nanoranger** (Long-read pipeline run in 5p10XGEX mode: aligns amplicon reads to the CAR_CD28 reference, extracts 10x cell barcodes and UMIs, and assigns reads to CAR or wildtype CD28 transcript classes) — https://github.com/mehdiborji/nanoranger
- **nanoranger.R** (Downstream R package that collapses sequencing-error UMIs (starcode, Levenshtein distance 3) and reads out CAR sequences from amplicons for per-cell classification) — https://github.com/liviuspenter/nanoranger.R
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform used to sequence the targeted amplicons from the intermediate cDNA library)
- **10x Genomics** (Source of the 5'-anchored whole-transcriptome cDNA library whose cell barcodes and UMIs are carried into the targeted amplicons)
- **minimap2** (Long-read aligner suitable for mapping noisy ONT amplicon reads to the CAR_CD28 reference) — https://github.com/lh3/minimap2

## Examples

```
nanoranger 5p10XGEX --fastq 97_6_CAR.fastq.gz --reference CAR_CD28 --out car_wt_quantification.csv
```

## Evaluation signals

- Computed fraction of CAR+ cells is above 0.75, consistent with the reported >75% CAR+ T cell detection (15% more CAR+ cells than unenriched Illumina scRNA-seq).
- Every CAR+ cell identified by unenriched Illumina scRNA-seq is recovered in the per-cell quantification table.
- UMI collapsing groups spurious UMIs within Levenshtein distance 3 (starcode), so UMI counts per cell barcode reflect molecule counts rather than sequencing-error duplicates.
- Each read is unambiguously assigned to either the CAR construct or the endogenous CD28 locus in the CAR_CD28 reference alignment.
- car_wt_quantification.csv contains one row per cell barcode with a defined transcript-class label (CAR+ or wildtype CD28).

## Limitations

- Optimal targets are loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB; distant or long targets perform poorly.
- The genotyping ceiling is set by low, incomplete transcript representation and cannot be merely overcome through amplicon optimization or deeper sequencing.
- Detection depends on a CD28-specific primer capturing both CAR and wildtype transcripts; targets without such a shared primer design lose the wildtype discrimination capability.
- 10x UMIs have no whitelist, so sequencing-error-derived UMIs must be collapsed heuristically (Levenshtein distance 3), which can merge distinct true UMIs.

## Evidence

- [abstract] we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [intro] limited primer sets to amplify target genes from 5′-anchored 10x libraries: "we provide a long-read based pipeline for the ONT platform using limited primer sets to amplify target genes from 5′-anchored 10x Genomics scRNA-seq whole-transcriptome cDNA libraries"
- [readme] input data from 10x cDNA libraries or targeted amplicons: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification"
- [readme] UMI collapsing via starcode at Levenshtein distance 3: [evidence span withheld — anchor status: external_doc]
- [intro] short reads cannot resolve transgenes/fusions: "short-read sequencing is unable to easily resolve structural transcriptomic variants such as gene fusions, transgenes or even isoforms"
