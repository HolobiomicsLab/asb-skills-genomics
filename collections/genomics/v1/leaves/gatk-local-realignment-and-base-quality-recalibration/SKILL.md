---
name: gatk-local-realignment-and-base-quality-recalibration
description: Use when a sorted, duplicate-marked BAM from WGS/WES data must be locally
  realigned around indels and base quality scores recalibrated before SNP/INDEL calling.
when_to_use_negative:
- Input is raw FASTQ reads that have not yet been aligned (BWA), sorted (SortSam),
  duplicate-marked (MarkDuplicates), and indexed (SAMtools) — run those JWES steps
  first.
- Analysis goal does not include SNP/INDEL variant discovery (e.g., expression quantification
  or assembly), since realignment/BQAR here is specifically preparation for GATK HaplotypeCaller.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: GATK
  role: Provides RealignerTargetCreator, IndelRealigner, BaseRecalibrator, and AnalyzeCovariates
    for local realignment and quality-score adjustment
- name: BWA
  role: Upstream tool that mapped reads against hg38, producing the SAM that becomes
    this stage's BAM input
- name: SortSam
  role: Sorted the BWA SAM by reference sequence name and left-most mapping position
    before duplicate marking
- name: MarkDuplicates
  role: Located, tagged, and removed duplicate reads to reduce variant false positives
    before recalibration
- name: SAMtools
  role: index tool created the BAM index enabling fast access during GATK processing
- name: AnalyzeCovariates
  role: Inputs the two recalibration tables and produces line and bar plots for quality
    inspection
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:83fc29a5357e9484cebbdd09423ab6c5a4869223f97119e5e4876617b4d89667
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnalyzeCovariates
  - BWA
  - FastQC
  - GATK
  - JWES
  - Java
  - MarkDuplicates
  - SAMtools
  - SortSam
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# GATK local realignment and base quality recalibration

## Summary

The GATK-based indel realignment and base quality score recalibration stage of the JWES pipeline, applied to a duplicate-marked, indexed BAM (here aligned to hg38) to correct mapping/quality artifacts before HaplotypeCaller variant calling.

## When to use

Use when a sorted, duplicate-marked BAM from WGS/WES data must be locally realigned around indels and base quality scores recalibrated before SNP/INDEL calling.

## When NOT to use

- Input is raw FASTQ reads that have not yet been aligned (BWA), sorted (SortSam), duplicate-marked (MarkDuplicates), and indexed (SAMtools) — run those JWES steps first.
- Analysis goal does not include SNP/INDEL variant discovery (e.g., expression quantification or assembly), since realignment/BQAR here is specifically preparation for GATK HaplotypeCaller.

## Inputs

- Sorted, duplicate-marked BAM file (from BWA -> SortSam -> MarkDuplicates) with a SAMtools .bai index
- Human reference genome hg38 (FASTA with indices)
- Known indel/resource sites expected by GATK (implicit in RealignerTargetCreator/BaseRecalibrator runs)

## Outputs

- Locally realigned BAM (IndelRealigner output)
- Two base quality recalibration tables (BaseRecalibrator output, GATK table format)
- AnalyzeCovariates line and bar plots of recalibration metrics
- Recalibrated BAM ready for HaplotypeCaller

## How to apply

After BWA alignment to hg38 and SAMtools indexing, run the GATK pair RealignerTargetCreator and IndelRealigner to determine intervals and locally realign them, which reduces false SNP evidence around indels. Then run BaseRecalibrator twice (e.g., for two recalibration cycles/tables) using an empirical machine-learning model to adjust base quality scores, inspecting the two recalibration tables with AnalyzeCovariates which produces line and bar plots to judge recalibration quality. Once the recalibrated BAM is produced, proceed to HaplotypeCaller for SNP/INDEL calling. Correct application is judged by the presence of the indexed BAM, the two recalibration tables, AnalyzeCovariates plots, and a downstream call set whose counts are comparable to published JWES results (e.g., 20,749 variants for SRR891275 of PRJNA207663).

## Related tools

- **GATK** (Provides RealignerTargetCreator, IndelRealigner, BaseRecalibrator, and AnalyzeCovariates for local realignment and quality-score adjustment)
- **BWA** (Upstream tool that mapped reads against hg38, producing the SAM that becomes this stage's BAM input)
- **SortSam** (Sorted the BWA SAM by reference sequence name and left-most mapping position before duplicate marking)
- **MarkDuplicates** (Located, tagged, and removed duplicate reads to reduce variant false positives before recalibration)
- **SAMtools** (index tool created the BAM index enabling fast access during GATK processing)
- **AnalyzeCovariates** (Inputs the two recalibration tables and produces line and bar plots for quality inspection)

## Evaluation signals

- RealignerTargetCreator/IndelRealigner run completes on the duplicate-marked hg38 BAM without interval errors, producing a realigned BAM.
- Two recalibration tables are produced and AnalyzeCovariates renders expected line and bar plots from them.
- Downstream HaplotypeCaller runs on the recalibrated BAM and yields a call set (after SelectVariants/VariantRecalibrator/ApplyRecalibration) whose variant count is in the expected range — e.g., 20,749 total variants for SRR891275 (PRJNA207663).
- QC metrics from VariantEval (total SNPs, transition/transversion ratio) fall in plausible ranges after recalibration.
- BAM remains accompanied by a valid SAMtools .bai index throughout GATK processing.

## Limitations

- The GATK realignment/BQAR stage is part of JWES, which currently offers only a command-line interface; no interactive interface is implemented.
- Duplicate-marking quality directly affects variant false positives, so errors in SortSam/MarkDuplicates propagate into realignment and recalibration.
- Results depend on reference build (hg38) and project-specific datasets; counts are not transferable across projects (e.g., 43,685 for SRR12474733/PRJNA657985 vs 20,749 for PRJNA207663).

## Evidence

- [methods] RealignerTargetCreator and IndelRealigner: "(a) determine intervals with RealignerTargetCreator and (b) realign intervals with IndelRealigner"
- [methods] BaseRecalibrator: "BaseRecalibrator is used to adjust the quality scores by using an empirical machine learning (ML) model"
- [methods] AnalyzeCovariates: "AnalyzeCovariates inputs these two recalibrations tables and produces multiple lines and bar plots"
- [methods] MarkDuplicates rationale: "to locate, tag, and remove duplicate reads, and enhance the quality of the alignment and reduce the number of variant false positives"
- [methods] BWA hg38 reference: "The Burrows–Wheeler Aligner (BWA) tool is then used for mapping sequence data against the reference human genome (hg38)"
- [results] PRJNA207663 result: "and 20 749 variants for PRJNA207663"
