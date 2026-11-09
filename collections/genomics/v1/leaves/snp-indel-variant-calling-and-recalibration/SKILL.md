---
name: snp-indel-variant-calling-and-recalibration
description: Use when starting from a duplicate-marked, indexed BAM aligned to hg38
  and the goal is a calibrated SNP/INDEL VCF with QC metrics (SNP count, transition/transversion
  ratio) comparable to published JWES runs.
when_to_use_negative:
- Input is still raw FASTQ or an unsorted/unmarked BAM — the skill presupposes the
  upstream JWES alignment, sorting, MarkDuplicates, and indexing steps are already
  complete
- The goal is detection of structural variants or imputation/haplotype resolution
  rather than SNP/INDEL calling — JWES' data-processing module targets variant calling
  for SNPs and INDELs, with SVs handled elsewhere in the pipeline scope
- A different reference genome than hg38 is required — the article's method maps against
  the hg38 human reference genome
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: JWES
  role: Pipeline orchestrating realignment, recalibration, variant calling, and filtering
    steps
  repo: github.com/drzeeshanahmed/JWES-DB
- name: GATK
  role: Provides RealignerTargetCreator, IndelRealigner, BaseRecalibrator, AnalyzeCovariates,
    HaplotypeCaller, SelectVariants, VariantRecalibrator, ApplyRecalibration, and
    VariantEval
- name: SAMtools
  role: Index tool creates the BAM index enabling fast access before variant calling
- name: MarkDuplicates
  role: Locates, tags, and removes duplicate reads to reduce variant false positives
    prior to calling
- name: snpEff
  role: Annotates and predicts variant effects on the final filtered call set
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:f9488f045adf29bcccb3b8753775bc4e4cbf9f1163a29e8286f996f28f4c4d0f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA
  - FastQC
  - GATK
  - JWES
  - Java
  - MarkDuplicates
  - SAMtools
  - SortSam
  - Trimmomatic
  - snpEff
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 20
---

# snp-indel-variant-calling-and-recalibration

## Summary

Call SNPs and INDELs from aligned WGS/WES BAMs using GATK HaplotypeCaller within the JWES pipeline, then recalibrate variant quality with VariantRecalibrator/ApplyRecalibration and quantify results with VariantEval.

## When to use

Use when starting from a duplicate-marked, indexed BAM aligned to hg38 and the goal is a calibrated SNP/INDEL VCF with QC metrics (SNP count, transition/transversion ratio) comparable to published JWES runs.

## When NOT to use

- Input is still raw FASTQ or an unsorted/unmarked BAM — the skill presupposes the upstream JWES alignment, sorting, MarkDuplicates, and indexing steps are already complete
- The goal is detection of structural variants or imputation/haplotype resolution rather than SNP/INDEL calling — JWES' data-processing module targets variant calling for SNPs and INDELs, with SVs handled elsewhere in the pipeline scope
- A different reference genome than hg38 is required — the article's method maps against the hg38 human reference genome

## Inputs

- Duplicate-marked, indexed BAM aligned to the human reference genome hg38 (from BWA + SortSam + MarkDuplicates + SAMtools index)
- Base quality score recalibration tables produced by BaseRecalibrator
- Known-variant resources for the Gaussian mixture model used by VariantRecalibrator

## Outputs

- Raw multi-sample VCF from HaplotypeCaller with SNPs and INDELs
- Recalibrated and filtered SNP and INDEL VCFs (post ApplyRecalibration, re-selected with SelectVariants)
- VariantEval QC report (total SNPs, transition/transversion ratio)
- Annotated final variant file from snpEff

## How to apply

After BWA alignment, SortSam sorting, MarkDuplicates, and SAMtools indexing, locally realign reads by determining intervals with GATK RealignerTargetCreator and realigning with IndelRealigner, then adjust base quality scores with BaseRecalibrator using an empirical machine learning model, inspecting the two recalibration tables with AnalyzeCovariates. Call SNPs and INDELs jointly on the recalibrated BAM with HaplotypeCaller, split the call set into SNP and INDEL VCFs with SelectVariants, build a Gaussian mixture model of high-quality SNP clusters with VariantRecalibrator, and filter with ApplyRecalibration at the specified sensitivity threshold. Re-select passing variants with SelectVariants and compute QC metrics — total SNPs and transition/transversion ratio — with VariantEval before annotating with snpEff. Validate success by comparing the resulting total variant count against the published value for the sample (e.g. 20,749 variants for SRR891275 of PRJNA207663).

## Related tools

- **JWES** (Pipeline orchestrating realignment, recalibration, variant calling, and filtering steps) — github.com/drzeeshanahmed/JWES-DB
- **GATK** (Provides RealignerTargetCreator, IndelRealigner, BaseRecalibrator, AnalyzeCovariates, HaplotypeCaller, SelectVariants, VariantRecalibrator, ApplyRecalibration, and VariantEval)
- **SAMtools** (Index tool creates the BAM index enabling fast access before variant calling)
- **MarkDuplicates** (Locates, tags, and removes duplicate reads to reduce variant false positives prior to calling)
- **snpEff** (Annotates and predicts variant effects on the final filtered call set)

## Evaluation signals

- Total variant count matches the published reference value for the sample (e.g. 20,749 variants for SRR891275 / PRJNA207663)
- VariantEval reports a plausible total SNP count and transition/transversion ratio for a WGS/WES call set
- All reported variants pass the ApplyRecalibration sensitivity threshold and are re-selected by SelectVariants
- Recalibration tables from BaseRecalibrator were produced in duplicate and reviewed via AnalyzeCovariates line/bar plots before calling
- The final VCF is snpEff-annotated and parseable by JWES-DB (the JWES VCF parser)

## Limitations

- Duplicate reads must be removed (MarkDuplicates) or variant false positives increase; skipping this step degrades call quality
- The current JWES version only offers a command-line interface; an easy-to-use interactive interface is not yet implemented
- Published counts are pipeline- and reference-specific (hg38, GATK recalibration sensitivity threshold), so counts will differ under other parameterizations or references

## Evidence

- [methods] HaplotypeCaller is used to call SNPs and INDELs: "HaplotypeCaller is used to call SNPs and INDELs"
- [methods] Gaussian mixture model: "The JWES applies VariantRecalibrator to create a Gaussian mixture model and produce high‐quality cluster of SNPs"
- [methods] BaseRecalibrator empirical ML model: "BaseRecalibrator is used to adjust the quality scores by using an empirical machine learning (ML) model"
- [methods] MarkDuplicates reduces false positives: "to locate, tag, and remove duplicate reads, and enhance the quality of the alignment and reduce the number of variant false positives"
- [results] 20,749 variants for PRJNA207663: "and 20 749 variants for PRJNA207663"
- [readme] JWES-DB is a VCF parser: "JWES-DB VCF Parser for JWES"
