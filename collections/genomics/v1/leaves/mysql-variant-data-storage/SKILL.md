---
name: mysql-variant-data-storage
description: Use when a task requires loading parsed VCF/annotated variant records
  into a relational (MySQL) storage module, or reconstructing a schema with Variant,
  Info, and Sample tables from snpEff-annotated variant data.
when_to_use_negative:
- Input is raw FASTQ/FASTA sequencing data — this skill handles post-variant-calling
  annotated variant records, not read data (JWES data-processing module covers that)
- The variant data is already stored in a persistent relational database and only
  needs visualization (use the JWES Circos visualization module instead)
- A non-relational or non-MySQL target store is required — the ETL routine and schema
  are modeled specifically for MySQL with the Variant/Info/Sample entity relationship
  diagram
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: JWES-DB
  role: VCF parser and ETL loader that extracts, cleanses, reformats, and uploads
    variant records into MySQL
  repo: github.com/drzeeshanahmed/JWES-DB
- name: JWES
  role: Parent Java-based pipeline whose storage module runs the variant-data loading
    workflow
- name: Java
  role: Implementation language of the JWES storage/ETL component
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:52de33c384db3a905858d4d23a2a20bf24078a50e8fe82be6fa063c22baff8ae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JWES
  - JWES-DB
  - Java
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# mysql-variant-data-storage

## Summary

Reconstructs the JWES-ETL (JWES-DB) VCF parser that extracts annotated variant records, cleanses and reformats them, and loads them into a MySQL relational schema (Variant, Info, Sample) for persistent genomic variant storage.

## When to use

Use when a task requires loading parsed VCF/annotated variant records into a relational (MySQL) storage module, or reconstructing a schema with Variant, Info, and Sample tables from snpEff-annotated variant data.

## When NOT to use

- Input is raw FASTQ/FASTA sequencing data — this skill handles post-variant-calling annotated variant records, not read data (JWES data-processing module covers that)
- The variant data is already stored in a persistent relational database and only needs visualization (use the JWES Circos visualization module instead)
- A non-relational or non-MySQL target store is required — the ETL routine and schema are modeled specifically for MySQL with the Variant/Info/Sample entity relationship diagram

## Inputs

- snpEff-annotated variant VCF (or CSV export of variant records) with fields: chromosome, position, reference base, alternate base, filters, data quality, mapping quality, genotype, genotype quality, allele count, combined depth, somatic-mutation record, allelic depths, phred
- MySQL database server credentials (username, password, connection details)

## Outputs

- Populated MySQL database with three relations: Variant, Info, and Sample
- Database dump file jwes_variant_db_dump.sql

## How to apply

Read the annotated variant file produced by the snpEff step and parse per-record fields: chromosome, reference position, reference base, alternate base, filters, data quality, mapping quality, genotype, genotype quality, allele count, combined depth, somatic-mutation record, allelic depths, and phred. Cleanse, classify, and reformat the extracted records as specified for the JWES-ETL module. Create the modeled relational schema of three tables — Variant, Info, and Sample — following the designed entity relationship diagram, then upload the reformatted records into the connected MySQL server using the ETL routine with configured username, password, and connection details. Finally export the populated database as a dump file (jwes_variant_db_dump.sql) and validate that the number of loaded Variant records equals the number of data lines in the input VCF.

## Related tools

- **JWES-DB** (VCF parser and ETL loader that extracts, cleanses, reformats, and uploads variant records into MySQL) — github.com/drzeeshanahmed/JWES-DB
- **JWES** (Parent Java-based pipeline whose storage module runs the variant-data loading workflow)
- **Java** (Implementation language of the JWES storage/ETL component)

## Evaluation signals

- The exported dump file jwes_variant_db_dump.sql exists and contains the Variant, Info, and Sample tables
- The number of loaded Variant records equals the number of data lines in the input VCF
- All required fields (chromosome, position, ref/alt bases, genotype, quality metrics, allelic depths, phred) are present and non-empty in loaded records
- Loaded records match the designed entity relationship diagram for the three-table schema
- Database connection succeeded with the supplied credentials (no load-time connection/insertion errors)

## Limitations

- Validation depends on field-level completeness of the input annotated VCF; missing required fields will break the field-count invariant
- Credentials and connection details must be supplied to the ETL routine; access failures stall the upload step
- The JWES-DB repository README provides only minimal documentation ('VCF Parser for JWES'), so implementation details must be inferred from the paper's workflow

## Evidence

- [readme] README title statement of tool role: "## README: drzeeshanahmed/JWES-DB
# JWES-DB
VCF Parser for JWES"
- [abstract] Storage module role in pipeline architecture: "JWES is a cross-platform, user-friendly, product line application, that entails three modules: (a) data processing, (b) storage, and (c) visualization"
- [abstract] Repository identity evidence from article: "# github.com__drzeeshanahmed__JWES-DB"
