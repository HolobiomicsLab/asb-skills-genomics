---
name: fastq-to-alignment-workflow-staging
description: Use when deploying the JWES FASTQ-to-alignment pipeline on a Slurm-managed
  HPC and a job script must be auto-generated from user inputs (project name, FASTQ
  paths, node count) with '#SBATCH' directives.
when_to_use_negative:
- Analysis target is already an aligned, sorted, duplicate-marked BAM with an index
  (i.e., the FASTQ-to-alignment staging is done)
- The execution environment is not a Slurm-managed cluster — JWES is cross-platform
  (Microsoft Windows, macOS Big Sur, UNIX) and can run without '#SBATCH' directives
- The task is downstream variant annotation, prediction, or genotyping rather than
  staging the alignment-stage workflow
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: JWES
  role: Pipeline application whose auto-generation logic writes and deploys the Slurm
    job script
  repo: github.com/drzeeshanahmed/JWES-DB
- name: FastQC
  role: 'First staged step: QC of FASTQ/FASTA input files'
- name: Trimmomatic
  role: 'Second staged step: trimming low-quality sequences, filtering adapters, base
    cutting'
- name: BWA
  role: 'Third staged step: maps sequence data against the hg38 human reference genome,
    producing SAM'
- name: SortSam
  role: 'Fourth staged step: sorts SAM by reference sequence name and left-most mapping
    position'
- name: MarkDuplicates
  role: 'Fifth staged step: locates, tags, and removes duplicate reads to reduce variant
    false positives'
- name: SAMtools
  role: 'Final staged step: index tool creates an index file allowing faster access'
- name: Slurm
  role: Workload manager receiving '#SBATCH' resource directives in the generated
    script
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:024ef4b128ce0f5f98cba01e880c57e2a95d646832daaf53c354a63c90968b34
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
  - Slurm
  - SortSam
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# fastq-to-alignment-workflow-staging

## Summary

Generates a Slurm batch script (jwes_slurm_job.sh) that stages and chains the JWES whole genome/exome processing steps — FastQC QC, Trimmomatic trimming, BWA alignment to hg38, SortSam, MarkDuplicates, and SAMtools indexing — with per-job resource directives of 1 node, 8 CPUs, and 46 GB memory as validated at Rutgers OARC.

## When to use

Use when deploying the JWES FASTQ-to-alignment pipeline on a Slurm-managed HPC and a job script must be auto-generated from user inputs (project name, FASTQ paths, node count) with '#SBATCH' directives.

## When NOT to use

- Analysis target is already an aligned, sorted, duplicate-marked BAM with an index (i.e., the FASTQ-to-alignment staging is done)
- The execution environment is not a Slurm-managed cluster — JWES is cross-platform (Microsoft Windows, macOS Big Sur, UNIX) and can run without '#SBATCH' directives
- The task is downstream variant annotation, prediction, or genotyping rather than staging the alignment-stage workflow

## Inputs

- JWES configuration file with filesystem paths of FastQC, Trimmomatic, BWA, SortSam, MarkDuplicates, and SAMtools
- User-supplied project name
- FASTQ (or FASTA) input file paths
- Requested number of Slurm nodes
- Optional HPC login information

## Outputs

- jwes_slurm_job.sh — Slurm batch script containing '#SBATCH' directives (nodes, 8 CPUs-per-task, 46 GB) and the FastQC -> Trimmomatic -> BWA -> SortSam -> MarkDuplicates -> SAMtools-index command chain
- Slurm-deployed pipeline run producing QC reports, trimmed FASTQ, SAM/BAM alignments against hg38, duplicate-marked alignments, and alignment index files

## How to apply

Parse the JWES configuration to collect filesystem paths of the pipeline tools (FastQC, Trimmomatic, BWA, SortSam, MarkDuplicates, SAMtools index) plus the user-supplied project name, FASTQ file paths, and requested node count; validate these inputs (optional HPC login information, project name, sample paths, node count) before writing anything. Assemble the ordered command chain: FastQC QC on FASTQ/FASTA input, Trimmomatic trimming of low-quality sequences and adapter filtering, BWA mapping against the hg38 human reference genome producing SAM, SortSam ordering by reference sequence name and left-most mapping position, MarkDuplicates to locate, tag, and remove duplicate reads, then the SAMtools index step for faster access. Prepend Slurm '#SBATCH' resource directives, using the user-supplied node count with CPU and memory allocations consistent with the Rutgers OARC job profile of 1 node / 8 CPUs-per-task / 46 GB memory per job. Write the complete script to jwes_slurm_job.sh only after validation succeeds; this staging matters because JWES functionality was validated and reproduced multiple times within the Rutgers Slurm-deployed HPC environment.

## Related tools

- **JWES** (Pipeline application whose auto-generation logic writes and deploys the Slurm job script) — github.com/drzeeshanahmed/JWES-DB
- **FastQC** (First staged step: QC of FASTQ/FASTA input files)
- **Trimmomatic** (Second staged step: trimming low-quality sequences, filtering adapters, base cutting)
- **BWA** (Third staged step: maps sequence data against the hg38 human reference genome, producing SAM)
- **SortSam** (Fourth staged step: sorts SAM by reference sequence name and left-most mapping position)
- **MarkDuplicates** (Fifth staged step: locates, tags, and removes duplicate reads to reduce variant false positives)
- **SAMtools** (Final staged step: index tool creates an index file allowing faster access)
- **Slurm** (Workload manager receiving '#SBATCH' resource directives in the generated script)

## Evaluation signals

- jwes_slurm_job.sh exists and parses as a valid shell script
- The script contains '#SBATCH' resource directives including nodes, CPUs-per-task=8, and memory=46 GB consistent with the Rutgers OARC job profile
- The script contains the full ordered command chain FastQC -> Trimmomatic -> BWA -> SortSam -> MarkDuplicates -> SAMtools index
- BWA reference is the hg38 human genome and BWA output is a SAM file passed onward to SortSam
- Script generation was conditioned on successful validation of user inputs (project name, sample paths, node count)

## Limitations

- The current version of JWES offers a command-line interface only; an easy-to-use interactive interface is not yet implemented
- Resource profile is tied to the Rutgers University HPC (Slurm) environment; other sites may need different node/CPU/memory allocations
- User-supplied node count is honored, but CPU (8) and memory (46 GB) per job are fixed in the generated directives

## Evidence

- [methods] FastQC first: "The JWES pipeline starts with the FastQC, a command‐line‐based tool that takes FASTQ/FASTA files as input and performs QC"
- [methods] BWA mapping: "The Burrows–Wheeler Aligner (BWA) tool is then used for mapping sequence data against the reference human genome (hg38)"
- [methods] SAMtools index: "Next, the index tool from SAMtools is used to create and index file, allowing to access faster."
- [methods] Slurm validation: "Its functionality has been validated and reproduced multiple times within the HPC environment (deployed with Slurm Workload Manager) provided by the Rutgers University."
- [other] Slurm directives: [evidence span withheld — anchor status: not_found]
