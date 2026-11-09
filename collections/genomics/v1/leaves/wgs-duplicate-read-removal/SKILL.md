---
name: wgs-duplicate-read-removal
description: Use when processing WGS/WES FASTQ reads aligned to a reference genome
  (e.g., hg38 with BWA) and a duplicate-free BAM is needed before GATK realignment,
  recalibration, and variant calling.
when_to_use_negative:
- Input is unaligned FASTQ — duplicates cannot be marked before mapping/alignment;
  run BWA and SortSam first
- Input SAM is not sorted by reference sequence name and left-most mapping position
  — MarkDuplicates requires the SortSam output ordering
- The target is variant annotation/visualization (snpEff, Circos) rather than alignment
  cleanup — those are separate downstream JWES modules
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MarkDuplicates
  role: Locates, tags, and removes duplicate reads from the sorted SAM file to reduce
    variant false positives
- name: SortSam
  role: Sorts the BWA SAM output by reference sequence name and left-most mapping
    position, producing the required input ordering for MarkDuplicates
- name: SAMtools
  role: Its index tool creates the index file on the post-deduplication BAM, allowing
    faster access in subsequent pipeline steps
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:9502316afcfdc1f425cbe654679b971e0828645f0fd959cb414170426b48573d
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# wgs-duplicate-read-removal

## Summary

Duplicate-read identification and removal step of the JWES whole-genome/exome pipeline: after BWA alignment against hg38 and SortSam sorting, MarkDuplicates locates, tags, and removes duplicate reads so that downstream GATK variant calling (HaplotypeCaller, VariantRecalibrator) produces fewer false-positive variants.

## When to use

Use when processing WGS/WES FASTQ reads aligned to a reference genome (e.g., hg38 with BWA) and a duplicate-free BAM is needed before GATK realignment, recalibration, and variant calling.

## When NOT to use

- Input is unaligned FASTQ — duplicates cannot be marked before mapping/alignment; run BWA and SortSam first
- Input SAM is not sorted by reference sequence name and left-most mapping position — MarkDuplicates requires the SortSam output ordering
- The target is variant annotation/visualization (snpEff, Circos) rather than alignment cleanup — those are separate downstream JWES modules

## Inputs

- Sorted SAM file from SortSam (ordered by reference sequence name and left-most mapping position), produced by BWA alignment of processed FASTQ reads against hg38

## Outputs

- BAM file with duplicate reads located, tagged, and removed
- Indexed BAM (via the SAMtools index tool) for fast access by downstream GATK steps

## How to apply

In the JWES pipeline, first sort the SAM file produced by BWA mapping against the reference human genome hg38 with SortSam, ordering by reference sequence name and left-most mapping position — this is the required input ordering for MarkDuplicates. Then feed the sorted SAM file to MarkDuplicates to locate, tag, and remove duplicate reads; the rationale is to enhance alignment quality and reduce the number of variant false positives in the eventual HaplotypeCaller SNP/INDEL calls. After duplicate removal, index the resulting BAM with the SAMtools index tool so downstream steps (RealignerTargetCreator/IndelRealigner, BaseRecalibrator, HaplotypeCaller) can access it faster. On the paper's case-study sample SRR891275 (PRJNA207663, downloaded from NCBI SRA), this step sits between BWA/SortSam and the GATK local-realignment and base-recalibration stages, ultimately contributing to a call set of 20,749 variants.

## Related tools

- **MarkDuplicates** (Locates, tags, and removes duplicate reads from the sorted SAM file to reduce variant false positives)
- **SortSam** (Sorts the BWA SAM output by reference sequence name and left-most mapping position, producing the required input ordering for MarkDuplicates)
- **SAMtools** (Its index tool creates the index file on the post-deduplication BAM, allowing faster access in subsequent pipeline steps)

## Evaluation signals

- Input SAM must be sorted by reference sequence name and left-most mapping position (SortSam output) before MarkDuplicates runs
- Duplicate reads are located and tagged, and duplicates removed, in the output BAM — no duplicate reads remain for downstream GATK steps
- The resulting BAM is indexed with the SAMtools index tool and accessible to RealignerTargetCreator/IndelRealigner and BaseRecalibrator
- Downstream variant calling shows a reduced number of variant false positives, consistent with the stated purpose of the deduplication step
- End-to-end reproduction on SRR891275 (PRJNA207663) matches the published total of 20,749 variants across all chromosomes

## Limitations

- MarkDuplicates only enhances alignment quality and reduces variant false positives; it does not itself improve base quality scores — JWES handles that separately with the GATK BaseRecalibrator ML model
- JWES currently offers only a command-line interface for this and other steps; no interactive interface is implemented yet
- JWES functionality has been validated mainly in the Rutgers University HPC environment (Slurm Workload Manager), so resource requirements (HPC/cloud) must be available

## Evidence

- [methods] BWA produces a SAM file, which is then passed to the SortSam tool for sorting by the reference sequence name and left‐most mapping position.: [evidence span withheld — anchor status: not_found]
- [methods] to locate, tag, and remove duplicate reads, and enhance the quality of the alignment and reduce the number of variant false positives: "to locate, tag, and remove duplicate reads, and enhance the quality of the alignment and reduce the number of variant false positives"
- [methods] Next, the index tool from SAMtools is used to create and index file, allowing to access faster.: "Next, the index tool from SAMtools is used to create and index file, allowing to access faster."
- [results] we downloaded SRR891275 from NCBI Sequence Read Architecture (SRA) database: "we downloaded SRR891275 from NCBI Sequence Read Architecture (SRA) database"
