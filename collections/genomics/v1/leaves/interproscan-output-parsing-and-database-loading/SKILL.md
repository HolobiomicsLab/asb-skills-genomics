---
name: interproscan-output-parsing-and-database-loading
description: 'Use when building a protein-domain annotation layer for a sequence-to-genome
  mapping database: you have matched Swiss-Prot FASTA sequences and need each Pfam
  domain instance parsed from InterProScan output and loaded per sequence entry into
  a relational database.'
when_to_use_negative:
- You only need domain annotations for exploratory browsing rather than populating
  the mapping database — the MetaDome web server or Pfam web services suffice without
  building the 'interpro_domains' table
- Your target sequences are not GENCODE-matched Swiss-Prot canonical/isoform sequences;
  running InterProScan on raw GENCODE translations would not reproduce the Swiss-Prot-keyed
  inventory
- Using MetaDome v2.0 prebuilt releases (Pfam-37.4 domain consensus positions already
  in the Zenodo mapping-database CSV) — the domain layer is already provided and InterProScan
  re-annotation is redundant
edam_operation: http://edamontology.org/operation_0246
edam_topics:
- http://edamontology.org/topic_0623
- http://edamontology.org/topic_0091
tools:
- name: InterProScan
  role: Annotates the matched Swiss-Prot sequences with Pfam domain instances using
    the Pfam-A library
- name: Pfam
  role: Domain library (Pfam-A v30.0 in the paper; Pfam-37.4 in the v2.0 prebuilt
    release) providing the domain definitions projected onto consensus positions
- name: MetaDome
  role: Consumes the populated 'interpro_domains' table; its PostgreSQL mapping database
    stores the domain annotations per Swiss-Prot sequence
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL
  role: Relational database (v10 in the paper) hosting the mapping database and the
    'interpro_domains' table the parsed annotations are loaded into
- name: Python
  role: Implements the InterProScan output parsing and batch loading (MetaDome is
    developed in Python v3.5.1 with a batch_load module for CSV ingestion)
  repo: https://github.com/cmbi/metadome
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:549bdb708f8cad17f88da96b399e8ba1459c40ab1396a3b266c3bd9f77856e8a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/laurensvdwiel/metadome
  repo_url: https://github.com/cmbi/metadome
  tools:
  - InterProScan
  - MetaDome
  - Pfam
  - PostgreSQL
  - Python
  edam_operation: http://edamontology.org/operation_0246
  edam_topics:
  - http://edamontology.org/topic_0623
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# interproscan-output-parsing-and-database-loading

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates protein sequences with Pfam domain instances by running InterProScan over the Swiss-Prot FASTA set and parsing the output into the PostgreSQL mapping database's domain table, establishing the domain inventory underlying MetaDome's meta-domain analyses.

## When to use

Use when building a protein-domain annotation layer for a sequence-to-genome mapping database: you have matched Swiss-Prot FASTA sequences and need each Pfam domain instance parsed from InterProScan output and loaded per sequence entry into a relational database.

## When NOT to use

- You only need domain annotations for exploratory browsing rather than populating the mapping database — the MetaDome web server or Pfam web services suffice without building the 'interpro_domains' table
- Your target sequences are not GENCODE-matched Swiss-Prot canonical/isoform sequences; running InterProScan on raw GENCODE translations would not reproduce the Swiss-Prot-keyed inventory
- Using MetaDome v2.0 prebuilt releases (Pfam-37.4 domain consensus positions already in the Zenodo mapping-database CSV) — the domain layer is already provided and InterProScan re-annotation is redundant

## Inputs

- FASTA of matched human Swiss-Prot canonical and isoform sequences (identical-sequence matches to GENCODE Basic translations from the 'proteins' table)
- InterProScan v5.20-59.0 with Pfam-A v30.0 domain library
- PostgreSQL 10 mapping database (MetaDome schema)

## Outputs

- Populated 'interpro_domains' table with one record per Pfam domain instance, keyed by Swiss-Prot sequence entry
- Domain inventory tally: total Pfam domain instances and count of distinct Swiss-Prot sequences carrying at least one domain

## How to apply

Retrieve the matched human Swiss-Prot canonical and isoform sequences (the identical-sequence matches to GENCODE Basic translations recorded in the 'proteins' table) as FASTA. Run InterProScan v5.20-59.0 over these sequences with the Pfam-A v30.0 domain library to annotate protein domain instances. Parse the InterProScan output and load each domain annotation into the 'interpro_domains' table of the PostgreSQL mapping database, keyed by the corresponding Swiss-Prot sequence entries. Tally the domain inventory — count total Pfam domain instances and the number of distinct Swiss-Prot sequences carrying at least one domain — and compare against the reported inventory. Note the README documents a newer v2.0 release where Pfam-37.4 domain consensus positions ship inside a prebuilt mapping-database CSV, so a reimplementation against current releases may bypass this step by downloading the Zenodo release instead of running InterProScan directly.

## Related tools

- **InterProScan** (Annotates the matched Swiss-Prot sequences with Pfam domain instances using the Pfam-A library)
- **Pfam** (Domain library (Pfam-A v30.0 in the paper; Pfam-37.4 in the v2.0 prebuilt release) providing the domain definitions projected onto consensus positions)
- **MetaDome** (Consumes the populated 'interpro_domains' table; its PostgreSQL mapping database stores the domain annotations per Swiss-Prot sequence) — https://github.com/cmbi/metadome
- **PostgreSQL** (Relational database (v10 in the paper) hosting the mapping database and the 'interpro_domains' table the parsed annotations are loaded into)
- **Python** (Implements the InterProScan output parsing and batch loading (MetaDome is developed in Python v3.5.1 with a batch_load module for CSV ingestion)) — https://github.com/cmbi/metadome

## Examples

```
./populate_database.sh --data-dir /path/to/csv --file MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz
```

## Evaluation signals

- Recomputed inventory matches the reported 71,419 Pfam domains linked to 30,406 Swiss-Prot human sequences
- The parsed annotation file contains exactly one record per domain instance
- Every row in 'interpro_domains' is keyed to an existing Swiss-Prot sequence entry in the 'proteins' table (referential integrity check)
- Count of distinct Swiss-Prot sequences with at least one domain is consistent with the loaded domain-instance count (30,406 sequences for 71,419 instances in the reference build)
- Cross-check domain consensus-position mapping against the released mapping database (tens of millions of rows per genome build) if reproducing the v2.0 pipeline

## Limitations

- Counts are version-dependent: the paper's inventory uses Pfam-A v30.0 and InterProScan v5.20-59.0, while the v2.0 release uses Pfam-37.4, so re-running with different versions will not reproduce the exact 71,419/30,406 counts
- Aggregating variation via meta-domains built on these domain instances loses specific context such as haplotype information or protein-protein interactions
- Domain annotation only encapsulates general biological or molecular functions attributed to the domain
- Only transcripts mapped to a Swiss-Prot protein can be used in the analysis

## Evidence
