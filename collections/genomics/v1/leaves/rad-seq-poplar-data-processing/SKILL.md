---
name: rad-seq-poplar-data-processing
description: Use when starting an InDelGT run on parental RAD-seq FASTQ files of two
  parents of a hybrid population and you need the merged parental indel catalog (parent.cls)
  before genotype calling.
when_to_use_negative:
- The parental InDel catalog (parent.cls) already exists from a prior run — skip this
  step with the --nocatalog flag
- Input reads are long-read sequencing rather than paired-end RAD-seq reads mapped
  to a reference genome with BWA
- The goal is calling InDel genotypes across progeny or testing segregation — those
  are InDelGT steps 2 and 3, run after this catalog exists
edam_operation: http://edamontology.org/operation_0452
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0625
tools:
- name: InDelGT
  role: Pipeline wrapper that generates the parental InDel catalog via its first step
    (launched with InDelGT.pl -o directory); also skips this step with --nocatalog
  repo: https://github.com/tongchf/InDelGT
- name: BWA 0.7.17
  role: Aligns parental paired-end reads to the reference genome during catalog generation
  repo: https://github.com/lh3/bwa/releases/tag/v0.7.17
- name: SAMtools 1.9
  role: Processes BWA alignments of the parental reads before indel calling
  repo: http://samtools.sourceforge.net/
- name: BCFtools 1.9
  role: Calls indels at parental restriction-site loci from the processed alignments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/aps3.11499
    title: 'InDelGT: An integrated pipeline for extracting indel genotypes for genetic
      mapping in a hybrid population using next‐generation sequencing data'
  source_content_hash: sha256:55d1baaabc8d5a9829589ef2052214a5e1755669576da67de6abbc4710b8faf5
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
  edam_operation: http://edamontology.org/operation_0452
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# rad-seq-poplar-data-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates the merged parental InDel catalog (parent.cls) by aligning two parents' RAD-seq paired-end FASTQ reads to the P. trichocarpa v4.0 reference with BWA and calling indels at parental restriction-site loci with SAMtools/BCFtools via InDelGT. This is the first of InDelGT's three steps and produced 50,532 indel sites in the Populus case study.

## When to use

Use when starting an InDelGT run on parental RAD-seq FASTQ files of two parents of a hybrid population and you need the merged parental indel catalog (parent.cls) before genotype calling.

## When NOT to use

- The parental InDel catalog (parent.cls) already exists from a prior run — skip this step with the --nocatalog flag
- Input reads are long-read sequencing rather than paired-end RAD-seq reads mapped to a reference genome with BWA
- The goal is calling InDel genotypes across progeny or testing segregation — those are InDelGT steps 2 and 3, run after this catalog exists

## Inputs

- Reference genome FASTA (e.g. P. trichocarpa v4.0 reference.fasta)
- Paired-end RAD-seq FASTQ files (R1/R2) for parent 1 (P. deltoides)
- Paired-end RAD-seq FASTQ files (R1/R2) for parent 2 (P. simonii)
- parameters.ini configuration file with [folders], [parameters], and [data files] sections

## Outputs

- Merged parental InDel catalog file parent.cls in the -o output directory (approximately 50,532 indel sites in the Populus case study)

## How to apply

Install BWA 0.7.17 and SAMtools 1.9 with BCFtools 1.9, plus Perl modules Parallel::ForkManager, Getopt::Long, and Statistics::Distributions via cpan. Fill in parameters.ini in the work directory: the [folders] section with software install paths (BWA_FOLD, SAMTOOLS_FOLD, BCFTOOLS_FOLD, INDELGT_FOLD, DATAFOLD), the [parameters] section (e.g. GQ:30, MAPQ:5, PROGENY_NUMBER:20, MISPCT:20, HETEROZYGOUS_DEPTH:3, HOMOZYGOUS_DEPTH:5, PVALUE:0.01, THREADS:20), and the [data files] section pointing REFERENCE_FILE to reference.fasta and PARENT1/PARENT2 to the parental R1/R2 fastq files. Launch the first InDelGT step from the work directory in the background with 'nohup perl PathToInDelGT/InDelGT.pl -o directory &' so the run (which can take hours to days) is not interrupted. Under the hood, InDelGT aligns each parent's paired-end reads to the reference with BWA and processes alignments with SAMtools/BCFtools to call indels at parental restriction-site loci, merging both parents into a single catalog.

## Related tools

- **InDelGT** (Pipeline wrapper that generates the parental InDel catalog via its first step (launched with InDelGT.pl -o directory); also skips this step with --nocatalog) — https://github.com/tongchf/InDelGT
- **BWA 0.7.17** (Aligns parental paired-end reads to the reference genome during catalog generation) — https://github.com/lh3/bwa/releases/tag/v0.7.17
- **SAMtools 1.9** (Processes BWA alignments of the parental reads before indel calling) — http://samtools.sourceforge.net/
- **BCFtools 1.9** (Calls indels at parental restriction-site loci from the processed alignments)

## Examples

```
nohup perl PathToInDelGT/InDelGT.pl -o directory &
```

## Evaluation signals

- parent.cls exists in the -o output directory after the run completes
- parent.cls contains approximately 50,532 indel sites, the count reported for the Populus case study between P. deltoides and P. simonii
- The catalog contains indels at parental EcoRI restriction-site loci aligned to the P. trichocarpa v4.0 reference
- Both parents' alignments were merged into a single catalog (not separate per-parent catalogs)
- The nohup background process completed without interruption, since a full run can take hours to days

## Limitations

- A full run can take hours to days, so it should be launched in the background (nohup) to avoid interruption
- Large indels (>5 bp) complicate or distort correct read mapping compared with small indels (1–5 bp), requiring higher coverage depth
- Determining heterozygotes requires a higher coverage depth than homozygotes; the pipeline uses HETEROZYGOUS_DEPTH:3 and HOMOZYGOUS_DEPTH:5 thresholds
- No version number is stated for InDelGT itself in the article text

## Evidence

- [results] Parental catalog count for the Populus case study: [evidence span withheld — anchor status: not_found]
