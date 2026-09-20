---
name: variant-effect-annotation-with-snpeff
description: Use when a filtered VCF of SNPs and INDELs (e.g., GATK HaplotypeCaller
  output recalibrated against hg38) needs gene-level variant annotation and effect
  prediction as the last stage of a WGS/WES variant-discovery workflow.
when_to_use_negative:
- Input is raw FASTQ or unaligned reads — annotation requires a called and recalibrated
  variant VCF, not primary sequence data
- Variant call set has not yet passed VariantRecalibrator/ApplyRecalibration filtering
  — annotate only the re-selected passing variants, not the full HaplotypeCaller output
- A web/interactive interface is required — the current JWES version offers only a
  command-line workflow
edam_operation: http://edamontology.org/operation_0803
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: snpEff
  role: Annotates the filtered SNP/INDEL VCF with predicted variant effects to produce
    the final annotated variant file in the JWES pipeline
- name: JWES
  role: Enclosing Java-based WGS/WES pipeline whose step 6 invokes snpEff for gene-variant
    discovery, annotation, prediction, and genotyping
  repo: github.com/drzeeshanahmed/JWES-DB
- name: JWES-DB
  role: VCF parser that consumes the annotated JWES variant output for management
    and downstream interpretation
  repo: github.com/drzeeshanahmed/JWES-DB
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:0f2e66ed9bb00f5280e8147cc1165e2cb3a5c3a9b2a61692cb2f987daa7d6bb3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA
  - FastQC
  - JWES
  - JWES-DB
  - Java
  - MarkDuplicates
  - SAMtools
  - SortSam
  - Trimmomatic
  - snpEff
  edam_operation: http://edamontology.org/operation_0803
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# variant-effect-annotation-with-snpeff

## Summary

Final step of the JWES whole genome/exome pipeline: annotate called and recalibrated SNP/INDEL variants with predicted gene-level effects using snpEff, producing the final annotated variant file that downstream tools (e.g., the JWES-DB VCF parser) can consume.

## When to use

Use when a filtered VCF of SNPs and INDELs (e.g., GATK HaplotypeCaller output recalibrated against hg38) needs gene-level variant annotation and effect prediction as the last stage of a WGS/WES variant-discovery workflow.

## When NOT to use

- Input is raw FASTQ or unaligned reads — annotation requires a called and recalibrated variant VCF, not primary sequence data
- Variant call set has not yet passed VariantRecalibrator/ApplyRecalibration filtering — annotate only the re-selected passing variants, not the full HaplotypeCaller output
- A web/interactive interface is required — the current JWES version offers only a command-line workflow

## Inputs

- Filtered/re-selected VCF of SNPs and INDELs (post ApplyRecalibration and SelectVariants) from the JWES data-processing module
- hg38 human reference genome annotation database for snpEff

## Outputs

- Final annotated variant VCF with gene-level effect predictions from snpEff
- Per-sample variant counts and QC metrics (total SNPs, transition/transversion ratio) for downstream tabulation

## How to apply

Run snpEff as step 6 of the JWES pipeline, after HaplotypeCaller-based variant calling, SelectVariants splitting into SNP and INDEL VCFs, VariantRecalibrator/ApplyRecalibration filtering at the specified sensitivity threshold, and re-selection of passing variants. Input is the final passing-variant VCF aligned against the hg38 human reference genome; snpEff annotates each variant with gene-level consequences and predicted effects, producing the final annotated variant file. The annotated output is then used to tabulate per-sample variant totals (e.g., 20,749 variants for SRR891275 / PRJNA207663; 2,736,453 for PRJNA657938) and QC metrics such as transition/transversion ratio via VariantEval. Correct application is judged by the annotated file retaining all previously passing variants, carrying gene annotations/effect predictions, and matching expected counts and Ti/Tv values from the published case studies.

## Related tools

- **snpEff** (Annotates the filtered SNP/INDEL VCF with predicted variant effects to produce the final annotated variant file in the JWES pipeline)
- **JWES** (Enclosing Java-based WGS/WES pipeline whose step 6 invokes snpEff for gene-variant discovery, annotation, prediction, and genotyping) — github.com/drzeeshanahmed/JWES-DB
- **JWES-DB** (VCF parser that consumes the annotated JWES variant output for management and downstream interpretation) — github.com/drzeeshanahmed/JWES-DB

## Examples

```
java -jar snpEff.jar hg38 filtered_pass.vcf > SRR891275_annotated.vcf
```

## Evaluation signals

- Annotated VCF contains the same variant records as the pre-annotation passing-variant VCF (no variants lost by SelectVariants re-selection)
- Every called variant carries gene-level annotation/effect prediction entries from snpEff
- Total variant count matches published case-study values (e.g., 20,749 for SRR891275 of PRJNA207663; 43,685 for SRR12474733 of PRJNA657985)
- VariantEval QC metrics (total SNPs, transition/transversion ratio) are computed on the annotated call set and fall in expected ranges for WGS data
- Annotated output is parseable by the JWES-DB VCF parser

## Limitations

- JWES currently offers only a command-line interface; an easy-to-use interactive interface has not yet been implemented
- Advanced visualizations such as dynamic heat maps, gene pathways/networks, and cluster maps for variant analysis are planned but not yet implemented
- Computing environments must be FAIR-aligned and require command-line competence; the paper notes a lacuna of simpler clinician/patient-friendly pipelines for clinical personalized medicine settings

## Evidence

- [abstract] We have successfully applied JWES for processing, management, and gene-variant discovery, annotation, prediction, and genotyping of WGS and WES data: "We have successfully applied JWES for processing, management, and gene-variant discovery, annotation, prediction, and genotyping of WGS and WES data"
- [other] Annotate and predict variant effects with snpEff to produce the final annotated variant file.: [evidence span withheld — anchor status: not_found]
- [readme] VCF Parser for JWES: "# JWES-DB
VCF Parser for JWES"
- [results] and 20 749 variants for PRJNA207663: "and 20 749 variants for PRJNA207663"
