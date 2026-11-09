---
name: indel-genotype-calling-from-vcf-records
description: Use when calling InDel genotypes across progeny of a hybrid mapping population
  (BC1, BC2, F2, or CP) from reads aligned to a reference genome, at InDel sites predefined
  in a parental InDel catalog.
when_to_use_negative:
- When the parental InDel catalog has not yet been generated — the catalog step must
  run first (skip it only with --nocatalog on a rerun)
- When variant sites are SNPs or catalog InDels shorter/longer than the intended marker
  set, since homozygote calling depends on the catalog start-to-end span
- When the goal is segregation analysis rather than genotyping — that is the separate
  third InDelGT step
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: InDelGT
  role: Pipeline orchestrating genotype calling at parental catalog InDel sites; writes
    per-sample genotype files
  repo: https://github.com/tongchf/InDelGT
- name: BWA 0.7.17
  role: Aligns progeny paired-end FASTQ reads to the reference genome; alignments
    kept at MAPQ ≥ 5
  repo: https://github.com/lh3/bwa/releases/tag/v0.7.17
- name: SAMtools 1.9
  role: Processes alignments for variant evidence extraction
- name: BCFtools 1.9
  role: Extracts VCF records supporting heterozygous calls at catalog InDel sites
- name: Parallel::ForkManager
  role: Perl module parallelizing genotype calling across progeny samples (THREADS:20)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/aps3.11499
    title: 'InDelGT: An integrated pipeline for extracting indel genotypes for genetic
      mapping in a hybrid population using next‐generation sequencing data'
  source_content_hash: sha256:cf76569e2586507ab8582e84e27bc5103a78b478bc4f0ca230154626f2a82e40
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
  - Parallel::ForkManager
  - SAMtools
  - SAMtools 1.9
  - Statistics::Distributions
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# indel-genotype-calling-from-vcf-records

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Second-step InDelGT procedure that calls per-progeny InDel genotypes at parental catalog sites: heterozygous genotypes are taken directly from sample VCF records (depth ≥ HETEROZYGOUS_DEPTH:3), while homozygous indel genotypes require allele absence across the full catalog InDel start-to-end span at depth ≥ HOMOZYGOUS_DEPTH:5, with all calls filtered by GQ:30.

## When to use

Use when calling InDel genotypes across progeny of a hybrid mapping population (BC1, BC2, F2, or CP) from reads aligned to a reference genome, at InDel sites predefined in a parental InDel catalog.

## When NOT to use

- When the parental InDel catalog has not yet been generated — the catalog step must run first (skip it only with --nocatalog on a rerun)
- When variant sites are SNPs or catalog InDels shorter/longer than the intended marker set, since homozygote calling depends on the catalog start-to-end span
- When the goal is segregation analysis rather than genotyping — that is the separate third InDelGT step

## Inputs

- Paired-end FASTQ read files for each progeny sample (and parents for catalog context)
- Reference genome FASTA (REFERENCE_FILE:reference.fasta)
- Parental InDel catalog with per-site start and end positions
- parameters.ini specifying GQ:30, MAPQ:5, HETEROZYGOUS_DEPTH:3, HOMOZYGOUS_DEPTH:5, PROGENY_NUMBER:20, THREADS:20, and software paths

## Outputs

- Per-sample genotype files in the output directory, each recording InDel site positions, allele nucleotides, and genotypes
- VCF records extracted with BCFtools 1.9 supporting heterozygous calls at catalog InDel sites

## How to apply

Skip the catalog stage by running the InDelGT genotype-calling step (perl PathToInDelGT/InDelGT.pl with --nocatalog semantics) from the work directory containing parameters.ini. Align each progeny's paired-end FASTQ reads to reference.fasta with BWA 0.7.17, retaining alignments with mapping quality at or above MAPQ:5, then process with SAMtools 1.9 and BCFtools 1.9 to obtain VCF records supporting heterozygous calls. Assign a heterozygous genotype from a VCF record when depth reaches HETEROZYGOUS_DEPTH:3; assign a homozygous indel genotype only when every position from the site's start to end (as recorded in the parental indel catalog) shows homozygous allele absence at depth of at least HOMOZYGOUS_DEPTH:5 — this span check is the key distinction from heterozygote calling, which relies on direct VCF evidence. Discard genotypes scoring below GQ:30, and parallelize across all progeny (20 in the example) with Parallel::ForkManager using THREADS:20, writing per-sample genotype files that record InDel site positions, allele nucleotides, and genotypes.

## Related tools

- **InDelGT** (Pipeline orchestrating genotype calling at parental catalog InDel sites; writes per-sample genotype files) — https://github.com/tongchf/InDelGT
- **BWA 0.7.17** (Aligns progeny paired-end FASTQ reads to the reference genome; alignments kept at MAPQ ≥ 5) — https://github.com/lh3/bwa/releases/tag/v0.7.17
- **SAMtools 1.9** (Processes alignments for variant evidence extraction)
- **BCFtools 1.9** (Extracts VCF records supporting heterozygous calls at catalog InDel sites)
- **Parallel::ForkManager** (Perl module parallelizing genotype calling across progeny samples (THREADS:20))

## Examples

```
perl PathToInDelGT/InDelGT.pl --nocatalog  # run from work directory containing parameters.ini, skipping catalog and filtering stages
```

## Evaluation signals

- Every emitted genotype meets the minimum InDel genotype score GQ:30
- Homozygous indel calls are supported by allele absence at every position across the catalog site's start-to-end span, not just the start position
- Heterozygous calls have read depth ≥ HETEROZYGOUS_DEPTH:3 and homozygous confirmations ≥ HOMOZYGOUS_DEPTH:5
- One genotype file per progeny sample (PROGENY_NUMBER:20 files in the example) is produced in the output directory
- Downstream segregation-type genotype files (e.g. aaxab.txt for BC1/CP) conform to Mendelian segregation tested at PVALUE:0.01

## Limitations

- Large indels (>5 bp) complicate or distort correct read mapping compared with small indels (1–5 bp), requiring higher coverage depth
- Determining heterozygotes requires higher coverage depth than homozygotes, so low-depth samples may yield missing genotypes (MISPCT:20 tolerated)
- Genotypes below the GQ:30 score threshold are discarded, and genotype data are further filtered to follow the Mendelian segregation ratio

## Evidence

- [results] Homozygous calls require full-span absence: [evidence span withheld — anchor status: not_found]
