---
name: reference-genome-alignment
description: Use when raw or Trimmomatic-trimmed FASTQ reads from WGS/WES samples
  (e.g. SRA projects PRJNA624223, PRJEB39632) must be aligned to a reference genome
  before variant calling with GATK HaplotypeCaller.
when_to_use_negative:
- Input is raw untrimmed FASTQ that has not passed the FastQC/Trimmomatic QC stage
  — the JWES chain requires QC and trimming before mapping
- Non-human or non-hg38 reference genomes — the article grounds this workflow in mapping
  against the reference human genome (hg38)
- Downstream analysis needing only variant counts from already-annotated VCFs (e.g.
  parsing with JWES-DB) — no re-alignment is required
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: BWA
  role: Burrows–Wheeler Aligner used to map Trimmomatic-processed sequence data against
    the reference human genome hg38, producing the SAM file
- name: SortSam
  role: Sorts the BWA-produced SAM file by reference sequence name and left-most mapping
    position
- name: MarkDuplicates
  role: Locates, tags, and removes duplicate reads in the sorted SAM to enhance alignment
    quality and reduce variant false positives
- name: SAMtools
  role: Its index tool creates an index file of the alignment, allowing faster access
- name: Trimmomatic
  role: Upstream step that trims low-quality sequences, filters adapters, and cuts
    bases before reads are forwarded to BWA
- name: JWES
  role: The Java-based pipeline that orchestrates the FastQC→Trimmomatic→BWA→SortSam→MarkDuplicates→SAMtools
    index chain
  repo: github.com/drzeeshanahmed/JWES-DB
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:ab54c0a162b59e5112612dd469bcef77dbc9f0367052eaf164bbaab548702041
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
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# reference-genome-alignment

## Summary

Maps quality-trimmed FASTQ reads against the human reference genome hg38 with BWA as the alignment stage of the JWES pipeline, then sorts, deduplicates, and indexes the resulting SAM file to prepare a clean BAM for GATK variant calling.

## When to use

Use when raw or Trimmomatic-trimmed FASTQ reads from WGS/WES samples (e.g. SRA projects PRJNA624223, PRJEB39632) must be aligned to a reference genome before variant calling with GATK HaplotypeCaller.

## When NOT to use

- Input is raw untrimmed FASTQ that has not passed the FastQC/Trimmomatic QC stage — the JWES chain requires QC and trimming before mapping
- Non-human or non-hg38 reference genomes — the article grounds this workflow in mapping against the reference human genome (hg38)
- Downstream analysis needing only variant counts from already-annotated VCFs (e.g. parsing with JWES-DB) — no re-alignment is required

## Inputs

- Trimmed FASTQ reads (output of Trimmomatic: low-quality trimming, adapter filtering, base cutting)
- Human reference genome hg38

## Outputs

- SAM file from BWA mapping
- Sorted SAM/BAM file (SortSam, ordered by reference sequence name and left-most mapping position)
- Deduplicated alignment (MarkDuplicates: duplicate reads located, tagged, and removed)
- SAMtools index file for faster access

## How to apply

Forward the Trimmomatic-trimmed FASTQ (after low-quality sequence trimming, adapter filtering, and base cutting) to the Burrows–Wheeler Aligner (BWA) and map the reads against the reference human genome hg38. BWA emits a SAM file, which is passed to SortSam to sort reads by reference sequence name and left-most mapping position. The sorted SAM is then input to MarkDuplicates to locate, tag, and remove duplicate reads, which enhances alignment quality and reduces the number of variant false positives. Finally, create an index file with the index tool from SAMtools to allow faster access to the alignment. This produces the indexed, deduplicated alignment that downstream GATK steps (RealignerTargetCreator/IndelRealigner, BaseRecalibrator, HaplotypeCaller) consume.

## Related tools

- **BWA** (Burrows–Wheeler Aligner used to map Trimmomatic-processed sequence data against the reference human genome hg38, producing the SAM file)
- **SortSam** (Sorts the BWA-produced SAM file by reference sequence name and left-most mapping position)
- **MarkDuplicates** (Locates, tags, and removes duplicate reads in the sorted SAM to enhance alignment quality and reduce variant false positives)
- **SAMtools** (Its index tool creates an index file of the alignment, allowing faster access)
- **Trimmomatic** (Upstream step that trims low-quality sequences, filters adapters, and cuts bases before reads are forwarded to BWA)
- **JWES** (The Java-based pipeline that orchestrates the FastQC→Trimmomatic→BWA→SortSam→MarkDuplicates→SAMtools index chain) — github.com/drzeeshanahmed/JWES-DB

## Evaluation signals

- BWA must be run on hg38 specifically; confirm the reference build matches the article's stated human reference genome (hg38)
- The BWA output must be a SAM file that is subsequently sorted by SortSam on reference sequence name and left-most mapping position — a positional check on read ordering catches a skipped sort step
- MarkDuplicates must report duplicate reads located, tagged, and removed; a large number of retained duplicates predicts inflated variant false positives
- A SAMtools index file must exist alongside the final alignment to enable faster downstream access
- Downstream sanity check: the pipeline applied to SRR12474733 (project PRJNA657985) yielded 43,685 variants, and cohort WGS runs produced counts such as 2,736,453 (PRJNA657938) and 1,793,959 (PRJNA624223); orders-of-magnitude deviation suggests an alignment-stage failure

## Limitations

- The pipeline and its alignment chain are designed for and validated on WGS/WES human data (hg38); other organisms or reference builds are out of scope of the reported workflow
- JWES functionality was validated in the Rutgers University HPC environment deployed with Slurm Workload Manager — large WGS runs require substantial computational resources (HPC/cloud)
- Only the duplicate-removal and sorting steps are described for post-alignment cleanup; the article does not report per-sample mapping-rate or coverage thresholds for this stage

## Evidence

- [methods] BWA mapping against hg38: "The Burrows–Wheeler Aligner (BWA) tool is then used for mapping sequence data against the reference human genome (hg38)"
- [methods] SortSam sorting of SAM: [evidence span withheld — anchor status: not_found]
- [methods] MarkDuplicates purpose: "to locate, tag, and remove duplicate reads, and enhance the quality of the alignment and reduce the number of variant false positives"
- [methods] Upstream Trimmomatic step: [evidence span withheld — anchor status: not_found]
- [abstract] Pipeline positioning in abstract: "The JWES pipeline starts with the FastQC, a command‐line‐based tool that takes FASTQ/FASTA files as input and performs QC"
