---
name: relational-database-schema-design
description: Use when annotated variant output (VCF/snpEff-annotated records) from
  a variant-calling pipeline must be persisted into a relational database with a modeled
  schema for downstream querying and dump/export.
when_to_use_negative:
- Input is already a structured database dump or a feature table loaded in persistent
  storage rather than raw annotated variant records
- The goal is upstream data processing (QC, trimming, alignment, variant calling)
  rather than variant-data storage/loading — use the JWES data-processing module instead
- The variant data must remain in VCF text form for tools that consume VCF directly
  and no relational persistence is required
tools:
- name: JWES
  role: Pipeline whose storage module (JWES-DB) implements the VCF parsing and ETL
    loading into MySQL
  repo: github.com/drzeeshanahmed/JWES-DB
- name: Java
  role: Implementation language of the Java-based JWES pipeline that hosts the JWES-DB
    storage module
- name: MySQL
  role: Relational database management server into which the reformatted Variant/Info/Sample
    records are uploaded via the ETL routine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:79f48807020ca0ac04d8c96012149f029bfd00d4f5c53c6989e302f0db0eebed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JWES
  - Java
  - MySQL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# relational-database-schema-design

## Summary

Designs a relational schema for storing annotated VCF variant records as the JWES-DB (JWES-ETL) storage module, organizing parsed variant data into Variant, Info, and Sample tables and loading them into a MySQL RDBMS via an ETL routine.

## When to use

Use when annotated variant output (VCF/snpEff-annotated records) from a variant-calling pipeline must be persisted into a relational database with a modeled schema for downstream querying and dump/export.

## When NOT to use

- Input is already a structured database dump or a feature table loaded in persistent storage rather than raw annotated variant records
- The goal is upstream data processing (QC, trimming, alignment, variant calling) rather than variant-data storage/loading — use the JWES data-processing module instead
- The variant data must remain in VCF text form for tools that consume VCF directly and no relational persistence is required

## Inputs

- Annotated VCF / CSV variant file (snpEff-annotated per-record fields: chromosome, position, reference base, alternate base, filters, data quality, mapping quality, genotype, genotype quality, allele count, combined depth, somatic-mutation record, allelic depths, phred)
- Designed entity relationship diagram for the Variant/Info/Sample schema
- MySQL database connection credentials (username, password, connection details)

## Outputs

- Populated MySQL relational database with Variant, Info, and Sample tables
- Database dump file jwes_variant_db_dump.sql

## How to apply

Read the annotated variant file produced by the snpEff step and parse per-record fields: chromosome, reference position, reference base, alternate base, filters, data quality, mapping quality, genotype, genotype quality, allele count, combined depth, somatic-mutation record, allelic depths, and phred. Cleanse, classify, and reformat the extracted records as specified for the JWES-ETL module. Create the modeled relational schema organized into three tables/relations — Variant, Info, and Sample — following the designed entity relationship diagram. Upload the reformatted records into the connected MySQL database management server using the ETL routine with database credentials (username, password, connection details), then export the populated database as a dump file jwes_variant_db_dump.sql.

## Related tools

- **JWES** (Pipeline whose storage module (JWES-DB) implements the VCF parsing and ETL loading into MySQL) — github.com/drzeeshanahmed/JWES-DB
- **Java** (Implementation language of the Java-based JWES pipeline that hosts the JWES-DB storage module)
- **MySQL** (Relational database management server into which the reformatted Variant/Info/Sample records are uploaded via the ETL routine)

## Evaluation signals

- The exported dump jwes_variant_db_dump.sql exists and contains the Variant, Info, and Sample tables
- The number of loaded Variant records equals the number of data lines in the input VCF
- All required fields are present for each loaded variant record (chromosome, position, ref/alt bases, filters, quality metrics, genotype fields)
- Database connectivity is verified via the supplied credentials before the ETL upload step

## Limitations

- Validation depends on the input VCF having all required per-record fields; malformed or missing fields break the record count/field completeness check
- The ETL requires working MySQL credentials and a reachable database server; connection failures halt loading
- The current JWES version offers only a command-line interface; an interactive interface is not yet implemented

## Evidence

- [other] Create the modeled relational database schema organized into three tables/relations: Variant, Info, and Sample, following the designed entity relationship diagram.: [evidence span withheld — anchor status: not_found]
- [readme] VCF Parser for JWES: "# JWES-DB
VCF Parser for JWES"
- [other] JWES-ETL module operates by extracting all variants from a CSV file and storing them in a relational database management server: [evidence span withheld — anchor status: not_found]
- [abstract] JWES is a cross-platform, user-friendly, product line application, that entails three modules: (a) data processing, (b) storage, and (c) visualization: "JWES is a cross-platform, user-friendly, product line application, that entails three modules: (a) data processing, (b) storage, and (c) visualization"
