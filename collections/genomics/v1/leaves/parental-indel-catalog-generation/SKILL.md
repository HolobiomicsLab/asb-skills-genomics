---
name: parental-indel-catalog-generation
description: Use when you need to build a catalog of indel sites segregating between
  two parents (e.g., P. deltoides × P. simonii) from parental paired-end FASTQ reads,
  as the first step before calling indel genotypes across a hybrid population.
when_to_use_negative:
- The parental indel catalog already exists (parent.cls from a previous run) — skip
  this step with the --nocatalog flag to go directly to genotype calling.
- The goal is calling indel genotypes across the progeny or testing Mendelian segregation
  — those are steps 2 and 3 of InDelGT, not catalog generation.
- Input data are not paired-end reads from two parents plus a reference genome aligned
  to restriction sites (e.g., whole-genome variant calls from other tools).
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3053
tools:
- name: InDelGT
  role: Pipeline orchestrating step 1 (parental InDel catalog generation); invoked
    with 'perl InDelGT.pl -o directory' to align parental reads and merge indel calls
    into parent.cls
  repo: https://github.com/tongchf/InDelGT
- name: BWA 0.7.17
  role: Aligns the parental paired-end reads to the reference genome; path given as
    BWA_FOLD in parameters.ini
  repo: https://github.com/lh3/bwa/releases/tag/v0.7.17
- name: SAMtools 1.9
  role: Processes the parental alignments for indel calling; path given as SAMTOOLS_FOLD
    in parameters.ini
- name: BCFtools 1.9
  role: Used with SAMtools to call indels at parental restriction-site loci; path
    given as BCFTOOLS_FOLD in parameters.ini
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/aps3.11499
    title: 'InDelGT: An integrated pipeline for extracting indel genotypes for genetic
      mapping in a hybrid population using next‐generation sequencing data'
  source_content_hash: sha256:071a5e23b322d85418d16da5ec9192710eb858b0bf9fd630b04fcb2d0c717686
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
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# parental-indel-catalog-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate the parental indel catalog (parent.cls) by aligning the two parental RAD-seq FASTQ files to a reference genome with BWA and calling indels at restriction-site loci with SAMtools/BCFtools — step 1 of the InDelGT pipeline. This catalog defines the indel marker set used for downstream genotyping of the hybrid progeny.

## When to use

Use when you need to build a catalog of indel sites segregating between two parents (e.g., P. deltoides × P. simonii) from parental paired-end FASTQ reads, as the first step before calling indel genotypes across a hybrid population.

## When NOT to use

- The parental indel catalog already exists (parent.cls from a previous run) — skip this step with the --nocatalog flag to go directly to genotype calling.
- The goal is calling indel genotypes across the progeny or testing Mendelian segregation — those are steps 2 and 3 of InDelGT, not catalog generation.
- Input data are not paired-end reads from two parents plus a reference genome aligned to restriction sites (e.g., whole-genome variant calls from other tools).

## Inputs

- Reference genome FASTA (e.g., reference.fasta, P. trichocarpa v4.0)
- Parental paired-end FASTQ files (R1/R2) for parent 1 and parent 2 (e.g., P. deltoides and P. simonii RAD-seq reads)
- parameters.ini configuration file with [folders], [parameters], and [data files] sections
- Installed BWA 0.7.17, SAMtools 1.9, BCFtools 1.9, and Perl modules Parallel::ForkManager, Getopt::Long, Statistics::Distributions

## Outputs

- parent.cls — merged parental indel catalog (~50,532 indel sites in the Populus case study) written to the -o output directory
- Per-parent and merged BWA/SAMtools/BCFtools alignments and indel calls at restriction-site loci (intermediate files)

## How to apply

Install BWA 0.7.17 and SAMtools 1.9 with BCFtools 1.9, plus the Perl modules Parallel::ForkManager, Getopt::Long, and Statistics::Distributions via cpan. Prepare parameters.ini with a [folders] section giving paths to BWA, SAMtools, and BCFtools, a [parameters] section (e.g., GQ:30, MAPQ:5, THREADS:20), and a [data files] section pointing REFERENCE_FILE to reference.fasta and PARENT1/PARENT2 to the parental R1/R2 fastq files. Align the parental paired-end reads to the reference genome with BWA, process the alignments with SAMtools/BCFtools to call indels at parental restriction-site loci, and merge both parents into a single catalog. Launch the run from the work directory with 'nohup perl PathToInDelGT/InDelGT.pl -o directory &' in the background, since the full run can take hours to days; the -o output directory then contains the merged parental indel catalog file parent.cls. In the Populus case study this produced 50,532 indel sites between the two parents against the P. trichocarpa v4.0 reference.

## Related tools

- **InDelGT** (Pipeline orchestrating step 1 (parental InDel catalog generation); invoked with 'perl InDelGT.pl -o directory' to align parental reads and merge indel calls into parent.cls) — https://github.com/tongchf/InDelGT
- **BWA 0.7.17** (Aligns the parental paired-end reads to the reference genome; path given as BWA_FOLD in parameters.ini) — https://github.com/lh3/bwa/releases/tag/v0.7.17
- **SAMtools 1.9** (Processes the parental alignments for indel calling; path given as SAMTOOLS_FOLD in parameters.ini)
- **BCFtools 1.9** (Used with SAMtools to call indels at parental restriction-site loci; path given as BCFTOOLS_FOLD in parameters.ini)

## Examples

```
nohup perl PathToInDelGT/InDelGT.pl -o directory &
```

## Evaluation signals

- The -o output directory contains the merged parental indel catalog file parent.cls.
- parent.cls contains approximately 50,532 indel sites, the count reported for the Populus case study (P. deltoides vs. P. simonii against the P. trichocarpa v4.0 reference).
- parameters.ini is complete: [folders] lists BWA_FOLD, SAMTOOLS_FOLD, BCFTOOLS_FOLD, INDELGT_FOLD, and DATAFOLD; [data files] has REFERENCE_FILE pointing to reference.fasta and PARENT1/PARENT2 fastq entries.
- Step can be skipped cleanly with --nocatalog when parent.cls already exists, confirming the catalog step completed independently.
- Downstream segregation analysis yields ~2805 of the 50,532 indel sites following Mendelian segregation ratios (P > 0.01) in the progeny.

## Limitations

- The full catalog-generation run can take hours to days, so it must be launched with nohup/in the background to avoid interruption.
- Large indels (>5 bp) complicate or distort correct read mapping compared with small indels (1–5 bp), requiring higher coverage depth; heterozygote determination also requires higher coverage depth than homozygotes.
- Coverage depends on the RAD design: paired-end reads from EcoRI restriction sites covered at most 20% of the poplar genome, so effective local coverage varies (avg ~20x genome-wide but up to ~100x at sites).
- No version number is stated for InDelGT, and dependency requirements are described only in the intro/README rather than validated programmatically.

## Evidence

- [abstract] Applying InDelGT to the F1 hybrid Populus population, a total of 50,532 indel sites were identified between the two parents: [evidence span withheld — anchor status: not_found]
- [methods] align the parental paired-end reads to the reference genome with BWA: [evidence span withheld — anchor status: not_found]
