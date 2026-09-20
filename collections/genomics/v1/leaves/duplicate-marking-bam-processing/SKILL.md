---
name: duplicate-marking-bam-processing
description: Reference for tagging and removing PCR/sequencing duplicate reads in
  a sorted SAM/BAM alignment against hg38 prior to GATK variant calling, following
  BWA mapping of trimmed FASTQ reads.
when_to_use_negative:
- Input is still raw FASTQ that has not been QC-checked (FastQC) and adapter/quality-trimmed
  (Trimmomatic) and aligned by BWA — duplicate marking requires a sorted alignment
  file.
- The analysis goal is transcriptome (RNA-seq) processing rather than WGS/WES variant
  discovery; JWES is scoped to WGS and WES data.
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: BWA
  role: Produces the initial SAM alignment of trimmed reads against hg38 that feeds
    SortSam/MarkDuplicates
- name: SortSam
  role: Sorts the BWA SAM file by reference sequence name and left-most mapping position
    before duplicate marking
- name: MarkDuplicates
  role: Locates, tags, and removes duplicate reads to reduce variant false positives
- name: SAMtools
  role: Its index tool creates the index file for faster access after duplicate removal
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:37a5bb6c3f3c767b05ebc2b3a5764d6e6b0a829795d3ff2451a1c52e72dfd6b7
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
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# duplicate-marking-bam-processing

## Summary

Post-alignment duplicate handling in the JWES pipeline: sorted SAM/BAM output from BWA is passed to Picard/GATK SortSam (sorted by reference sequence name and left-most mapping position) and then MarkDuplicates to locate, tag, and remove duplicate reads, reducing variant false positives before SAMtools indexing.

## When to use

Reference for tagging and removing PCR/sequencing duplicate reads in a sorted SAM/BAM alignment against hg38 prior to GATK variant calling, following BWA mapping of trimmed FASTQ reads.

## When NOT to use

- Input is still raw FASTQ that has not been QC-checked (FastQC) and adapter/quality-trimmed (Trimmomatic) and aligned by BWA — duplicate marking requires a sorted alignment file.
- The analysis goal is transcriptome (RNA-seq) processing rather than WGS/WES variant discovery; JWES is scoped to WGS and WES data.

## Inputs

- SAM file produced by BWA mapping of trimmed FASTQ reads against hg38
- Human reference genome hg38

## Outputs

- Coordinate-sorted SAM/BAM with duplicate reads tagged and removed
- SAMtools index file (.bai) enabling faster random access to the BAM

## How to apply

After BWA maps Trimmomatic-trimmed FASTQ reads against the human reference genome hg38, the resulting SAM file is sorted by the reference sequence name and left-most mapping position with the SortSam tool. The sorted SAM is then fed to MarkDuplicates, which locates, tags, and removes duplicate reads to enhance alignment quality and reduce the number of variant false positives downstream in GATK HaplotypeCaller/VariantRecalibrator. Once duplicates are marked and removed, the SAMtools index tool is used to create an index file so the BAM can be accessed faster in later steps (IndelRealigner, BaseRecalibrator, HaplotypeCaller). In JWES this chain runs inside the data-processing module and was reproduced on HPC (Slurm) environments such as Rutgers University's.

## Related tools

- **BWA** (Produces the initial SAM alignment of trimmed reads against hg38 that feeds SortSam/MarkDuplicates)
- **SortSam** (Sorts the BWA SAM file by reference sequence name and left-most mapping position before duplicate marking)
- **MarkDuplicates** (Locates, tags, and removes duplicate reads to reduce variant false positives)
- **SAMtools** (Its index tool creates the index file for faster access after duplicate removal)

## Evaluation signals

- SortSam output is ordered by reference sequence name and left-most mapping position as stated in the JWES method chain.
- Duplicate reads are both tagged and removed by MarkDuplicates; downstream variant counts should show fewer false positives.
- A SAMtools index file is created for the duplicate-processed BAM, enabling fast access in subsequent GATK steps.
- The full chain (BWA -> SortSam -> MarkDuplicates -> index) reproduces the JWES data-processing module that was validated in the Rutgers HPC (Slurm) environment.
- Downstream HaplotypeCaller variant calls remain consistent with reported JWES cohort outputs (e.g. 43,685 variants for SRR12474733) when the whole pipeline is rerun.

## Limitations

- Duplicate removal reduces variant false positives but relies on the quality of the preceding alignment; misalignment can cause wrong duplicate calls.
- The JWES data-processing module is designed for FASTQ-based WGS/WES input and hg38 as reference; other references or data types are outside its scope.
- Running the full module requires substantial computational resources (HPC/cloud environments and bioinformatics manpower), as noted in the paper's discussion of big data analytics challenges.

## Evidence

- [methods] BWA produces a SAM file, which is then passed to the SortSam tool for sorting by the reference sequence name and left-most mapping position.: [evidence span withheld — anchor status: not_found]
- [methods] MarkDuplicates to locate, tag, and remove duplicate reads: [evidence span withheld — anchor status: not_found]
- [methods] enhance the quality of the alignment and reduce the number of variant false positives: "and enhance the quality of the alignment and reduce the number of variant false positives"
- [methods] the index tool from SAMtools is used to create and index file, allowing to access faster.: "Next, the index tool from SAMtools is used to create and index file, allowing to access faster."
- [methods] Its functionality has been validated and reproduced multiple times within the HPC environment (deployed with Slurm Workload Manager) provided by the Rutgers University.: "Its functionality has been validated and reproduced multiple times within the HPC environment (deployed with Slurm Workload Manager) provided by the Rutgers University."
- [readme] # github.com__drzeeshanahmed__JWES-DB: "# github.com__drzeeshanahmed__JWES-DB"
