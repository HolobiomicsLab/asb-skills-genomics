---
name: reference-guided-read-alignment
description: Use when paired-end Illumina RAD-seq reads (e.g., EcoRI-site reads from
  Populus parents) must be mapped to a reference genome to enable indel calling at
  restriction-site loci.
when_to_use_negative:
- Input reads are long-read (PacBio/Oxford Nanopore) data — BWA-MEM is intended for
  Illumina-type short reads and the article workflow uses parental RAD-seq paired-end
  reads
- No reference genome is available — reference-guided alignment is inapplicable; a
  de novo approach would be required
- The parental indel catalog already exists and the goal is genotyping only — InDelGT
  supports '--nocatalog' to skip this alignment/catalog step
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: BWA 0.7.17
  role: Aligns the parental paired-end reads to the reference genome in the first
    InDelGT step (parental InDel catalog generation)
  repo: https://github.com/lh3/bwa/releases/tag/v0.7.17
- name: SAMtools 1.9
  role: Processes the BWA alignments of parental reads prior to indel calling at parental
    restriction-site loci
  repo: http://samtools.sourceforge.net/
- name: BCFtools 1.9
  role: Used with SAMtools to call indels from the parental alignments, merged into
    the parent.cls catalog
  repo: http://samtools.sourceforge.net/
- name: InDelGT
  role: Integrated pipeline that wraps the BWA/SAMtools/BCFtools alignment and indel-calling
    workflow via a single command and parameters.ini
  repo: https://github.com/tongchf/InDelGT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/aps3.11499
    title: 'InDelGT: An integrated pipeline for extracting indel genotypes for genetic
      mapping in a hybrid population using next‐generation sequencing data'
  source_content_hash: sha256:ff35f4f49a8f7320b4c5ee98c8cd84aac2b15db6e334f813e10b1bff51d76c10
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/tongchf/InDelGT
  tools:
  - BCFtools
  - BCFtools 1.9
  - BWA
  - BWA 0.7.17
  - InDelGT
  - SAMtools
  - SAMtools 1.9
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Reference-guided read alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns parental paired-end RAD-seq FASTQ reads to the P. trichocarpa v4.0 reference genome using BWA 0.7.17, then processes alignments with SAMtools 1.9/BCFtools 1.9, as the first underlying step of InDelGT's parental InDel catalog generation.

## When to use

Use when paired-end Illumina RAD-seq reads (e.g., EcoRI-site reads from Populus parents) must be mapped to a reference genome to enable indel calling at restriction-site loci.

## When NOT to use

- Input reads are long-read (PacBio/Oxford Nanopore) data — BWA-MEM is intended for Illumina-type short reads and the article workflow uses parental RAD-seq paired-end reads
- No reference genome is available — reference-guided alignment is inapplicable; a de novo approach would be required
- The parental indel catalog already exists and the goal is genotyping only — InDelGT supports '--nocatalog' to skip this alignment/catalog step

## Inputs

- Parental paired-end FASTQ read files (R1/R2) for P. deltoides and P. simonii (PARENT1/PARENT2 in parameters.ini)
- Reference genome FASTA (P. trichocarpa v4.0, REFERENCE_FILE:reference.fasta)
- parameters.ini with [folders], [parameters] (GQ:30, MAPQ:5, THREADS:20), and [data files] sections

## Outputs

- Merged parental indel catalog file parent.cls in the -o output directory (~50,532 indel sites for the Populus case study)
- BWA alignments of parental reads processed via SAMtools/BCFtools (intermediate SAM/BAM/variant files)

## How to apply

Install BWA 0.7.17, SAMtools 1.9 with BCFtools 1.9, and Perl modules Parallel::ForkManager, Getopt::Long, and Statistics::Distributions via cpan. Fill the [folders] section of parameters.ini with software paths (BWA_FOLD, SAMTOOLS_FOLD, BCFTOOLS_FOLD) and the [data files] section with REFERENCE_FILE pointing to reference.fasta and PARENT1/PARENT2 pointing to the parental R1/R2 FASTQ files; set filtering thresholds such as GQ:30 and MAPQ:5 and THREADS:20. Launch InDelGT from the work directory with 'nohup perl PathToInDelGT/InDelGT.pl -o directory &' so the multi-hour run is not interrupted; under the hood the pipeline aligns parental paired-end reads to the reference with BWA and processes alignments with SAMtools/BCFtools to call indels at parental restriction-site loci, merging both parents into a single catalog. Validate by confirming that the -o output directory contains the merged parent.cls file with approximately 50,532 indel sites, the count reported for the Populus case study.

## Related tools

- **BWA 0.7.17** (Aligns the parental paired-end reads to the reference genome in the first InDelGT step (parental InDel catalog generation)) — https://github.com/lh3/bwa/releases/tag/v0.7.17
- **SAMtools 1.9** (Processes the BWA alignments of parental reads prior to indel calling at parental restriction-site loci) — http://samtools.sourceforge.net/
- **BCFtools 1.9** (Used with SAMtools to call indels from the parental alignments, merged into the parent.cls catalog) — http://samtools.sourceforge.net/
- **InDelGT** (Integrated pipeline that wraps the BWA/SAMtools/BCFtools alignment and indel-calling workflow via a single command and parameters.ini) — https://github.com/tongchf/InDelGT

## Examples

```
nohup perl PathToInDelGT/InDelGT.pl -o directory &
```

## Evaluation signals

- The -o output directory contains the merged parental indel catalog file parent.cls
- parent.cls contains approximately 50,532 indel sites, the count reported for the Populus case study between P. deltoides and P. simonii
- parameters.ini correctly sets BWA_FOLD, SAMTOOLS_FOLD, BCFTOOLS_FOLD paths and PARENT1/PARENT2 point to the parental R1/R2 FASTQ files
- Thresholds GQ:30 and MAPQ:5 are applied as configured, and the run completes without interruption when launched with nohup in the background

## Limitations

- The full InDelGT run (including this alignment step) can take hours to days and should be launched in the background with nohup to avoid interruption
- Large indels (>5 bp) complicate or distort correct read mapping compared with small indels (1–5 bp), requiring higher coverage depth
- Progeny average genome coverage was only ~20x (potentially up to 100x at EcoRI restriction sites covering at most 20% of the genome), and heterozygote determination requires higher coverage depth than homozygotes

## Evidence
