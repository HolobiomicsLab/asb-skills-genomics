---
name: pipeline-parameter-file-configuration
description: 'Use when preparing to run InDelGT on two-parent hybrid population RAD-seq
  data: you must fill in parameters.ini (folders, parameters, data files sections)
  before launching InDelGT.pl.'
when_to_use_negative:
- Data are not paired-end FASTQ reads from a two-parent hybrid population being genotyped
  with InDelGT — this file format is specific to InDelGT's folder/parameter/data-file
  convention
- The first parental catalog step is being skipped via --nocatalog (e.g. re-running
  only genotyping or segregation analysis with an existing parent.cls), so parental
  FASTQ paths and catalog-generation settings need not be reconfigured
- Alignment/genotyping is handled by a different indel-calling toolchain that does
  not read parameters.ini
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3053
tools:
- name: InDelGT
  role: Pipeline configured via parameters.ini; launched as 'perl InDelGT.pl -o directory'
    to generate the parental indel catalog and indel genotypes
  repo: https://github.com/tongchf/InDelGT
- name: BWA 0.7.17
  role: Aligner whose install path is recorded in the [folders] section (BWA_FOLD)
    and used to map parental reads to the reference
  repo: https://github.com/lh3/bwa/releases/tag/v0.7.17
- name: SAMtools 1.9
  role: Install path recorded in parameters.ini (SAMTOOLS_FOLD); processes BWA alignments
    for indel calling
- name: BCFtools 1.9
  role: Install path recorded in parameters.ini (BCFTOOLS_FOLD); calls indels at parental
    restriction-site loci
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/aps3.11499
    title: 'InDelGT: An integrated pipeline for extracting indel genotypes for genetic
      mapping in a hybrid population using next‐generation sequencing data'
  source_content_hash: sha256:53996c88fbe1d12b10ef8c91de452de0caa9d293147c09e2d95e62dc78ccc022
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/tongchf/InDelGT
  repo_url: https://github.com/tongchf/InDelGT
  tools:
  - BCFtools
  - BCFtools 1.9
  - BWA
  - BWA 0.7.17
  - InDelGT
  - SAMtools
  - SAMtools 1.9
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# pipeline-parameter-file-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure the InDelGT pipeline through its parameters.ini file, which supplies software install paths (BWA 0.7.17, SAMtools 1.9, BCFtools 1.9), quality/depth/segregation thresholds, and the reference plus parental/progeny FASTQ file locations needed to run the three-step indel genotyping workflow.

## When to use

Use when preparing to run InDelGT on two-parent hybrid population RAD-seq data: you must fill in parameters.ini (folders, parameters, data files sections) before launching InDelGT.pl.

## When NOT to use

- Data are not paired-end FASTQ reads from a two-parent hybrid population being genotyped with InDelGT — this file format is specific to InDelGT's folder/parameter/data-file convention
- The first parental catalog step is being skipped via --nocatalog (e.g. re-running only genotyping or segregation analysis with an existing parent.cls), so parental FASTQ paths and catalog-generation settings need not be reconfigured
- Alignment/genotyping is handled by a different indel-calling toolchain that does not read parameters.ini

## Inputs

- parameters.ini template (three sections: folders, parameters, data files)
- Reference genome FASTA (reference.fasta)
- Parental paired-end RAD-seq FASTQ files (R1/R2 for PARENT1 and PARENT2)
- Progeny paired-end FASTQ files (PROGENY_NUMBER of individuals)
- Installed BWA 0.7.17, SAMtools 1.9, BCFtools 1.9 binaries and their paths
- Perl modules Parallel::ForkManager, Getopt::Long, Statistics::Distributions

## Outputs

- Configured parameters.ini consumable by InDelGT.pl
- Merged parental indel catalog file parent.cls in the -o output directory

## How to apply

Create parameters.ini in the work directory with three parts: [folders] listing BWA_FOLD, SAMTOOLS_FOLD, BCFTOOLS_FOLD, INDELGT_FOLD and DATAFOLD install/data paths; [parameters] with GQ:30 (minimum InDel genotype score), MAPQ:5 (minimum read mapping quality), PROGENY_NUMBER, MISPCT:20 (percent missing genotypes per InDel), HETEROZYGOUS_DEPTH:3, HOMOZYGOUS_DEPTH:5, PVALUE:0.01 (segregation ratio test threshold) and THREADS:20; and [data files] pointing REFERENCE_FILE to reference.fasta and PARENT1/PARENT2 to the parental R1/R2 FASTQ files. Install Perl modules Parallel::ForkManager, Getopt::Long, and Statistics::Distributions via cpan before running. Launch the full pipeline with 'nohup perl PathToInDelGT/InDelGT.pl -o directory &' from the work directory so the multi-hour run is not interrupted; the first step generates the merged parental indel catalog parent.cls via BWA alignment and SAMtools/BCFtools indel calling at restriction-site loci.

## Related tools

- **InDelGT** (Pipeline configured via parameters.ini; launched as 'perl InDelGT.pl -o directory' to generate the parental indel catalog and indel genotypes) — https://github.com/tongchf/InDelGT
- **BWA 0.7.17** (Aligner whose install path is recorded in the [folders] section (BWA_FOLD) and used to map parental reads to the reference) — https://github.com/lh3/bwa/releases/tag/v0.7.17
- **SAMtools 1.9** (Install path recorded in parameters.ini (SAMTOOLS_FOLD); processes BWA alignments for indel calling)
- **BCFtools 1.9** (Install path recorded in parameters.ini (BCFTOOLS_FOLD); calls indels at parental restriction-site loci)

## Examples

```
nohup perl PathToInDelGT/InDelGT.pl -o directory &
```

## Evaluation signals

- parameters.ini parses into exactly three parts — [folders], [parameters], [data files] — with every software path (BWA_FOLD, SAMTOOLS_FOLD, BCFTOOLS_FOLD, INDELGT_FOLD, DATAFOLD) resolving to an existing directory
- Threshold values match the published configuration: GQ:30, MAPQ:5, MISPCT:20, HETEROZYGOUS_DEPTH:3, HOMOZYGOUS_DEPTH:5, PVALUE:0.01, THREADS:20, with PROGENY_NUMBER equal to the actual progeny count
- REFERENCE_FILE points to a valid FASTA and PARENT1/PARENT2 point to existing paired R1/R2 FASTQ files
- After the run, the -o output directory contains parent.cls, the merged parental indel catalog (approximately 50,532 indel sites in the Populus case study)
- The run completes with a single command line without being interrupted (nohup background execution survives shell exit)

## Limitations

- The full run can take hours to days, so it must be launched with nohup in the background to avoid interruption
- Users must separately download and install BWA 0.7.17, SAMtools 1.9 (with BCFtools 1.9) and the required Perl modules before configuration; the parameter file only records their paths
- Large indels (>5 bp) complicate or distort correct read mapping compared with small indels (1–5 bp), so genotype calling quality depends on coverage depth at restriction-site loci

## Evidence

- [methods] # the minimum score of InDel genotypes
    GQ:30: "# the minimum score of InDel genotypes
    GQ:30"
