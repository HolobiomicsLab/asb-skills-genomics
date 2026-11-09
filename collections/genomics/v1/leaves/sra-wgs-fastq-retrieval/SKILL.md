---
name: sra-wgs-fastq-retrieval
description: Use when the analysis goal is to benchmark or reproduce variant counts
  on public WGS/WES data identified by SRA run or BioProject accessions (e.g. SRR12474733,
  PRJNA657938) and the raw FASTQ files must be downloaded from NCBI SRA first.
when_to_use_negative:
- Input data is already locally available as FASTQ files and only pipeline execution
  is needed
- The target data are in-house/experimentally generated rather than publicly archived
  in SRA
- The task concerns WGS/WES analysis steps downstream of data acquisition (alignment,
  variant calling, annotation)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NCBI Sequence Read Archive (SRA)
  role: Public repository from which the WGS FASTQ samples for each project accession
    are downloaded
- name: JWES
  role: Downstream pipeline that consumes the retrieved FASTQ files, starting with
    FastQC QC of FASTQ/FASTA input
  repo: github.com/drzeeshanahmed/JWES-DB
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:54e070a5c67b11275d330a4d92a6263fc767cc6c9866863e0327480e295e72ce
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA
  - FastQC
  - JWES
  - MarkDuplicates
  - NCBI Sequence Read Archive (SRA)
  - SAMtools
  - SortSam
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sra-wgs-fastq-retrieval

## Summary

Retrieve publicly available WGS FASTQ samples from the NCBI Sequence Read Archive (SRA) for specified BioProject accessions (e.g. PRJNA624223, PRJNA649101, PRJEB39632, PRJNA657938, PRJNA657985/SRR12474733) so they can serve as inputs to the JWES variant-discovery pipeline.

## When to use

Use when the analysis goal is to benchmark or reproduce variant counts on public WGS/WES data identified by SRA run or BioProject accessions (e.g. SRR12474733, PRJNA657938) and the raw FASTQ files must be downloaded from NCBI SRA first.

## When NOT to use

- Input data is already locally available as FASTQ files and only pipeline execution is needed
- The target data are in-house/experimentally generated rather than publicly archived in SRA
- The task concerns WGS/WES analysis steps downstream of data acquisition (alignment, variant calling, annotation)

## Inputs

- SRA run accessions (e.g. SRR12474733, SRR891275)
- BioProject accessions (e.g. PRJNA657985, PRJNA657938, PRJNA624223, PRJEB39632, PRJNA649101)
- NCBI Sequence Read Archive (SRA) database

## Outputs

- Raw WGS FASTQ sample files ready for pipeline input
- Mapping of project accession to downloaded sample/run used for the benchmark table (variant_counts_benchmark.csv)

## How to apply

Identify the BioProject accessions of interest and their associated SRA run identifiers (e.g. sample SRR12474733 of project PRJNA657985, as used in the JWES case study). Download the corresponding WGS FASTQ samples from the NCBI Sequence Read Archive database, noting the original submitter database where applicable (the JWES paper's example run SRR891275 was submitted via Gene Expression Omnibus). Feed the retrieved FASTQ files directly into the JWES data-processing module, which expects FASTQ/FASTA input starting at the FastQC QC step, followed by Trimmomatic trimming, BWA alignment to hg38, and downstream GATK variant calling. Record the project accession alongside each downloaded sample so per-project variant counts can later be compiled into a benchmark table such as variant_counts_benchmark.csv.

## Related tools

- **NCBI Sequence Read Archive (SRA)** (Public repository from which the WGS FASTQ samples for each project accession are downloaded)
- **JWES** (Downstream pipeline that consumes the retrieved FASTQ files, starting with FastQC QC of FASTQ/FASTA input) — github.com/drzeeshanahmed/JWES-DB

## Evaluation signals

- Each requested project accession has at least one downloaded FASTQ sample (e.g. PRJNA624223, PRJNA649101, PRJEB39632, PRJNA657938, and SRR12474733 for PRJNA657985 all present)
- Downloaded files are valid FASTQ and accepted as input by the FastQC QC step of the JWES data-processing module
- Each downloaded sample can be traced back to its SRA run accession (e.g. SRR12474733 ↔ PRJNA657985)
- Downstream variant counts per project fall within tolerance of reported figures: 1,793,959 (PRJNA624223), 54,930 (PRJNA649101), 154,016 (PRJEB39632), 2,736,453 (PRJNA657938), 43,685 (PRJNA657985)
- The compiled benchmark table (variant_counts_benchmark.csv) contains a value for every project accession field

## Limitations

- The JWES paper's example run (SRR891275) was submitted via GEO, so run-to-submitter-database provenance varies across SRA entries and must be checked per accession
- Public cohort analyses span many samples (the JWES cohort comprised 14 WGS samples yielding 4,803,792 variants total), so retrieval and downstream processing require substantial HPC/computational resources
- Accession mismatches (wrong run pulled for a project) silently propagate into per-project variant count comparisons

## Evidence

- [results] we downloaded SRR891275 from NCBI Sequence Read Architecture (SRA) database: "we downloaded SRR891275 from NCBI Sequence Read Architecture (SRA) database"
- [results] SRA submitter provenance: "The SRR891275 was submitted by the Gene Expression Omnibus (GEO)"
- [methods] Pipeline input format: "The JWES pipeline starts with the FastQC, a command‐line‐based tool that takes FASTQ/FASTA files as input and performs QC"
- [results] Cohort scale: "which included total 14 whole genome sequenced samples (Table 1). The JWES pipeline delivered promising results by offering a wider range of variants (4 803 792)"
- [results] Example public WGS project: "we applied the JWES to the WGS sample (SRR12474733) of project PRJNA657985, which resulted in 43 685 variants"
