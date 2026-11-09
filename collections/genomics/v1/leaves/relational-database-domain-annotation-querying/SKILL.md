---
name: relational-database-domain-annotation-querying
description: Use when domain-instance homology counts, Pfam family groupings, or position-level
  domain coverage must be derived from a prebuilt protein-to-genome mapping database
  rather than recomputed from raw FASTA/BLAST output.
when_to_use_negative:
- When the mapping database has not been populated — the query needs the loaded PostgreSQL
  table, not raw GENCODE/UniProt/Pfam downloads, which the application does not read
  directly
- When a Pfam family has only a single homologous domain instance in the human proteome
  — it fails the ≥2-homolog threshold and cannot form a meta-domain
- When the goal is to create transcript-to-Swiss-Prot mappings themselves (Protein–Protein
  BLAST v2.2.31+ against GENCODE Basic translations), which is an upstream mapping-database
  construction step
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PostgreSQL
  role: Relational database (v10) storing the mapping between genomic positions, protein
    residues, and domain annotations; queried to group and count homologous Pfam domain
    instances
- name: MetaDome
  role: Application providing the batch_load routine and prebuild pipelines that populate
    and query the mapping database
  repo: https://github.com/cmbi/metadome
- name: HMMER
  role: Aligns retrieved domain-instance sequences against the Pfam HMM into Stockholm-format
    MSAs for each qualifying meta-domain
- name: Docker
  role: Containerizes the MetaDome stack including the PostgreSQL database service
    used for the queries (Docker v17.12.1, docker-compose v1.17.1)
  repo: https://github.com/cmbi/metadome
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:42d67082c525abe20339bb577fe8957a9527407850c933deb5c4d0dd0ae0a5c5
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
  - Docker
  - HMMER
  - MetaDome
  - PostgreSQL
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# relational-database-domain-annotation-querying

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Query a PostgreSQL mapping database that links genomic positions, protein residues, and Pfam domain annotations to group homologous domain instances by Pfam family, the basis for constructing MetaDome meta-domains (3,334 families with ≥2 homologs covering 43.6% of domain positions).

## When to use

Use when domain-instance homology counts, Pfam family groupings, or position-level domain coverage must be derived from a prebuilt protein-to-genome mapping database rather than recomputed from raw FASTA/BLAST output.

## When NOT to use

- When the mapping database has not been populated — the query needs the loaded PostgreSQL table, not raw GENCODE/UniProt/Pfam downloads, which the application does not read directly
- When a Pfam family has only a single homologous domain instance in the human proteome — it fails the ≥2-homolog threshold and cannot form a meta-domain
- When the goal is to create transcript-to-Swiss-Prot mappings themselves (Protein–Protein BLAST v2.2.31+ against GENCODE Basic translations), which is an upstream mapping-database construction step

## Inputs

- Prebuilt MetaDome mapping-database CSV (e.g. MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz)
- PostgreSQL mapping database with 'interpro_domains' table holding Pfam-A annotations for Swiss-Prot human sequences
- Pfam HMM profiles corresponding to each qualifying Pfam family identifier

## Outputs

- Set of meta-domain families (Pfam families with ≥2 homologous domain instances; expected 3,334)
- Stockholm-format MSA files per meta-domain, built with HMMER 3.1b2
- meta_domain_families.tsv per-family table with domain-position coverage fraction (expected 43.6%)

## How to apply

Load the prebuilt mapping database CSV into PostgreSQL v10 via populate_database.sh (which runs metadome.batch_load against the 'interpro_domains' table), then group Pfam-A domain instances by family identifier and retain only families with at least two homologous occurrences across the human proteome — using all 56,319 protein-coding GENCODE Basic transcripts, not only the longest ones, raised the meta-domain count from 2,750 to 3,334. For each qualifying family, retrieve the domain-instance sequences and the corresponding Pfam HMM and align them with HMMER 3.1b2 into Stockholm-format MSAs. Finally, compute the fraction of mapping-database positions that fall inside an annotated homologous protein domain and write the per-family table to meta_domain_families.tsv. The rationale is that aggregation over homologous domains increases per-position variant resolution, so the database query defines exactly which families are aggregatable.

## Related tools

- **PostgreSQL** (Relational database (v10) storing the mapping between genomic positions, protein residues, and domain annotations; queried to group and count homologous Pfam domain instances)
- **MetaDome** (Application providing the batch_load routine and prebuild pipelines that populate and query the mapping database) — https://github.com/cmbi/metadome
- **HMMER** (Aligns retrieved domain-instance sequences against the Pfam HMM into Stockholm-format MSAs for each qualifying meta-domain)
- **Docker** (Containerizes the MetaDome stack including the PostgreSQL database service used for the queries (Docker v17.12.1, docker-compose v1.17.1)) — https://github.com/cmbi/metadome

## Examples

```
./populate_database.sh --data-dir /path/to/csv-dir --file MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz
```

## Evaluation signals

- Count of qualifying meta-domain families must equal the reported 3,334
- Computed fraction of mapping-database positions within annotated homologous protein domains must match the reported 43.6%
- Every retained family must have at least two homologous domain occurrences across the human proteome
- If meta-domains were built from longest transcripts only, the count would drop to 2,750 — using all protein-coding transcripts is required to reproduce 3,334
- The populated mapping table should hold tens of millions of rows per genome build with 32,595,355 unique genomic positions linked to 19,226,961 Swiss-Prot residues

## Limitations

- Aggregating variation via meta-domains loses specific context such as haplotype information or protein–protein interactions
- Aggregation via meta-domains only encapsulates general biological or molecular functions attributed to the domain
- Only transcripts mapped to a Swiss-Prot protein can be used in the analysis (33,492 of 42,130 Swiss-Prot canonical or isoform sequences are linked to GENCODE transcripts)

## Evidence

- [results] The meta-domain aggregation mechanism groups Pfam families with two or more homologous domain instances; incorporating every protein-coding transcript raised the count from 2,750 to 3,334: [evidence span withheld — anchor status: not_found]
