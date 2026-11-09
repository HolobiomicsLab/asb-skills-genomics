---
name: snp-indel-variant-annotation
description: Use when a VCF of called SNPs/INDELs (e.g., from GATK HaplotypeCaller
  after SelectVariants/ApplyRecalibration) needs gene-level annotation and effect
  prediction, as in JWES-based whole genome/exome variant interpretation.
when_to_use_negative:
- Input is a raw SAM/BAM alignment file rather than a filtered VCF call set — run
  the JWES calling chain (HaplotypeCaller, SelectVariants, VariantRecalibrator, ApplyRecalibration)
  first
- The goal is read preprocessing or alignment (FastQC, Trimmomatic, BWA) rather than
  post-calling variant effect annotation
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0199
tools:
- name: snpEff
  role: Annotates called SNPs and INDELs and predicts their effects via an interval-forest
    approach, producing the annotated variant file in the final JWES step
- name: GATK
  role: HaplotypeCaller calls SNPs/INDELs; SelectVariants splits and re-selects passing
    variants; VariantRecalibrator/ApplyRecalibration filter the call set before annotation
- name: VariantEval
  role: Computes QC metrics (total SNP count and transition-to-transversion ratio)
    on the call set prior to annotation
- name: genomecov
  role: Computes per-base reports and coverage supporting the annotation stage
- name: JWES
  role: Pipeline orchestrating the calling-to-annotation workflow; its visualization
    module renders Circos graphs of annotated variants
  repo: github.com/drzeeshanahmed/JWES-DB
- name: Circos
  role: Data visualization module supporting variant data interpretation of the annotated
    results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:e282bf7a00d0f64aaea10f0166893c3f55801e92ecbe61da16b849c80b07f845
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA
  - Circos
  - FastQC
  - GATK
  - JWES
  - MarkDuplicates
  - SAMtools
  - SortSam
  - Trimmomatic
  - VariantEval
  - genomecov
  - snpEff
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# snp-indel-variant-annotation

## Summary

Annotate and predict the functional effects of SNPs and INDELs called from WGS/WES data using snpEff's interval-forest approach, as implemented in the final stage of the JWES pipeline, optionally pairing annotation output with per-base coverage reports from genomecov.

## When to use

Use when a VCF of called SNPs/INDELs (e.g., from GATK HaplotypeCaller after SelectVariants/ApplyRecalibration) needs gene-level annotation and effect prediction, as in JWES-based whole genome/exome variant interpretation.

## When NOT to use

- Input is a raw SAM/BAM alignment file rather than a filtered VCF call set — run the JWES calling chain (HaplotypeCaller, SelectVariants, VariantRecalibrator, ApplyRecalibration) first
- The goal is read preprocessing or alignment (FastQC, Trimmomatic, BWA) rather than post-calling variant effect annotation

## Inputs

- Filtered SNP VCF (post ApplyRecalibration/SelectVariants)
- Filtered INDEL VCF (post ApplyRecalibration/SelectVariants)
- hg38 human reference genome annotation context

## Outputs

- snpEff-annotated variant file with predicted variant effects
- Per-base coverage reports from genomecov
- Circos graphs for variant data interpretation

## How to apply

After variant calling (GATK HaplotypeCaller producing SNPs and INDELs) and filtering (VariantRecalibrator Gaussian mixture model, ApplyRecalibration score cutoff, SelectVariants re-selection of passing variants), forward the passing variant call set to snpEff for annotation. snpEff uses an interval-forest approach to annotate and predict the effects of variants, producing an annotated variant file. Optionally compute per-base reports and coverage with genomecov to support variant interpretation, and use the JWES data visualization module with Circos graphs to interpret the annotated variant data. QC the call set before annotation by computing total SNP count and transition-to-transversion ratio with VariantEval.

## Related tools

- **snpEff** (Annotates called SNPs and INDELs and predicts their effects via an interval-forest approach, producing the annotated variant file in the final JWES step)
- **GATK** (HaplotypeCaller calls SNPs/INDELs; SelectVariants splits and re-selects passing variants; VariantRecalibrator/ApplyRecalibration filter the call set before annotation)
- **VariantEval** (Computes QC metrics (total SNP count and transition-to-transversion ratio) on the call set prior to annotation)
- **genomecov** (Computes per-base reports and coverage supporting the annotation stage)
- **JWES** (Pipeline orchestrating the calling-to-annotation workflow; its visualization module renders Circos graphs of annotated variants) — github.com/drzeeshanahmed/JWES-DB
- **Circos** (Data visualization module supporting variant data interpretation of the annotated results)

## Evaluation signals

- The input VCF contains only variants that passed ApplyRecalibration and were re-selected with SelectVariants
- The output is an annotated variant file produced by snpEff with predicted effects for each SNP and INDEL
- Variant QC metrics computed with VariantEval (total SNP count and transition-to-transversion ratio) are within plausible ranges for the sample
- Per-base coverage reports from genomecov are present alongside the annotated variants
- Cohort-scale sanity check: JWES reported 4,803,792 variants across 14 publicly available WGS samples

## Limitations

- The current version of JWES offers only a command-line interface; an interactive/easy-to-use interface is not yet implemented
- Advanced visualizations such as dynamic heat maps, gene pathways/networks, and cluster maps for gene expression and variant analysis are not yet implemented
- JWES annotates variants but the paper notes a broader lack of simpler clinician/patient-friendly pipelines for clinical personalized medicine settings

## Evidence

- [methods] Compute per-base reports and coverage with genomecov, and annotate and predict the effects of the variants with snpEff's interval-forest approach to produce the annotated variant file.: [evidence span withheld — anchor status: not_found]
- [methods] HaplotypeCaller is used to call SNPs and INDELs: "HaplotypeCaller is used to call SNPs and INDELs"
- [abstract] the data visualization module supports variant data interpretation with Circos graphs: "the data visualization module supports variant data interpretation with Circos graphs"
- [readme] VCF Parser for JWES: "JWES-DB VCF Parser for JWES"
- [discussion] The current version of the JWES offers command line but an easy-to-use interface.: "The current version of the JWES offers command line but an easy‐to‐use interface."
