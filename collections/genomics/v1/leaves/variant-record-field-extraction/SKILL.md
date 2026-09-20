---
name: variant-record-field-extraction
description: Use when you have a snpEff-annotated VCF from the JWES pipeline and need
  to parse chromosome, position, ref/alt, filters, quality, genotype, and depth fields
  into persistent relational storage.
when_to_use_negative:
- Input is raw FASTQ/FASTA or unaligned BAM data — the JWES data processing module
  (FastQC, Trimmomatic, BWA, GATK steps) must run first to produce and annotate variants
- The goal is variant calling or annotation itself (HaplotypeCaller, snpEff) rather
  than storage of already-annotated variant records
- No relational database server is available or a non-tabular store (e.g. flat file,
  graph DB) is required, since the ETL routine is built for MySQL
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_0622
tools:
- name: JWES-DB / JWES-ETL
  role: VCF parser and ETL loader that extracts variant fields from the annotated
    file and uploads reformatted records into MySQL
  repo: github.com/drzeeshanahmed/JWES-DB
- name: Java
  role: Implementation language of the Java-based Whole Genome/Exome Sequence Data
    Processing Pipeline (JWES) including its storage module
  repo: github.com/drzeeshanahmed/JWES-DB
- name: MySQL
  role: Relational database management server into which the parsed variant records
    are loaded across the Variant, Info, and Sample tables
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:cdd54ab3090019c63d23c976f57bc62803ff6ecdcedb5b4f4035b977fb027c5f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JWES
  - JWES-DB / JWES-ETL
  - Java
  - MySQL
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# variant-record-field-extraction

## Summary

Parses per-record fields from a snpEff-annotated VCF file and loads the cleansed, reformatted variant records into a MySQL relational database (Variant, Info, Sample tables) using the JWES-ETL (JWES-DB) module, producing the dump file jwes_variant_db_dump.sql.

## When to use

Use when you have a snpEff-annotated VCF from the JWES pipeline and need to parse chromosome, position, ref/alt, filters, quality, genotype, and depth fields into persistent relational storage.

## When NOT to use

- Input is raw FASTQ/FASTA or unaligned BAM data — the JWES data processing module (FastQC, Trimmomatic, BWA, GATK steps) must run first to produce and annotate variants
- The goal is variant calling or annotation itself (HaplotypeCaller, snpEff) rather than storage of already-annotated variant records
- No relational database server is available or a non-tabular store (e.g. flat file, graph DB) is required, since the ETL routine is built for MySQL

## Inputs

- snpEff-annotated VCF file (per-record fields: chromosome, reference position, reference base, alternate base, filters, data quality, mapping quality, genotype, genotype quality, allele count, combined depth, somatic-mutation record, allelic depths, phred)
- MySQL database server connection credentials (username, password, connection details)
- Designed entity relationship diagram for the Variant, Info, and Sample relations

## Outputs

- Populated MySQL relational database with Variant, Info, and Sample tables
- Database dump file jwes_variant_db_dump.sql

## How to apply

Read the annotated variant file produced by the snpEff step and parse the per-record fields: chromosome, reference position, reference base, alternate base, filters, data quality, mapping quality, genotype, genotype quality, allele count, combined depth, somatic-mutation record, allelic depths, and phred. Cleanse, classify, and reformat the extracted records as specified for the JWES-ETL module. Create the modeled relational schema of three tables/relations (Variant, Info, Sample) following the designed entity relationship diagram, then upload the reformatted records into the connected MySQL database management server using the ETL routine with the supplied database credentials. Finally export the populated database as jwes_variant_db_dump.sql and validate that the dump contains the three tables and that loaded Variant records equal the number of data lines in the input VCF.

## Related tools

- **JWES-DB / JWES-ETL** (VCF parser and ETL loader that extracts variant fields from the annotated file and uploads reformatted records into MySQL) — github.com/drzeeshanahmed/JWES-DB
- **Java** (Implementation language of the Java-based Whole Genome/Exome Sequence Data Processing Pipeline (JWES) including its storage module) — github.com/drzeeshanahmed/JWES-DB
- **MySQL** (Relational database management server into which the parsed variant records are loaded across the Variant, Info, and Sample tables)

## Evaluation signals

- Verify the dump file jwes_variant_db_dump.sql exists
- Verify the dump contains the Variant, Info, and Sample tables per the designed entity relationship diagram
- Verify the number of loaded Variant records equals the number of data lines in the input VCF
- Verify every loaded record has all required parsed fields present (chromosome, position, ref/alt, filters, quality, genotype, depths, phred)
- Verify the records were reformatted/cleansed per the JWES-ETL specification before upload

## Limitations

- The JWES-DB README is minimal ('VCF Parser for JWES'), offering no documented CLI usage, parameters, or error handling beyond the ETL routine
- Validation is limited to dump existence, table presence, and record-count equality against VCF data lines — field-level correctness beyond 'all required fields present' is not explicitly checked
- Requires a reachable MySQL server with valid credentials; the parser expects the specific snpEff-annotated field layout produced by the JWES pipeline

## Evidence

- [readme] VCF parser and storage role: "# JWES-DB
VCF Parser for JWES"
- [abstract] pipeline modules including storage: "JWES is a cross-platform, user-friendly, product line application, that entails three modules: (a) data processing, (b) storage, and (c) visualization"
- [other] parsed per-record fields: [evidence span withheld — anchor status: not_found]
- [other] three-table relational schema: [evidence span withheld — anchor status: not_found]
- [other] MySQL upload via ETL: [evidence span withheld — anchor status: not_found]
- [other] validation criteria: [evidence span withheld — anchor status: not_found]
