---
name: gatk-variant-recalibration-filtering
description: Use when a VCF call set produced by GATK HaplotypeCaller (from BWA/hg38-aligned
  WGS/WES reads in JWES) must be recalibrated and filtered to a high-quality SNP/INDEL
  subset before annotation or counting.
when_to_use_negative:
- The call set is not the output of GATK HaplotypeCaller in the JWES chain (e.g.,
  reads have not yet been BWA-aligned to hg38, sorted, duplicate-marked, and indexed
  with SAMtools)
- Only raw FASTQ/FASTA input is available — the FastQC and Trimmomatic QC/trimming
  and the BaseRecalibrator/IndelRealigner preprocessing steps must run first
- A hard-filter approach is required instead of VQSR-style Gaussian mixture model
  recalibration, or the variant types needed are SVs rather than SNPs/INDELs
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: GATK
  role: Provides VariantRecalibrator (Gaussian mixture model), ApplyRecalibration
    (score cutoff filtering), HaplotypeCaller (upstream calling), and VariantEval
    (SNP count and Ti/Tv QC)
- name: SelectVariants
  role: Splits HaplotypeCaller output into SNP and INDEL VCFs and re-selects variants
    passing ApplyRecalibration
- name: JWES
  role: Pipeline context that orchestrates the recalibration/filtering loop after
    BWA/SortSam/MarkDuplicates/SAMtools preprocessing
  repo: github.com/drzeeshanahmed/JWES-DB
- name: snpEff
  role: Consumes the filtered variant file for annotation and effect prediction after
    recalibration
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:0a57fe09803f440e28c38b53fda5acb690be816e1af807cd1a5e3abad796d5a8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA
  - FastQC
  - GATK
  - JWES
  - MarkDuplicates
  - SAMtools
  - SelectVariants
  - SortSam
  - Trimmomatic
  - snpEff
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# gatk-variant-recalibration-filtering

## Summary

Recalibrates and filters GATK HaplotypeCaller SNP/INDEL call sets from the JWES WGS/WES pipeline using VariantRecalibrator's Gaussian mixture model and ApplyRecalibration score cutoffs, then re-selects passing variants and computes QC metrics with VariantEval.

## When to use

Use when a VCF call set produced by GATK HaplotypeCaller (from BWA/hg38-aligned WGS/WES reads in JWES) must be recalibrated and filtered to a high-quality SNP/INDEL subset before annotation or counting.

## When NOT to use

- The call set is not the output of GATK HaplotypeCaller in the JWES chain (e.g., reads have not yet been BWA-aligned to hg38, sorted, duplicate-marked, and indexed with SAMtools)
- Only raw FASTQ/FASTA input is available — the FastQC and Trimmomatic QC/trimming and the BaseRecalibrator/IndelRealigner preprocessing steps must run first
- A hard-filter approach is required instead of VQSR-style Gaussian mixture model recalibration, or the variant types needed are SVs rather than SNPs/INDELs

## Inputs

- SNP VCF from GATK HaplotypeCaller, split with SelectVariants (hg38-aligned reads)
- INDEL VCF from GATK HaplotypeCaller, split with SelectVariants
- GATK recalibration resources (Gaussian mixture model inputs) for VariantRecalibrator

## Outputs

- High-quality recalibrated SNP VCF (variants passing the ApplyRecalibration sensitivity threshold)
- Recalibrated INDEL VCF after re-selection with SelectVariants
- VariantEval QC metrics: total SNP count and transition-to-transversion ratio
- Annotated variant file after downstream snpEff effect prediction

## How to apply

Run this step after HaplotypeCaller has called SNPs and INDELs and SelectVariants has split the call set into separate SNP and INDEL VCF files. Apply VariantRecalibrator to build a Gaussian mixture model and produce a high-quality cluster of SNPs, then run ApplyRecalibration to apply a score cutoff that filters variants below the specified sensitivity threshold. Re-select the variants that passed ApplyRecalibration with SelectVariants, and compute QC metrics — total SNP count and transition-to-transversion ratio — with VariantEval. The recalibrated, filtered VCFs are then ready for per-base coverage reporting (genomecov) and snpEff annotation. In the JWES benchmark, this loop ran on HPC (Slurm) over SRA WGS samples (e.g., PRJNA657985/SRR12474733 yielding 43,685 variants and PRJNA657938 yielding 2,736,453 variants).

## Related tools

- **GATK** (Provides VariantRecalibrator (Gaussian mixture model), ApplyRecalibration (score cutoff filtering), HaplotypeCaller (upstream calling), and VariantEval (SNP count and Ti/Tv QC))
- **SelectVariants** (Splits HaplotypeCaller output into SNP and INDEL VCFs and re-selects variants passing ApplyRecalibration)
- **JWES** (Pipeline context that orchestrates the recalibration/filtering loop after BWA/SortSam/MarkDuplicates/SAMtools preprocessing) — github.com/drzeeshanahmed/JWES-DB
- **snpEff** (Consumes the filtered variant file for annotation and effect prediction after recalibration)

## Evaluation signals

- VariantRecalibrator successfully built a Gaussian mixture model and produced a high-quality cluster of SNPs before ApplyRecalibration ran
- ApplyRecalibration applied a score cutoff and removed variants below the specified sensitivity threshold; only passing variants remain after SelectVariants re-selection
- VariantEval reports a finite total SNP count and a transition-to-transversion ratio in a plausible range for the sample (e.g., 43,685 variants for SRR12474733; 54,930 for PRJNA649101; 154,016 for PRJEB39632)
- Per-project variant counts match the reported figures: 2,736,453 (PRJNA657938) and 1,793,959 (PRJNA624223), checked within tolerance of each reported value
- The filtered VCF is parseable by the JWES-DB VCF parser and downstream snpEff annotation completes without errors

## Limitations

- The current JWES version offers only a command-line interface; no interactive interface is implemented
- VQSR-style recalibration requires a sufficiently large variant set — small samples (e.g., the 43,685-variant SRR12474733 exome-scale case) may behave differently from large WGS cohorts (4,803,792 variants across 14 samples)
- Recalibration quality should be inspected via AnalyzeCovariates line and bar plots on the two recalibration tables produced by BaseRecalibrator

## Evidence

- [methods] The JWES applies VariantRecalibrator to create a Gaussian mixture model and produce high‐quality cluster of SNPs: "The JWES applies VariantRecalibrator to create a Gaussian mixture model and produce high‐quality cluster of SNPs"
- [methods] SelectVariants is then used to select the variants that have successfully passed the ApplyRecalibration step.: "SelectVariants is then used to select the variants that have successfully passed the ApplyRecalibration step."
- [methods] HaplotypeCaller is used to call SNPs and INDELs: "HaplotypeCaller is used to call SNPs and INDELs"
- [results] we applied the JWES to the WGS sample (SRR12474733) of project PRJNA657985, which resulted in 43 685 variants: "we applied the JWES to the WGS sample (SRR12474733) of project PRJNA657985, which resulted in 43 685 variants"
