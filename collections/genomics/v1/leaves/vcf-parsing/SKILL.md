---
name: vcf-parsing
description: Use when annotated variant data from a VCF (e.g. snpEff-annotated somatic
  variant records) must be parsed, cleansed, classified, reformatted, and persisted
  into a relational database such as MySQL.
when_to_use_negative:
- The data are still raw reads (FASTQ/FASTA) that have not been processed and variant-called
  by the JWES data processing module
- The goal is variant annotation or visualization (e.g. Circos graphs) rather than
  structured storage of parsed variant records
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: JWES
  role: Parent Java-based pipeline; JWES-DB is its storage-module VCF parser component
  repo: https://github.com/drzeeshanahmed/JWES-DB
- name: Java
  role: Implementation language of the JWES/JWES-DB pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:eca0ec5ba88ce4f3a54653ac4703956f0c2c64c3f707dc12a671beec6ea34ecf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drzeeshanahmed/JWES-DB
  tools:
  - JWES
  - Java
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# vcf-parsing

## Summary

The JWES-DB (JWES-ETL) skill parses annotated VCF variant records and loads them into a relational MySQL storage schema (Variant, Info, Sample tables), serving as the storage module of the JWES pipeline.

## When to use

Use when annotated variant data from a VCF (e.g. snpEff-annotated somatic variant records) must be parsed, cleansed, classified, reformatted, and persisted into a relational database such as MySQL.

## When NOT to use

- The data are still raw reads (FASTQ/FASTA) that have not been processed and variant-called by the JWES data processing module
- The goal is variant annotation or visualization (e.g. Circos graphs) rather than structured storage of parsed variant records

## Inputs

- Annotated variant VCF/CSV file (snpEff-annotated) with per-record fields: chromosome, position, ref/alt bases, filters, data quality, mapping quality, genotype, genotype quality, allele count, combined depth, somatic-mutation record, allelic depths, phred
- MySQL database connection credentials (username, password, connection details)

## Outputs

- Populated MySQL relational database with Variant, Info, and Sample tables
- Database dump file jwes_variant_db_dump.sql

## How to apply

Read the annotated variant file (the snpEff-annotated VCF/CSV) and parse per-record fields: chromosome, reference position, reference base, alternate base, filters, data quality, mapping quality, genotype, genotype quality, allele count, combined depth, somatic-mutation record, allelic depths, and phred. Cleanse, classify, and reformat the extracted records as specified for the JWES-ETL module. Create the modeled relational schema with three relations — Variant, Info, and Sample — following the designed entity relationship diagram. Upload the reformatted records into the connected MySQL server using the ETL routine with database credentials, then export the populated database as a dump file (jwes_variant_db_dump.sql). Rationale: JWES is a Java-based FAIR pipeline whose storage module makes processed variant data findable, accessible, interoperable, and reusable.

## Related tools

- **JWES** (Parent Java-based pipeline; JWES-DB is its storage-module VCF parser component) — https://github.com/drzeeshanahmed/JWES-DB
- **Java** (Implementation language of the JWES/JWES-DB pipeline)

## Evaluation signals

- The exported dump file jwes_variant_db_dump.sql exists and contains the Variant, Info, and Sample tables
- The number of loaded Variant records equals the number of data lines in the input VCF
- All required per-record fields (chromosome, position, ref/alt, genotype, depth, phred, etc.) are present in the loaded records
- The schema matches the designed entity relationship diagram (three relations: Variant, Info, Sample)
- Records can be queried from the connected MySQL management server after upload

## Limitations

- The current JWES version only offers a command-line interface; no interactive interface is implemented
- Loading targets a MySQL relational management server; other RDBMS backends are not described
- Validation relies on matching loaded Variant counts to input VCF data lines, so malformed or extra lines in the source file will surface as record-count mismatches

## Evidence

- [other] The JWES-ETL module operates by extracting all variants from a CSV file and storing them in a relational database management server: [evidence span withheld — anchor status: not_found]
- [abstract] Java based Whole Genome/Exome Sequence Data Processing Pipeline (JWES): "Java based Whole Genome/Exome Sequence Data Processing Pipeline (JWES)"
- [other] Create the modeled relational database schema organized into three tables/relations: Variant, Info, and Sample: [evidence span withheld — anchor status: not_found]
