---
name: sample-metadata-curation
description: Use when compiling multiple long-read Iso-Seq runs (per-sample SQANTI3
  RulesFilter classification.txt files) into a shared, updatable isoSeQL database
  and each sample's source material and analysis provenance must be captured for cross-sample
  comparability.
when_to_use_negative:
- Input is a single sample with no intention of cross-sample comparison — per-sample
  PB.X.Y IDs and the SQANTI3 classification file alone suffice, so a shared metadata
  database adds overhead
- Reads have not yet passed the isoseq > minimap2 > collapse > SQANTI3 RulesFilter
  pipeline — there are no high-quality filtered isoforms or counts to which sample
  metadata can be linked
- The analysis is short-read transcriptomics or de novo assembly without SQANTI3 output
  — isoSeQL was built specifically to work with SQANTI2/SQANTI3 output
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: isoSeQL
  role: Compiles per-sample SQANTI3 output and writes the sampleData, exp, and single-cell
    metadata tables into the updatable isoseql.db SQLite database
  repo: https://github.com/christine-liu/isoSeQL
- name: SQANTI3
  role: Produces the RulesFilter-filtered classification.txt files that define the
    samples whose metadata is curated into the database
  repo: https://github.com/ConesaLab/SQANTI3
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf680
    title: 'isoSeQL: comparing long-read isoforms across multiple datasets'
  source_content_hash: sha256:1ac8bbf8f3a837d0f958961c1d05a8ea332b3c1ad6d49eba1eef6932ea9d1c47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/christine-liu/isoSeQL
  tools:
  - Python
  - SQANTI3
  - isoSeQL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# sample-metadata-curation

## Summary

Curates and records per-sample provenance metadata—cell line/tissue source material, software and reference versions, experiment date, and single-cell barcodes/cell types—into the sampleData, exp, and single-cell tables of an isoSeQL SQLite database (isoseql.db) as SQANTI3-filtered Iso-Seq samples are compiled.

## When to use

Use when compiling multiple long-read Iso-Seq runs (per-sample SQANTI3 RulesFilter classification.txt files) into a shared, updatable isoSeQL database and each sample's source material and analysis provenance must be captured for cross-sample comparability.

## When NOT to use

- Input is a single sample with no intention of cross-sample comparison — per-sample PB.X.Y IDs and the SQANTI3 classification file alone suffice, so a shared metadata database adds overhead
- Reads have not yet passed the isoseq > minimap2 > collapse > SQANTI3 RulesFilter pipeline — there are no high-quality filtered isoforms or counts to which sample metadata can be linked
- The analysis is short-read transcriptomics or de novo assembly without SQANTI3 output — isoSeQL was built specifically to work with SQANTI2/SQANTI3 output

## Inputs

- Per-sample SQANTI3 RulesFilter-filtered classification.txt files
- Sample source-material metadata (cell line / tissue origin per sample)
- Software versions, reference genome/transcriptome versions, and experiment date
- For single-cell runs: cellular barcodes and cell-type assignments

## Outputs

- sampleData table in isoseql.db keyed by sample with cell line/tissue source information
- exp table in isoseql.db with software versions, reference versions, and experiment date
- cellular barcode and cell-type tables for single-cell inputs
- Updatable SQLite database (isoseql.db) whose metadata tables support appending future samples

## How to apply

As each SQANTI3 RulesFilter-filtered classification.txt is loaded with pandas and isoforms are consolidated by intron chain into stable common-junction IDs, populate the sampleData table with the cell line/tissue source-material information for that sample. Record the exp table with the software versions (e.g. isoSeQL built on Python 3.9.6, plotting in R 4.1.1), reference versions, and experiment date so the database is self-documenting across appended runs. For single-cell inputs, additionally populate the cellular barcode and cell-type tables alongside the isoform, counts, isoform_ends, and ends_counts tables. Write everything into isoseql.db so the metadata tables persist as new samples are appended on subsequent runs, keeping sample identity linked to the stable isoform IDs rather than the sample-specific PB.X.Y identifiers.

## Related tools

- **isoSeQL** (Compiles per-sample SQANTI3 output and writes the sampleData, exp, and single-cell metadata tables into the updatable isoseql.db SQLite database) — https://github.com/christine-liu/isoSeQL
- **SQANTI3** (Produces the RulesFilter-filtered classification.txt files that define the samples whose metadata is curated into the database) — https://github.com/ConesaLab/SQANTI3

## Evaluation signals

- sampleData table contains one row per input sample with non-empty cell line/tissue source-material information matching the supplied metadata sheet
- exp table records concrete software versions (e.g. Python 3.9.6, R 4.1.1), reference versions, and an experiment date
- For single-cell inputs, cellular barcode and cell-type tables exist and barcodes reconcile with the read counts in the counts and ends_counts tables
- Per-sample counts in the counts and ends_counts tables are consistent with the input SQANTI3 classification.txt files
- Appending a new sample on a later run preserves existing sampleData/exp rows and stable common-junction isoform IDs rather than re-issuing sample-specific PB.X.Y IDs

## Limitations

- Metadata tables are only as complete as the user-supplied sample annotations; the database stores but does not validate cell line/tissue provenance
- No changelog was found for the isoSeQL repository, so schema changes affecting the sampleData/exp tables between versions cannot be tracked from documentation alone
- Single-cell metadata support (cellular barcode and cell-type tables) requires single-cell Iso-Seq inputs prepared through the modified SQANTI3 pipeline; bulk-only runs will not populate those tables

## Evidence

- [methods] sampleData and exp table population: [evidence span withheld — anchor status: not_found]
- [methods] single-cell metadata tables: [evidence span withheld — anchor status: not_found]
- [methods] updatable database design: [evidence span withheld — anchor status: not_found]
- [readme] tool scope: "Database tool for comparing across many Iso-Seq runs analyzed through SQANTI3"
