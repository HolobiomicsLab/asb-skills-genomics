---
name: fastq-quality-control
description: Use when raw FASTQ or FASTA reads (e.g. SRA downloads such as SRR891275)
  are the starting point of a WGS/WES variant-calling workflow and per-read QC must
  precede trimming and alignment.
when_to_use_negative:
- Input is already trimmed/processed FASTQ that has passed QC and is ready for BWA
  alignment to hg38
- Input is a sorted SAM/BAM file or a VCF — FastQC in JWES only accepts FASTQ/FASTA
  read input at the start of the pipeline
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: FastQC
  role: Command-line tool that starts the JWES pipeline by taking FASTQ/FASTA files
    as input and performing the initial data quality check (QC)
- name: Trimmomatic
  role: Downstream trimming stage that consumes the FastQC-assessed FASTQ, cutting
    low-quality bases and filtering adapters
- name: JWES
  role: Java-based WGS/WES pipeline whose first stage is the FastQC quality check
    before Trimmomatic, BWA, GATK, and snpEff steps
  repo: github.com/drzeeshanahmed/JWES-DB
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:956269e94085b113371a9be8d75f496a9168e9a5485b702b19e2b30b89b5402f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA
  - FastQC
  - JWES
  - Java
  - MarkDuplicates
  - SAMtools
  - SortSam
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# fastq-quality-control

## Summary

Initial quality check of raw FASTQ/FASTA sequencing reads using the command-line FastQC tool, which starts the JWES whole genome/exome pipeline before trimming and alignment.

## When to use

Use when raw FASTQ or FASTA reads (e.g. SRA downloads such as SRR891275) are the starting point of a WGS/WES variant-calling workflow and per-read QC must precede trimming and alignment.

## When NOT to use

- Input is already trimmed/processed FASTQ that has passed QC and is ready for BWA alignment to hg38
- Input is a sorted SAM/BAM file or a VCF — FastQC in JWES only accepts FASTQ/FASTA read input at the start of the pipeline

## Inputs

- Raw FASTQ sequencing read files (e.g. paired-end FASTQ downloaded from NCBI SRA, sample SRR891275)
- FASTA files (also accepted by FastQC as input)

## Outputs

- FastQC per-sample quality-control reports used to assess read quality before Trimmomatic trimming

## How to apply

Run FastQC on the raw FASTQ/FASTA files as the first stage of the pipeline: it takes FASTQ/FASTA as input and performs the data quality check (QC) before any downstream processing. Inspect the FastQC reports to decide whether trimming is needed; in the JWES workflow the QC step feeds directly into Trimmomatic, which trims low-quality sequences, filters adapters, and cuts bases to yield processed FASTQ. In the published reproducibility case study, this QC step was applied to SRR891275 downloaded from the NCBI SRA database (project PRJNA207663) before BWA alignment to hg38. The rationale is that base-quality and adapter problems detected at this stage would propagate into false-positive variant calls if left uncorrected, so QC gates the rest of the chain.

## Related tools

- **FastQC** (Command-line tool that starts the JWES pipeline by taking FASTQ/FASTA files as input and performing the initial data quality check (QC))
- **Trimmomatic** (Downstream trimming stage that consumes the FastQC-assessed FASTQ, cutting low-quality bases and filtering adapters)
- **JWES** (Java-based WGS/WES pipeline whose first stage is the FastQC quality check before Trimmomatic, BWA, GATK, and snpEff steps) — github.com/drzeeshanahmed/JWES-DB

## Examples

```
fastqc SRR891275_1.fastq.gz SRR891275_2.fastq.gz
```

## Evaluation signals

- FastQC runs successfully on the raw FASTQ/FASTA files without parse errors, producing per-sample QC reports for every read file
- QC reports justify the Trimmomatic parameters used next (low-quality sequence trimming, adapter filtering, base cutting) on the same FASTQ files
- Downstream BWA alignment accepts the post-QC/post-trim FASTQ without input-format failures
- In the reproducibility benchmark, the full chain (QC → trim → align → GATK → snpEff) reproduces the reported variant count for the sample (e.g. 20,749 variants for SRR891275 / PRJNA207663)

## Limitations

- FastQC only reports on quality; it does not repair the reads — trimming and adapter removal must be done by the subsequent Trimmomatic stage
- In the JWES workflow, QC is the first stage of the WGS/WES chain and does not itself perform alignment, variant calling, or annotation

## Evidence

- [methods] FastQC starts the JWES pipeline with a QC check on FASTQ/FASTA input: "The JWES pipeline starts with the FastQC, a command‐line‐based tool that takes FASTQ/FASTA files as input and performs QC"
- [methods] Trimmomatic consumes the FASTQ after QC to trim low-quality sequences, filter adapters, and cut bases: [evidence span withheld — anchor status: not_found]
- [results] The SRR891275 sample used in the reproducibility case study was downloaded from NCBI SRA: "we downloaded SRR891275 from NCBI Sequence Read Architecture (SRA) database"
- [readme] README of the JWES-DB repository describing the repo as a VCF parser for JWES: "JWES-DB VCF Parser for JWES"
