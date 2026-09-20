---
name: wgs-pipeline-command-chain-authoring
description: Use when a WGS/WES pipeline must be scripted for deployment on a Slurm-managed
  HPC cluster, requiring a fixed tool command chain plus per-job node/CPU/memory resource
  directives.
when_to_use_negative:
- The target environment is not a Slurm workload manager (e.g. a local desktop run
  — JWES itself was validated cross-platform on Windows, macOS Big Sur, and UNIX outside
  HPC)
- 'Only command-line interface execution without cluster job scheduling is needed,
  since #SBATCH directives and node/CPU/memory allocations would be irrelevant'
tools:
- name: JWES
  role: Pipeline application whose auto-generated jwes_slurm_job.sh this skill reproduces
  repo: https://github.com/drzeeshanahmed/JWES-DB
- name: Java
  role: Implementation language of the JWES pipeline being scripted
- name: FastQC
  role: First stage of the authored command chain; QC of FASTQ/FASTA input
- name: Trimmomatic
  role: Second stage; trims low-quality sequences, filters adapters, and cuts bases
- name: BWA
  role: Third stage; maps reads against the hg38 human reference genome producing
    SAM
- name: SortSam
  role: Fourth stage; sorts SAM by reference sequence name and left-most mapping position
- name: MarkDuplicates
  role: Fifth stage; locates, tags, and removes duplicate reads to reduce variant
    false positives
- name: SAMtools
  role: Final stage; index tool creates an index file allowing faster access
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:a5acefb0255af9dd4e48e7e353c21f5e02da8f2352c903ba3e17e891927826c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drzeeshanahmed/JWES-DB
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# wgs-pipeline-command-chain-authoring

## Summary

Method for automatically authoring a Slurm-executable shell script (jwes_slurm_job.sh) that chains the JWES WGS/WES processing tools—FastQC, Trimmomatic, BWA, SortSam, MarkDuplicates, and SAMtools index—under #SBATCH resource directives matching a fixed HPC job profile.

## When to use

Use when a WGS/WES pipeline must be scripted for deployment on a Slurm-managed HPC cluster, requiring a fixed tool command chain plus per-job node/CPU/memory resource directives.

## When NOT to use

- The target environment is not a Slurm workload manager (e.g. a local desktop run — JWES itself was validated cross-platform on Windows, macOS Big Sur, and UNIX outside HPC)
- Only command-line interface execution without cluster job scheduling is needed, since #SBATCH directives and node/CPU/memory allocations would be irrelevant

## Inputs

- JWES configuration file with tool paths (FastQC, Trimmomatic, BWA, SortSam, MarkDuplicates, SAMtools)
- User-supplied project name
- FASTQ (or FASTA) input file paths
- Requested number of Slurm nodes
- Optional HPC login information

## Outputs

- jwes_slurm_job.sh — a Slurm-ready shell script containing #SBATCH directives and the FastQC → Trimmomatic → BWA → SortSam → MarkDuplicates → SAMtools-index command chain
- SAM file sorted by reference sequence name and left-most mapping position (intermediate from BWA/SortSam)
- Indexed BAM for fast downstream access (from SAMtools index)

## How to apply

First parse the pipeline configuration file to collect tool paths (FastQC, Trimmomatic, BWA, SortSam, MarkDuplicates, SAMtools index) and user-supplied values: project name, FASTQ file paths, and requested node count. Then assemble the ordered command chain: FastQC QC on FASTQ/FASTA input, Trimmomatic trimming of low-quality sequences and adapter filtering, BWA mapping against the hg38 human reference genome producing SAM, SortSam sorting by reference sequence name and left-most mapping position, MarkDuplicates to locate, tag, and remove duplicate reads, and SAMtools index for fast access. Prepend #SBATCH directives using the user-supplied node count plus the Rutgers OARC job profile of 1 node, 8 CPUs-per-task, and 46 GB of memory per job. Write the script to jwes_slurm_job.sh only after validating optional HPC login information, project name, sample paths, and node count.

## Related tools

- **JWES** (Pipeline application whose auto-generated jwes_slurm_job.sh this skill reproduces) — https://github.com/drzeeshanahmed/JWES-DB
- **Java** (Implementation language of the JWES pipeline being scripted)
- **FastQC** (First stage of the authored command chain; QC of FASTQ/FASTA input)
- **Trimmomatic** (Second stage; trims low-quality sequences, filters adapters, and cuts bases)
- **BWA** (Third stage; maps reads against the hg38 human reference genome producing SAM)
- **SortSam** (Fourth stage; sorts SAM by reference sequence name and left-most mapping position)
- **MarkDuplicates** (Fifth stage; locates, tags, and removes duplicate reads to reduce variant false positives)
- **SAMtools** (Final stage; index tool creates an index file allowing faster access)

## Examples

```
sbatch jwes_slurm_job.sh
```

## Evaluation signals

- jwes_slurm_job.sh exists after generation
- jwes_slurm_job.sh parses as a valid shell script (e.g. sh -n or bash -n passes)
- Script contains '#SBATCH' resource directives with the 1 node / 8 CPUs-per-task / 46 GB job profile
- Script contains the full FastQC → Trimmomatic → BWA → SortSam → MarkDuplicates → SAMtools-index command chain in that order
- BWA stage references the hg38 human reference genome and produces a SAM file for the SortSam stage

## Limitations

- Resource profile is fixed to the Rutgers OARC job shape (1 node / 8 CPUs / 46 GB per job); only the node count is user-supplied
- The current JWES version offers only a command-line interface; no interactive interface is implemented
- Generation is conditioned on validation of user inputs; invalid project name, sample paths, or node count blocks script writing
- The linked repository (JWES-DB) is a VCF parser for JWES, not the script generator itself

## Evidence

- [methods] Its functionality has been validated and reproduced multiple times within the HPC environment (deployed with Slurm Workload Manager) provided by the Rutgers University.: "Its functionality has been validated and reproduced multiple times within the HPC environment (deployed with Slurm Workload Manager) provided by the Rutgers University."
- [methods] The JWES pipeline starts with the FastQC, a command‐line‐based tool that takes FASTQ/FASTA files as input and performs QC: "The JWES pipeline starts with the FastQC, a command‐line‐based tool that takes FASTQ/FASTA files as input and performs QC"
- [methods] BWA produces a SAM file, which is then passed to the SortSam tool for sorting by the reference sequence name and left‐most mapping position.: [evidence span withheld — anchor status: not_found]
- [methods] The Burrows–Wheeler Aligner (BWA) tool is then used for mapping sequence data against the reference human genome (hg38): "The Burrows–Wheeler Aligner (BWA) tool is then used for mapping sequence data against the reference human genome (hg38)"
