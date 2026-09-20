---
name: variant-calling-pipeline-orchestration
description: Use when raw WGS/WES FASTQ files must be converted into annotated variant
  calls and the full tool chain (FastQC to HaplotypeCaller) plus HPC resource allocation
  needs to be scripted and launched automatically.
when_to_use_negative:
- The current JWES version offers only a command-line interface, so an interactive/GUI
  workflow is required
- Data are not WGS/WES FASTQ/FASTA sequence files (e.g. gene-expression matrices)
  — the pipeline targets sequence data for variant discovery
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3623
- http://edamontology.org/topic_3169
tools:
- name: JWES
  role: 'Java-based orchestrator that validates user inputs, generates jwes_slurm_job.sh
    with #SBATCH directives, and chains all pipeline steps'
  repo: github.com/drzeeshanahmed/JWES-DB
- name: FastQC
  role: first pipeline step; performs QC on FASTQ/FASTA input
- name: Trimmomatic
  role: trims low-quality sequences, filters adapters, and cuts bases after FastQC
- name: BWA
  role: maps trimmed reads against the hg38 human reference genome, producing SAM
    output
- name: SortSam
  role: sorts BWA's SAM output by reference sequence name and left-most mapping position
- name: MarkDuplicates
  role: locates, tags, and removes duplicate reads to enhance alignment quality and
    reduce variant false positives
- name: SAMtools
  role: index tool creates an index file allowing faster access to the alignment
- name: HaplotypeCaller
  role: calls SNPs and INDELs after realignment and recalibration
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:c976af1fd4e25334eb8f7156eaf1b8f9c1931a0170b1187fe410ceb3ac3c9ab7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA
  - FastQC
  - HaplotypeCaller
  - JWES
  - Java
  - MarkDuplicates
  - SAMtools
  - SortSam
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3623
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# variant-calling-pipeline-orchestration

## Summary

Orchestrates the JWES end-to-end WGS/WES variant-calling chain—FastQC QC, Trimmomatic trimming, BWA alignment to hg38, SortSam, MarkDuplicates, SAMtools indexing, and GATK realignment/recalibration/variant-calling steps—and deploys it as an auto-generated Slurm job on HPC.

## When to use

Use when raw WGS/WES FASTQ files must be converted into annotated variant calls and the full tool chain (FastQC to HaplotypeCaller) plus HPC resource allocation needs to be scripted and launched automatically.

## When NOT to use

- The current JWES version offers only a command-line interface, so an interactive/GUI workflow is required
- Data are not WGS/WES FASTQ/FASTA sequence files (e.g. gene-expression matrices) — the pipeline targets sequence data for variant discovery

## Inputs

- user-supplied FASTQ/FASTA sequence files (e.g. SRA downloads such as SRR12474733)
- JWES configuration file with tool paths, project name, and node count
- hg38 human reference genome
- optional HPC login information (validated before script generation)

## Outputs

- jwes_slurm_job.sh: shell script with #SBATCH directives and the full pipeline command chain
- SAM/BAM alignment files sorted by reference sequence name and left-most mapping position
- duplicate-tagged BAMs from MarkDuplicates
- SAMtools .bai index files
- VCF variant calls (SNPs and INDELs from HaplotypeCaller), consumable by the JWES-DB VCF parser

## How to apply

Parse the JWES configuration to collect tool paths (FastQC, Trimmomatic, BWA, SortSam, MarkDuplicates, SAMtools), the project name, user FASTQ paths, and requested node count. Assemble the ordered command chain: FastQC QC on FASTQ/FASTA input, Trimmomatic trimming of low-quality sequences and adapter filtering, BWA mapping against the hg38 human reference genome producing SAM, SortSam ordering by reference sequence name and left-most mapping position, MarkDuplicates locating/tagging/removing duplicate reads, then SAMtools index for fast access. Downstream GATK steps handle interval realignment (RealignerTargetCreator/IndelRealigner), base-quality recalibration (BaseRecalibrator, AnalyzeCovariates), HaplotypeCaller SNP/INDEL calling, and VariantRecalibrator/SelectVariants filtering. Prepend #SBATCH resource directives (1 node, 8 CPUs-per-task, 46 GB per job, matching the Rutgers OARC profile) and write the script to jwes_slurm_job.sh, only after validating optional HPC login info, project name, sample paths, and node count.

## Related tools

- **JWES** (Java-based orchestrator that validates user inputs, generates jwes_slurm_job.sh with #SBATCH directives, and chains all pipeline steps) — github.com/drzeeshanahmed/JWES-DB
- **FastQC** (first pipeline step; performs QC on FASTQ/FASTA input)
- **Trimmomatic** (trims low-quality sequences, filters adapters, and cuts bases after FastQC)
- **BWA** (maps trimmed reads against the hg38 human reference genome, producing SAM output)
- **SortSam** (sorts BWA's SAM output by reference sequence name and left-most mapping position)
- **MarkDuplicates** (locates, tags, and removes duplicate reads to enhance alignment quality and reduce variant false positives)
- **SAMtools** (index tool creates an index file allowing faster access to the alignment)
- **HaplotypeCaller** (calls SNPs and INDELs after realignment and recalibration)

## Examples

```
sbatch jwes_slurm_job.sh
```

## Evaluation signals

- jwes_slurm_job.sh exists, parses as a valid shell script, and contains '#SBATCH' resource directives
- The script contains the full FastQC -> Trimmomatic -> BWA -> SortSam -> MarkDuplicates -> SAMtools-index command chain in order
- Slurm directives reflect the Rutgers OARC job profile: 1 node, 8 CPUs-per-task, 46 GB memory per job, with node count as supplied by the user
- Pipeline run yields variant counts consistent with reported case studies (e.g. SRR12474733 of PRJNA657985 yielded 43,685 variants)
- Downstream outputs are parseable VCFs consumable by the JWES-DB VCF parser

## Limitations

- Functionality was validated only on Microsoft Windows, macOS Big Sur, and UNIX, and within the Rutgers University HPC environment deployed with the Slurm Workload Manager
- Per-job resource profile is fixed to the Rutgers OARC allocation (1 node / 8 CPUs / 46 GB); other HPC profiles need directive adjustments
- Script generation is conditioned on successful validation of user inputs (optional HPC login info, project name, sample paths, node count); invalid inputs block deployment
- Big data analytics at scale require manpower, computational resources (HPC/cloud), and bioinformatics applications

## Evidence

- [methods] The JWES pipeline starts with the FastQC, a command‐line‐based tool that takes FASTQ/FASTA files as input and performs QC: "The JWES pipeline starts with the FastQC, a command‐line‐based tool that takes FASTQ/FASTA files as input and performs QC"
- [methods] BWA produces a SAM file, which is then passed to the SortSam tool for sorting by the reference sequence name and left‐most mapping position.: [evidence span withheld — anchor status: not_found]
- [methods] Its functionality has been validated and reproduced multiple times within the HPC environment (deployed with Slurm Workload Manager) provided by the Rutgers University.: "Its functionality has been validated and reproduced multiple times within the HPC environment (deployed with Slurm Workload Manager) provided by the Rutgers University."
- [results] we applied the JWES to the WGS sample (SRR12474733) of project PRJNA657985, which resulted in 43 685 variants: "we applied the JWES to the WGS sample (SRR12474733) of project PRJNA657985, which resulted in 43 685 variants"
- [readme] VCF Parser for JWES: "# JWES-DB
VCF Parser for JWES"
