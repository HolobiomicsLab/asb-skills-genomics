---
name: short-read-alignment-to-reference-genome
description: Use when raw FASTQ/FASTA reads from WGS or WES samples must be aligned
  to a reference genome (e.g., hg38) to prepare sorted, duplicate-marked, indexed
  BAM files for variant calling.
when_to_use_negative:
- Input is already a sorted, duplicate-marked, indexed BAM — the alignment step would
  only duplicate work
- Input is a VCF or annotated variant file (e.g., a JWES-DB parse target) rather than
  sequence reads
- The task is de novo assembly rather than mapping to a known reference genome
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BWA
  role: Maps the Trimmomatic-processed sequence data against the reference human genome
    hg38, producing a SAM file
- name: SortSam
  role: Sorts the BWA-produced SAM file by reference sequence name and left-most mapping
    position
- name: MarkDuplicates
  role: Locates, tags, and removes duplicate reads in the sorted SAM file to reduce
    variant false positives
- name: SAMtools
  role: Its index tool creates an index file for the BAM, allowing faster access
- name: Trimmomatic
  role: 'Upstream: trimming of low-quality sequences, filtering adapters, and base
    cutting before alignment'
- name: JWES
  role: Cross-platform Java-based pipeline that orchestrates the alignment and post-alignment
    steps
  repo: github.com/drzeeshanahmed/JWES-DB
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:c97bcf695c8f32ce0856911ce9c5a86802e775036a6a7be70e20ab322972125f
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# short-read-alignment-to-reference-genome

## Summary

Maps processed WGS/WES reads against the human reference genome hg38 using BWA within the JWES pipeline, then converts, sorts, deduplicates, and indexes the alignment to produce an analysis-ready BAM for downstream GATK variant calling.

## When to use

Use when raw FASTQ/FASTA reads from WGS or WES samples must be aligned to a reference genome (e.g., hg38) to prepare sorted, duplicate-marked, indexed BAM files for variant calling.

## When NOT to use

- Input is already a sorted, duplicate-marked, indexed BAM — the alignment step would only duplicate work
- Input is a VCF or annotated variant file (e.g., a JWES-DB parse target) rather than sequence reads
- The task is de novo assembly rather than mapping to a known reference genome

## Inputs

- Processed (trimmed/adapter-filtered) FASTQ reads, e.g., SRR891275 from NCBI SRA project PRJNA207663
- Human reference genome hg38

## Outputs

- SAM file from BWA
- BAM sorted by reference sequence name and left-most mapping position (SortSam)
- BAM with duplicates located, tagged, and removed (MarkDuplicates)
- BAM index file created with SAMtools index

## How to apply

First trim low-quality sequences, filter adapters, and cut bases from the FASTQ input with Trimmomatic (after an initial FastQC check), then map the processed reads to hg38 with BWA. Because BWA produces a SAM file, pass it to SortSam to sort by reference sequence name and left-most mapping position. Run MarkDuplicates on the sorted file to locate, tag, and remove duplicate reads — this enhances alignment quality and reduces variant false positives — and use the SAMtools index tool to create an index file that allows faster access. The resulting indexed BAM feeds directly into GATK realignment (RealignerTargetCreator/IndelRealigner) and BaseRecalibrator quality-score adjustment in the JWES workflow.

## Related tools

- **BWA** (Maps the Trimmomatic-processed sequence data against the reference human genome hg38, producing a SAM file)
- **SortSam** (Sorts the BWA-produced SAM file by reference sequence name and left-most mapping position)
- **MarkDuplicates** (Locates, tags, and removes duplicate reads in the sorted SAM file to reduce variant false positives)
- **SAMtools** (Its index tool creates an index file for the BAM, allowing faster access)
- **Trimmomatic** (Upstream: trimming of low-quality sequences, filtering adapters, and base cutting before alignment)
- **JWES** (Cross-platform Java-based pipeline that orchestrates the alignment and post-alignment steps) — github.com/drzeeshanahmed/JWES-DB

## Evaluation signals

- FastQC quality check on the input FASTQ/FASTA passes before alignment is attempted
- BWA output is a valid SAM file that SortSam can sort by reference sequence name and left-most mapping position
- Duplicate reads are located and tagged/removed by MarkDuplicates without breaking downstream processing
- A SAMtools index (.bai) is successfully created for the final BAM, enabling fast access
- The indexed BAM proceeds without errors through GATK RealignerTargetCreator/IndelRealigner and HaplotypeCaller, and the published JWES run on SRR891275 (PRJNA207663) is reproducible (20,749 variants total)

## Limitations

- The alignment stage depends on upstream Trimmomatic trimming; untrimmed low-quality reads and adapters degrade mapping
- JWES was validated in a specific HPC environment (Rutgers University, Slurm Workload Manager) and on Microsoft Windows, macOS Big Sur, and UNIX; other environments are not reported
- JWES currently offers only a command-line interface; no interactive interface is implemented yet

## Evidence

- [methods] The Burrows–Wheeler Aligner (BWA) tool is then used for mapping sequence data against the reference human genome (hg38): "The Burrows–Wheeler Aligner (BWA) tool is then used for mapping sequence data against the reference human genome (hg38)"
- [methods] BWA produces a SAM file, which is then passed to the SortSam tool for sorting by the reference sequence name and left‐most mapping position.: [evidence span withheld — anchor status: not_found]
- [methods] The sorted SAM file is then inputted to the MarkDuplicates to locate, tag, and remove duplicate reads: [evidence span withheld — anchor status: not_found]
- [methods] Next, the index tool from SAMtools is used to create and index file, allowing to access faster.: "Next, the index tool from SAMtools is used to create and index file, allowing to access faster."
- [readme] ## README: drzeeshanahmed/JWES-DB — VCF Parser for JWES: [evidence span withheld — anchor status: not_found]
