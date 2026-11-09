---
name: depth-threshold-heterozygote-genotyping
description: Use when calling per-sample InDel genotypes in a segregating population
  (BC1/BC2/F2/CP) from progeny VCF records at pre-built parental indel catalog sites,
  with minimum depth and genotype-quality cutoffs.
when_to_use_negative:
- Calling SNP genotypes or de novo variants — InDelGT only genotypes InDel sites present
  in the parental catalog
- No parental InDel catalog exists yet — the first pipeline step (catalog generation)
  must be run instead of skipping it with --nocatalog
- Extremely low-coverage samples below HETEROZYGOUS_DEPTH:3 / HOMOZYGOUS_DEPTH:5,
  where nearly all sites would fail depth filters
edam_operation: http://edamontology.org/operation_3226
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: InDelGT
  role: Performs the genotype-calling step, applying depth and GQ thresholds to VCF
    evidence at catalog InDel sites; parallelized with Parallel::ForkManager
  repo: https://github.com/tongchf/InDelGT
- name: BWA 0.7.17
  role: Aligns progeny paired-end FASTQ reads to reference.fasta; alignments below
    MAPQ:5 are discarded
  repo: https://github.com/lh3/bwa/releases/tag/v0.7.17
- name: SAMtools 1.9
  role: Processes BWA alignments for variant evidence extraction
  repo: http://samtools.sourceforge.net/
- name: BCFtools 1.9
  role: Extracts VCF variant records at parental-catalog InDel sites that support
    heterozygous calls
  repo: http://samtools.sourceforge.net/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/aps3.11499
    title: 'InDelGT: An integrated pipeline for extracting indel genotypes for genetic
      mapping in a hybrid population using next‐generation sequencing data'
  source_content_hash: sha256:1c70974512b5d3201e60c54319402abef53b7174d29bbbb2dc18189e96f4e9d7
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
  - Getopt::Long
  - InDelGT
  - SAMtools
  - SAMtools 1.9
  - Statistics::Distributions
  edam_operation: http://edamontology.org/operation_3226
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# depth-threshold-heterozygote-genotyping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns heterozygous and homozygous InDel genotypes across progeny of a hybrid population by applying read-depth thresholds (HETEROZYGOUS_DEPTH:3, HOMOZYGOUS_DEPTH:5) to VCF evidence at parental-catalog InDel sites in InDelGT, then discards calls below GQ:30.

## When to use

Use when calling per-sample InDel genotypes in a segregating population (BC1/BC2/F2/CP) from progeny VCF records at pre-built parental indel catalog sites, with minimum depth and genotype-quality cutoffs.

## When NOT to use

- Calling SNP genotypes or de novo variants — InDelGT only genotypes InDel sites present in the parental catalog
- No parental InDel catalog exists yet — the first pipeline step (catalog generation) must be run instead of skipping it with --nocatalog
- Extremely low-coverage samples below HETEROZYGOUS_DEPTH:3 / HOMOZYGOUS_DEPTH:5, where nearly all sites would fail depth filters

## Inputs

- Paired-end FASTQ read files for all progeny (PROGENY_NUMBER:20 in the example)
- Reference genome FASTA (REFERENCE_FILE:reference.fasta)
- Parental InDel catalog with start and end positions for each InDel site
- parameters.ini with GQ:30, MAPQ:5, HETEROZYGOUS_DEPTH:3, HOMOZYGOUS_DEPTH:5, THREADS:20
- BWA 0.7.17, SAMtools 1.9, BCFtools 1.9 installations

## Outputs

- Per-sample genotype files in the output directory, each recording InDel site positions, allele nucleotides, and genotypes (het/hom) with GQ >= 30
- VCF records from BCFtools supporting heterozygous calls at catalog sites

## How to apply

Run InDelGT's second step (genotype calling; skip catalog generation with --nocatalog) from the directory containing parameters.ini. Align each progeny's paired-end FASTQ to reference.fasta with BWA 0.7.17, retain alignments at MAPQ >= 5, and extract variant evidence at parental-catalog InDel sites with SAMtools 1.9 and BCFtools 1.9. Assign a heterozygous genotype from the VCF records when allele depth reaches HETEROZYGOUS_DEPTH:3; call a homozygous indel genotype only when allele absence is confirmed at every position across the full catalog InDel start-to-end span at depth >= HOMOZYGOUS_DEPTH:5, because homozygous calls are made by absence of evidence rather than positive VCF support. Discard any genotype scoring below GQ:30 and parallelize per-sample calling (20 progeny in the example) with Parallel::ForkManager at THREADS:20.

## Related tools

- **InDelGT** (Performs the genotype-calling step, applying depth and GQ thresholds to VCF evidence at catalog InDel sites; parallelized with Parallel::ForkManager) — https://github.com/tongchf/InDelGT
- **BWA 0.7.17** (Aligns progeny paired-end FASTQ reads to reference.fasta; alignments below MAPQ:5 are discarded) — https://github.com/lh3/bwa/releases/tag/v0.7.17
- **SAMtools 1.9** (Processes BWA alignments for variant evidence extraction) — http://samtools.sourceforge.net/
- **BCFtools 1.9** (Extracts VCF variant records at parental-catalog InDel sites that support heterozygous calls) — http://samtools.sourceforge.net/

## Examples

```
perl PathToInDelGT/InDelGT.pl parameters.ini --nocatalog
```

## Evaluation signals

- Every emitted genotype in per-sample files meets the minimum InDel genotype score (GQ:30)
- Heterozygous calls are supported by VCF records at allele depth >= HETEROZYGOUS_DEPTH:3
- Homozygous indel calls show allele absence at every position from the catalog site start to end with depth >= HOMOZYGOUS_DEPTH:5
- Downstream segregation-type files (e.g. aaxab.txt for BC1/CP) pass Mendelian segregation tests at PVALUE:0.01
- Missing-genotype percentage per InDel stays within MISPCT:20

## Limitations

- Homozygous calls are absence-based across the full start-to-end span, so low coverage or misalignment around a site can mask true homozygotes
- Large indels (>5 bp) complicate or distort correct read mapping compared with 1-5 bp indels, requiring higher coverage depth
- Determining heterozygotes requires higher coverage depth than homozygotes, so depth thresholds trade sensitivity for specificity

## Evidence

- [abstract] The InDelGT algorithm is implemented in three steps: generating an indel catalog, calling indel genotypes, and analyzing indel segregation.: "The InDelGT algorithm is implemented in three steps: generating an indel catalog, calling indel genotypes, and analyzing indel segregation."
- [discussion] Heterozygote depth requirement: "The determination of heterozygotes requires a higher coverage depth than that of homozygotes (McKernan et al., 2009)"
- [readme] Depth parameters in parameters.ini: "# percent of missing genotypes at each InDel
    MISPCT:20
    HETEROZYGOUS_DEPTH:3
    HOMOZYGOUS_DEPTH:5"
- [discussion] Mendelian segregation filtering: "the genotype data generated from InDelGT are filtered to follow the Mendelian segregation ratio, which could further improve the data accuracy overall"
