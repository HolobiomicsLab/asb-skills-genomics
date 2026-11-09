---
name: slurm-hpc-script-generation
description: Use when a user supplies a project name, FASTQ file paths, and a node
  count for a Slurm-managed HPC cluster and needs the JWES pipeline auto-scripted
  and deployed as a batch job.
when_to_use_negative:
- The target cluster is not managed by Slurm (e.g., PBS/Torque, LSF) — the generated
  '#SBATCH' directives would not be honored.
- The user intends a local desktop run on Windows, macOS, or UNIX without a scheduler;
  JWES is cross-platform and can run directly without HPC deployment.
- The per-job resource needs exceed the 1 node / 8 CPUs / 46 GB profile (e.g., very
  large WGS cohorts requiring different node/CPU/memory allocations).
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: JWES
  role: Java-based pipeline application that auto-generates and deploys jwes_slurm_job.sh
    with Slurm resource directives
  repo: github.com/drzeeshanahmed/JWES-DB
- name: Java
  role: Implementation language of the JWES cross-platform application that performs
    the script generation
- name: FastQC
  role: First command in the generated script; QC on FASTQ/FASTA input
- name: Trimmomatic
  role: Second command in the generated script; trims low-quality sequences and filters
    adapters
- name: BWA
  role: Maps reads against the hg38 human reference genome producing SAM inside the
    generated script
- name: SortSam
  role: Sorts BWA's SAM output by reference sequence name and left-most mapping position
    in the generated script
- name: MarkDuplicates
  role: Locates, tags, and removes duplicate reads in the generated script
- name: SAMtools
  role: index tool invoked at the end of the generated script for fast access to the
    alignment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:900146090673c830361641c2ac9c6d699e057f181688d8c07489cf1ae6fc264c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# slurm-hpc-script-generation

## Summary

Automatically generates a Slurm batch script (jwes_slurm_job.sh) that deploys the JWES WGS/WES processing chain on an HPC cluster, embedding '#SBATCH' resource directives (1 node, 8 CPUs-per-task, 46 GB memory per job, matching the Rutgers OARC profile) ahead of the ordered FastQC -> Trimmomatic -> BWA -> SortSam -> MarkDuplicates -> SAMtools-index command chain.

## When to use

Use when a user supplies a project name, FASTQ file paths, and a node count for a Slurm-managed HPC cluster and needs the JWES pipeline auto-scripted and deployed as a batch job.

## When NOT to use

- The target cluster is not managed by Slurm (e.g., PBS/Torque, LSF) — the generated '#SBATCH' directives would not be honored.
- The user intends a local desktop run on Windows, macOS, or UNIX without a scheduler; JWES is cross-platform and can run directly without HPC deployment.
- The per-job resource needs exceed the 1 node / 8 CPUs / 46 GB profile (e.g., very large WGS cohorts requiring different node/CPU/memory allocations).

## Inputs

- JWES configuration file with filesystem paths to FastQC, Trimmomatic, BWA, SortSam, MarkDuplicates, and SAMtools
- User-supplied project name
- FASTQ/FASTA sample file paths
- Requested number of nodes
- Optional HPC login information

## Outputs

- jwes_slurm_job.sh — an executable Slurm batch script containing '#SBATCH' directives and the full FastQC -> Trimmomatic -> BWA -> SortSam -> MarkDuplicates -> SAMtools-index command chain
- Submitted Slurm job executing the JWES variant-calling pipeline (producing sorted BAM, indexed alignment, and variant calls)

## How to apply

Parse the JWES configuration file to collect the filesystem paths of the pipeline tools (FastQC, Trimmomatic, BWA, SortSam, MarkDuplicates, SAMtools index) plus the user-supplied project name, FASTQ file paths, and requested number of nodes. Assemble the ordered command chain: FastQC QC on FASTQ/FASTA input, Trimmomatic trimming of low-quality sequences and adapter filtering, BWA mapping against the hg38 human reference genome producing a SAM file, SortSam sorting by reference sequence name and left-most mapping position, MarkDuplicates to locate, tag, and remove duplicate reads, and the SAMtools index step for fast access. Prepend '#SBATCH' directives using the user's node count with per-job allocations consistent with the Rutgers OARC profile (1 node / 8 CPUs-per-task / 46 GB). Write the script to jwes_slurm_job.sh only after validating the optional HPC login information, project name, sample paths, and node count; the rationale is that JWES is a cross-platform Java application whose HPC deployments have been validated and reproduced under the Slurm Workload Manager at Rutgers University.

## Related tools

- **JWES** (Java-based pipeline application that auto-generates and deploys jwes_slurm_job.sh with Slurm resource directives) — github.com/drzeeshanahmed/JWES-DB
- **Java** (Implementation language of the JWES cross-platform application that performs the script generation)
- **FastQC** (First command in the generated script; QC on FASTQ/FASTA input)
- **Trimmomatic** (Second command in the generated script; trims low-quality sequences and filters adapters)
- **BWA** (Maps reads against the hg38 human reference genome producing SAM inside the generated script)
- **SortSam** (Sorts BWA's SAM output by reference sequence name and left-most mapping position in the generated script)
- **MarkDuplicates** (Locates, tags, and removes duplicate reads in the generated script)
- **SAMtools** (index tool invoked at the end of the generated script for fast access to the alignment)

## Examples

```
sbatch jwes_slurm_job.sh
```

## Evaluation signals

- jwes_slurm_job.sh exists in the working directory and parses as a valid shell script (e.g., bash -n jwes_slurm_job.sh).
- The script contains '#SBATCH' resource directives with allocations consistent with 1 node, 8 CPUs-per-task, and 46 GB of memory per job.
- The script contains the complete ordered command chain: FastQC -> Trimmomatic -> BWA -> SortSam -> MarkDuplicates -> SAMtools index.
- BWA in the script maps against the hg38 human reference genome.
- The script is only written after successful validation of user inputs (project name, sample paths, node count); a job submitted via sbatch reaches the RUNNING state on the Slurm cluster.

## Limitations

- JWES HPC deployment has been validated and reproduced in the Rutgers University Slurm environment; other HPC schedulers or Slurm site configurations are unverified.
- Script generation is conditioned on successful validation of user inputs (optional HPC login information, project name, sample paths, node count); invalid inputs abort script creation.
- The current JWES version offers only a command-line interface, with no interactive interface implemented.

## Evidence

- [methods] JWES functionality validated and reproduced in Rutgers HPC (Slurm): "Its functionality has been validated and reproduced multiple times within the HPC environment (deployed with Slurm Workload Manager) provided by the Rutgers University."
- [methods] FastQC starts the pipeline as a command-line tool: "The JWES pipeline starts with the FastQC, a command‐line‐based tool that takes FASTQ/FASTA files as input and performs QC"
- [methods] BWA maps against hg38 reference: "The Burrows–Wheeler Aligner (BWA) tool is then used for mapping sequence data against the reference human genome (hg38)"
- [methods] JWES is a cross-platform Java application integrating open-source command-line tools: "The JWES is a cross‐platform, hybrid, user‐friendly, and product line application that integrates multiple of the shelf open‐source command‐line tools."
