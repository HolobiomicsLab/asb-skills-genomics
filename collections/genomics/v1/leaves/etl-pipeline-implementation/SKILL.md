---
name: etl-pipeline-implementation
description: Use when annotated variant records (chromosome, position, ref/alt base,
  filters, genotype, allele count, combined depth, allelic depths, phred) from a JWES/snpEff
  VCF must be cleansed and loaded into a persistent relational database.
when_to_use_negative:
- Input is raw FASTQ/BAM or unannotated variant calls that have not passed the snpEff
  annotation step — run the upstream JWES data-processing and annotation modules first
- Variant data are already loaded in a relational store and only query/visualization
  (e.g., Circos) is needed rather than re-loading
- Target is not a relational database server (e.g., flat-file or NoSQL storage outside
  the JWES Variant/Info/Sample design)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: JWES
  role: Pipeline whose storage module is this VCF-parser/ETL loader; implemented as
    the JWES-DB component
  repo: github.com/drzeeshanahmed/JWES-DB
- name: Java
  role: Implementation language of the Java-based Whole Genome/Exome Sequence Data
    Processing Pipeline (JWES), including the JWES-DB parser
- name: snpEff
  role: Upstream annotation step that produces the annotated variant file the ETL
    parses
- name: MySQL
  role: Relational database management server into which the ETL routine uploads the
    reformatted Variant, Info, and Sample records
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/2211-5463.13261
    title: 'JWES: a new pipeline for whole genome/exome sequence data processing,
      management, and gene‐variant discovery, annotation, prediction, and genotyping'
  source_content_hash: sha256:d4561b087fee0c4e94fd41983521d46d0b3c14b07d4015ecbd7bfe9a7ad422c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JWES
  - Java
  - MySQL
  - snpEff
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# etl-pipeline-implementation

## Summary

Implements the JWES-ETL (JWES-DB) variant-data loader: extracts per-record fields from an snpEff-annotated VCF/CSV, reformatting them for a three-table relational schema (Variant, Info, Sample), and loads them into a MySQL server, exported as jwes_variant_db_dump.sql.

## When to use

Use when annotated variant records (chromosome, position, ref/alt base, filters, genotype, allele count, combined depth, allelic depths, phred) from a JWES/snpEff VCF must be cleansed and loaded into a persistent relational database.

## When NOT to use

- Input is raw FASTQ/BAM or unannotated variant calls that have not passed the snpEff annotation step — run the upstream JWES data-processing and annotation modules first
- Variant data are already loaded in a relational store and only query/visualization (e.g., Circos) is needed rather than re-loading
- Target is not a relational database server (e.g., flat-file or NoSQL storage outside the JWES Variant/Info/Sample design)

## Inputs

- snpEff-annotated VCF file of called variants (or CSV export of the same records) containing chromosome, reference position, reference base, alternate base, filters, data quality, mapping quality, genotype, genotype quality, allele count, combined depth, somatic-mutation record, allelic depths, and phred fields
- MySQL database connection credentials (username, password, host/connection details)

## Outputs

- Populated MySQL relational database with Variant, Info, and Sample tables
- Database dump file jwes_variant_db_dump.sql

## How to apply

Read the annotated variant file produced by the snpEff step and parse the per-record fields: chromosome, reference position, reference base, alternate base, filters, data quality, mapping quality, genotype, genotype quality, allele count, combined depth, somatic-mutation record, allelic depths, and phred. Cleanse, classify, and reformat the records as specified for the JWES-ETL module. Create the modeled relational schema of three relations — Variant, Info, and Sample — following the designed entity relationship diagram. Upload the reformatted records into the connected MySQL server using the ETL routine with database credentials (username, password, connection details), then export the populated database as jwes_variant_db_dump.sql.

## Related tools

- **JWES** (Pipeline whose storage module is this VCF-parser/ETL loader; implemented as the JWES-DB component) — github.com/drzeeshanahmed/JWES-DB
- **Java** (Implementation language of the Java-based Whole Genome/Exome Sequence Data Processing Pipeline (JWES), including the JWES-DB parser)
- **snpEff** (Upstream annotation step that produces the annotated variant file the ETL parses)
- **MySQL** (Relational database management server into which the ETL routine uploads the reformatted Variant, Info, and Sample records)

## Examples

```
mysql -u <username> -p -h <host> jwes_variant_db < jwes_variant_db_dump.sql
```

## Evaluation signals

- Verify the dump file jwes_variant_db_dump.sql exists after the ETL run
- Verify the dump contains the three modeled tables: Variant, Info, and Sample
- Verify the number of loaded Variant records equals the number of data lines in the input VCF
- Verify all required per-record fields (chromosome, position, ref, alt, filters, qualities, genotype, allele count, combined depth, somatic record, allelic depths, phred) are present in the loaded records
- Confirm the schema matches the designed entity relationship diagram (three relations)

## Limitations

- The ETL depends on upstream snpEff annotation; the parser expects annotated variant records with the full JWES field set
- Loading requires working MySQL connection credentials; connection failures block the upload step
- The current JWES version only offers a command-line interface; an easy-to-use interactive interface is not yet implemented

## Evidence

- [other] The JWES-ETL module operates by extracting all variants from a CSV file and storing them in a relational database management server: [evidence span withheld — anchor status: not_found]
- [readme] JWES-DB README identifies the component as the VCF parser: "# JWES-DB
VCF Parser for JWES"
- [abstract] JWES is Java-based with a dedicated storage module: "Java based Whole Genome/Exome Sequence Data Processing Pipeline (JWES)"
- [abstract] JWES comprises three modules including storage: "JWES is a cross-platform, user-friendly, product line application, that entails three modules: (a) data processing, (b) storage, and (c) visualization"
- [discussion] CLI-only limitation confirmed in discussion: "The current version of the JWES offers command line but an easy‐to‐use interface."
