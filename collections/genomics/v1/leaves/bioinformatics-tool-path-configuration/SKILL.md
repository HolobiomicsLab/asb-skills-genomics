---
name: bioinformatics-tool-path-configuration
description: Use when generating or reproducing a pipeline job script that must call
  external command-line tools by absolute path (e.g. building a Slurm submission for
  a WGS/WES pipeline) and tool install locations must be resolved from a config file
  before script generation.
when_to_use_negative:
- When the pipeline runs locally on Windows/macOS rather than on a Slurm-managed HPC
  cluster — JWES is cross-platform, but Slurm directives and the HPC job profile (1
  node / 8 CPUs / 46 GB) only apply to the Slurm deployment path.
- When a workflow manager (e.g. Nextflow/Snakemake) already resolves tool paths from
  environment modules or containers; manual path configuration would be redundant.
- When user inputs (project name, sample paths, node count) have not been validated
  — the JWES approach conditions script generation on successful validation.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: FastQC
  role: Command-line tool whose installed path is resolved from the JWES config and
    invoked as the first QC step of the generated script
- name: Trimmomatic
  role: Tool invoked by path from the generated script for trimming low-quality sequences,
    filtering adapters, and base cutting
- name: BWA
  role: Tool invoked by path from the generated script to map reads against the hg38
    human reference genome, producing SAM
- name: SortSam
  role: Tool invoked by path from the generated script to sort SAM by reference sequence
    name and left-most mapping position
- name: MarkDuplicates
  role: Tool invoked by path from the generated script to locate, tag, and remove
    duplicate reads
- name: SAMtools
  role: Its index tool is invoked by path from the generated script to create an index
    for faster access
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:d4be50eeaa886b2dd190ba412ac3981e61cdcf30a3e34119f6d23581721b209d
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# bioinformatics-tool-path-configuration

## Summary

Resolves the filesystem paths of external bioinformatics executables (FastQC, Trimmomatic, BWA, SortSam, MarkDuplicates, SAMtools) from a pipeline configuration file and wires them, together with user inputs, into an auto-generated Slurm submission script so the pipeline runs reproducibly on HPC.

## When to use

Use when generating or reproducing a pipeline job script that must call external command-line tools by absolute path (e.g. building a Slurm submission for a WGS/WES pipeline) and tool install locations must be resolved from a config file before script generation.

## When NOT to use

- When the pipeline runs locally on Windows/macOS rather than on a Slurm-managed HPC cluster — JWES is cross-platform, but Slurm directives and the HPC job profile (1 node / 8 CPUs / 46 GB) only apply to the Slurm deployment path.
- When a workflow manager (e.g. Nextflow/Snakemake) already resolves tool paths from environment modules or containers; manual path configuration would be redundant.
- When user inputs (project name, sample paths, node count) have not been validated — the JWES approach conditions script generation on successful validation.

## Inputs

- JWES configuration file listing filesystem paths of pipeline tools (FastQC, Trimmomatic, BWA, SortSam, MarkDuplicates, SAMtools)
- User-supplied project name
- FASTQ sample file paths
- Requested Slurm node count
- Optional HPC login information
- Human reference genome hg38 path for BWA mapping

## Outputs

- jwes_slurm_job.sh — a shell script containing '#SBATCH' directives and the FastQC -> Trimmomatic -> BWA -> SortSam -> MarkDuplicates -> SAMtools-index command chain

## How to apply

Parse the JWES configuration file to collect the filesystem paths of the pipeline tools (FastQC, Trimmomatic, BWA, SortSam, MarkDuplicates, SAMtools index) plus the user-supplied project name, FASTQ file paths, and requested number of nodes. Assemble the ordered command chain in that sequence: FastQC on FASTQ/FASTA input, Trimmomatic trimming, BWA mapping against hg38 producing SAM, SortSam sorting by reference sequence name and left-most mapping position, MarkDuplicates locating/tagging/removing duplicates, then SAMtools index. Prepend '#SBATCH' resource directives using the user's node count and CPU/memory allocations consistent with the Rutgers OARC profile of 1 node / 8 CPUs-per-task / 46 GB per job. Only write jwes_slurm_job.sh after validating user inputs (optional HPC login information, project name, sample paths, node count), so that an invalid configuration never produces a deployable script.

## Related tools

- **FastQC** (Command-line tool whose installed path is resolved from the JWES config and invoked as the first QC step of the generated script)
- **Trimmomatic** (Tool invoked by path from the generated script for trimming low-quality sequences, filtering adapters, and base cutting)
- **BWA** (Tool invoked by path from the generated script to map reads against the hg38 human reference genome, producing SAM)
- **SortSam** (Tool invoked by path from the generated script to sort SAM by reference sequence name and left-most mapping position)
- **MarkDuplicates** (Tool invoked by path from the generated script to locate, tag, and remove duplicate reads)
- **SAMtools** (Its index tool is invoked by path from the generated script to create an index for faster access)

## Examples

```
jwes --config jwes_tools.cfg --project PRJNA657985 --fq1 SRR12474733_1.fastq --fq2 SRR12474733_2.fastq --nodes 1  # writes jwes_slurm_job.sh with #SBATCH directives (1 node / 8 CPUs / 46 GB)
```

## Evaluation signals

- jwes_slurm_job.sh exists and parses as a valid shell script
- The generated script contains '#SBATCH' resource directives
- The script contains the full FastQC -> Trimmomatic -> BWA -> SortSam -> MarkDuplicates -> SAMtools-index command chain in that order
- Resource directives match the requested node count and the 1 node / 8 CPUs-per-task / 46 GB per-job profile
- Script generation is suppressed when user input validation (project name, sample paths, node count) fails

## Limitations

- Resource allocation (1 node / 8 CPUs / 46 GB) is tuned to the Rutgers OARC HPC environment; other clusters may need different profiles.
- Tool paths are resolved from a manually maintained configuration file, so stale or wrong paths silently break the generated script.
- The HPC deployment was validated only in the Rutgers Slurm environment; JWES functionality was validated on Windows, macOS Big Sur, and UNIX, but the Slurm auto-generation is environment-specific.

## Evidence

- [methods] Slurm config + pipeline chain: "Its functionality has been validated and reproduced multiple times within the HPC environment (deployed with Slurm Workload Manager) provided by the Rutgers University."
- [methods] pipeline order: "The JWES pipeline starts with the FastQC, a command‐line‐based tool that takes FASTQ/FASTA files as input and performs QC"
- [methods] BWA mapping: "The Burrows–Wheeler Aligner (BWA) tool is then used for mapping sequence data against the reference human genome (hg38)"
- [methods] index step: "Next, the index tool from SAMtools is used to create and index file, allowing to access faster."
- [other] tool invocation: "The JWES is a cross‐platform, hybrid, user‐friendly, and product line application that integrates multiple of the shelf open‐source command‐line tools."
