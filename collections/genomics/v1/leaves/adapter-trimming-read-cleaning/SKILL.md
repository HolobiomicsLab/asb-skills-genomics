---
name: adapter-trimming-read-cleaning
description: Use when raw FASTQ/FASTA reads have passed FastQC QC and need low-quality
  base trimming, adapter filtering, and base cutting before BWA alignment to hg38
  in a WGS/WES variant-calling workflow.
when_to_use_negative:
- Input is already aligned (SAM/BAM) — Trimmomatic operates on FASTQ reads only, before
  BWA mapping
- Reads have not yet undergone the FastQC quality check that starts the JWES pipeline
- The task is post-alignment cleanup (duplicate removal is handled later by MarkDuplicates,
  not Trimmomatic)
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Trimmomatic
  role: Performs trimming of low-quality sequences, filtering adapters, and base cutting
    on the FASTQ data after FastQC
- name: FastQC
  role: 'Upstream QC gate: takes FASTQ/FASTA files as input and performs the quality
    check that starts the JWES pipeline before trimming'
- name: BWA
  role: 'Downstream consumer: maps the Trimmomatic-processed reads against the reference
    human genome hg38'
- name: JWES
  role: Pipeline that chains FastQC -> Trimmomatic -> BWA; JWES-DB VCF parser is at
    the referenced repo
  repo: github.com/drzeeshanahmed/JWES-DB
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:a7c04a7c39468da584144fda42a509d8f4004004667b27ab487ec6c14854145d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA
  - FastQC
  - JWES
  - MarkDuplicates
  - SAMtools
  - SortSam
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Adapter trimming and read cleaning with Trimmomatic

## Summary

Performs the second stage of the JWES pipeline: raw FASTQ reads that have passed the initial FastQC quality check are cleaned with Trimmomatic by trimming low-quality sequences, filtering out adapters, and base cutting, producing processed FASTQ ready for BWA mapping against hg38.

## When to use

Use when raw FASTQ/FASTA reads have passed FastQC QC and need low-quality base trimming, adapter filtering, and base cutting before BWA alignment to hg38 in a WGS/WES variant-calling workflow.

## When NOT to use

- Input is already aligned (SAM/BAM) — Trimmomatic operates on FASTQ reads only, before BWA mapping
- Reads have not yet undergone the FastQC quality check that starts the JWES pipeline
- The task is post-alignment cleanup (duplicate removal is handled later by MarkDuplicates, not Trimmomatic)

## Inputs

- Raw FASTQ files (optionally FASTA) that have completed the initial FastQC QC step
- Adapter sequence definitions for adapter filtering

## Outputs

- Processed (trimmed, adapter-filtered, base-cut) FASTQ files ready for BWA mapping against hg38

## How to apply

After FastQC confirms data quality on the raw FASTQ files, forward the data (FASTQ) to Trimmomatic to trim low-quality sequences, filter adapters, and perform base cutting, yielding processed FASTQ output. This step precedes BWA mapping against the human reference genome hg38, so the processed reads must remain in FASTQ format with paired-end structure intact. Decision point: only reads that pass the FastQC QC step should enter trimming; the trimmed FASTQ is then handed directly to BWA. Judge success by whether the trimmed FASTQ proceeds cleanly through the downstream chain (BWA -> SortSam -> MarkDuplicates -> SAMtools index -> GATK realignment, BaseRecalibrator, HaplotypeCaller) without adapter contamination driving false-positive variant calls — a motivation the article explicitly attributes to duplicate removal and quality improvement steps.

## Related tools

- **Trimmomatic** (Performs trimming of low-quality sequences, filtering adapters, and base cutting on the FASTQ data after FastQC)
- **FastQC** (Upstream QC gate: takes FASTQ/FASTA files as input and performs the quality check that starts the JWES pipeline before trimming)
- **BWA** (Downstream consumer: maps the Trimmomatic-processed reads against the reference human genome hg38)
- **JWES** (Pipeline that chains FastQC -> Trimmomatic -> BWA; JWES-DB VCF parser is at the referenced repo) — github.com/drzeeshanahmed/JWES-DB

## Examples

```
jwes --input raw.fastq --qc FastQC --trim Trimmomatic --ref hg38
```

## Evaluation signals

- FastQC output on raw FASTQ/FASTA input confirms the QC gate passed before trimming began
- Output remains valid FASTQ consumable by BWA against hg38 with no adapter contamination
- Downstream chain (SortSam, MarkDuplicates, GATK) completes without errors attributable to malformed trimmed reads
- Variant false positives are reduced in the final call set, consistent with the pipeline's quality-enhancement rationale
- Whole JWES run is reproducible in an HPC environment (e.g., Rutgers Slurm deployment), as the paper reports repeated successful validation

## Limitations

- The current version of JWES offers only a command-line interface; an easy-to-use interactive interface is not yet implemented
- Trimmomatic in JWES only performs trimming/adapter filtering/base cutting — quality improvement against duplicates is done later by MarkDuplicates
- Trimmomatic is specific to the FASTQ stage; it cannot fix problems introduced after mapping (e.g., indel misalignments, which GATK IndelRealigner addresses)

## Evidence

- [methods] The JWES pipeline starts with the FastQC, a command‐line‐based tool that takes FASTQ/FASTA files as input and performs QC: "The JWES pipeline starts with the FastQC, a command‐line‐based tool that takes FASTQ/FASTA files as input and performs QC"
- [methods] The data (FASTQ) are then forwarded to the Trimmomatic tool for the trimming of low‐quality sequences, filtering adapters, and base cutting.: [evidence span withheld — anchor status: not_found]
- [other] Trim the reads with Trimmomatic to cut low-quality sequences, filter adapters, and perform base cutting, yielding processed FASTQ.: [evidence span withheld — anchor status: not_found]
- [readme] # github.com__drzeeshanahmed__JWES-DB: "# github.com__drzeeshanahmed__JWES-DB"
