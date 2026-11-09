---
name: per-sample-variant-count-benchmarking
description: 'Use when reproducing or validating a published WGS/WES pipeline''s variant
  yield: public SRA FASTQ samples are given with accession IDs (PRJNA/SRR) and expected
  total variant counts per project to compare against.'
when_to_use_negative:
- The goal is variant interpretation or clinical genotyping rather than validating
  reported variant yield — use the JWES visualization/annotation modules or dedicated
  clinical pipelines instead
- The input is raw sequencing data from non-human organisms — the pipeline aligns
  against the hg38 human reference genome and its variant counts are only comparable
  for human WGS/WES samples
- Only a single sample with no published reference count exists — per-sample benchmarking
  against reported values requires published figures to compare against
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: JWES
  role: Java-based FAIR pipeline providing the data-processing module (FastQC through
    SAMtools indexing) and GATK variant calling whose per-project outputs are benchmarked
  repo: github.com/drzeeshanahmed/JWES-DB
- name: FastQC
  role: First QC step on FASTQ/FASTA input; its per-sample QC output gates downstream
    trimming and is required before any counts are generated
- name: Trimmomatic
  role: Trims low-quality sequences, filters adapters, and cuts bases prior to alignment
- name: BWA
  role: Maps trimmed reads against the hg38 human reference genome to produce SAM
    files
- name: SortSam
  role: Sorts the BWA-produced SAM by reference sequence name and left-most mapping
    position before duplicate marking
- name: MarkDuplicates
  role: Locates, tags, and removes duplicate reads to reduce variant false positives
    before counting
- name: SAMtools
  role: Index tool creates the index file enabling faster access to aligned data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:497e42e2aa7bea2c8f098b1adad2b3f3c690bc2021c9908cdd71fe647644495a
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# per-sample-variant-count-benchmarking

## Summary

Benchmarking skill that reproduces the per-project variant counts reported by the JWES WGS/WES pipeline (e.g. 1,793,959 for PRJNA624223; 43,685 for PRJNA657985) by running the full FastQC→Trimmomatic→BWA(hg38)→GATK→snpEff workflow on public SRA samples and tabulating total variants per project accession.

## When to use

Use when reproducing or validating a published WGS/WES pipeline's variant yield: public SRA FASTQ samples are given with accession IDs (PRJNA/SRR) and expected total variant counts per project to compare against.

## When NOT to use

- The goal is variant interpretation or clinical genotyping rather than validating reported variant yield — use the JWES visualization/annotation modules or dedicated clinical pipelines instead
- The input is raw sequencing data from non-human organisms — the pipeline aligns against the hg38 human reference genome and its variant counts are only comparable for human WGS/WES samples
- Only a single sample with no published reference count exists — per-sample benchmarking against reported values requires published figures to compare against

## Inputs

- Public WGS FASTQ files from NCBI SRA for projects PRJNA624223, PRJNA649101, PRJEB39632, PRJNA657938, and PRJNA657985 (sample SRR12474733)
- hg38 human reference genome (for BWA alignment)
- Annotated variant VCF files produced by the JWES pipeline (snpEff output)

## Outputs

- variant_counts_benchmark.csv: a table mapping each project accession (e.g. PRJNA624223) to its total variant count
- Per-project total variant counts (e.g. 1,793,959 for PRJNA624223; 43,685 for SRR12474733/PRJNA657985)

## How to apply

Download the public WGS FASTQ samples (PRJNA624223, PRJNA649101, PRJEB39632, PRJNA657938, PRJNA657985/SRR12474733) from NCBI SRA, then execute the JWES data-processing module under Slurm: FastQC QC, Trimmomatic trimming, BWA alignment to hg38, SortSam, MarkDuplicates, SAMtools indexing. Call variants with GATK HaplotypeCaller (SNPs and INDELs), separate types with SelectVariants, recalibrate with VariantRecalibrator (Gaussian mixture model) and ApplyRecalibration, then annotate with snpEff. Count total variants per sample from the annotated VCF output and compile project accession vs. variant count into variant_counts_benchmark.csv. Rationale: the counts must reproduce the published figures — 1,793,959 (PRJNA624223), 54,930 (PRJNA649101), 154,016 (PRJEB39632), 2,736,453 (PRJNA657938), 43,685 (PRJNA657985) — confirming the pipeline was executed end-to-end identically to the authors' validated runs.

## Related tools

- **JWES** (Java-based FAIR pipeline providing the data-processing module (FastQC through SAMtools indexing) and GATK variant calling whose per-project outputs are benchmarked) — github.com/drzeeshanahmed/JWES-DB
- **FastQC** (First QC step on FASTQ/FASTA input; its per-sample QC output gates downstream trimming and is required before any counts are generated)
- **Trimmomatic** (Trims low-quality sequences, filters adapters, and cuts bases prior to alignment)
- **BWA** (Maps trimmed reads against the hg38 human reference genome to produce SAM files)
- **SortSam** (Sorts the BWA-produced SAM by reference sequence name and left-most mapping position before duplicate marking)
- **MarkDuplicates** (Locates, tags, and removes duplicate reads to reduce variant false positives before counting)
- **SAMtools** (Index tool creates the index file enabling faster access to aligned data)

## Examples

```
java -jar jwes.jar --run-all --input SRR12474733.fastq --reference hg38 --project PRJNA657985 && jwes-db parse annotated.vcf > variant_counts_benchmark.csv
```

## Evaluation signals

- Each project accession's count in variant_counts_benchmark.csv matches the published figure within tolerance: PRJNA624223=1,793,959; PRJNA649101=54,930; PRJEB39632=154,016; PRJNA657938=2,736,453; PRJNA657985/SRR12474733=43,685
- Every row of variant_counts_benchmark.csv has both a project accession field and a variant-count field populated (no empty cells)
- The counts derive from annotated (snpEff) variant files, not raw HaplotypeCaller output, ensuring the same counting basis as the published pipeline
- Pipeline completed under an HPC Slurm environment, matching the authors' validated Rutgers University HPC deployment
- Cross-check: summing counts across the full 14-sample cohort should approach the reported total of 4,803,792 variants when all public WGS samples are processed

## Limitations

- The current JWES version only offers a command-line interface; no interactive interface is implemented, complicating reproducible benchmark runs
- Reported counts are specific to the hg38 human reference and the exact GATK (HaplotypeCaller, VariantRecalibrator, ApplyRecalibration) configuration; small pipeline deviations can shift counts outside tolerance
- Some reported counts (e.g. 43,685 for PRJNA657985) come from a single sample, so benchmarking reproduces sample-level rather than project-level totals unless multiple runs are aggregated

## Evidence

- [other] variant_counts_benchmark.csv: [evidence span withheld — anchor status: not_found]
- [results] 43,685 variants for SRR12474733: "we applied the JWES to the WGS sample (SRR12474733) of project PRJNA657985, which resulted in 43 685 variants"
- [methods] Gaussian mixture model recalibration: "The JWES applies VariantRecalibrator to create a Gaussian mixture model and produce high‐quality cluster of SNPs"
- [methods] validated on Rutgers HPC (Slurm): "Its functionality has been validated and reproduced multiple times within the HPC environment (deployed with Slurm Workload Manager) provided by the Rutgers University."
- [results] cohort total 4,803,792 variants: "which included total 14 whole genome sequenced samples (Table 1). The JWES pipeline delivered promising results by offering a wider range of variants (4 803 792)"
