---
name: relational-database-schema-population
description: Use when building or populating a relational mapping database that links
  genomic positions to protein residues (e.g., GENCODE transcripts to Swiss-Prot canonical/isoform
  sequences) with indexed lookup fields for downstream variant aggregation.
when_to_use_negative:
- Downstream analyses such as tolerance landscapes, meta-domain construction or meta-domain
  variant annotation — these consume the already-populated mapping database rather
  than populating it.
- Input is a VCF of gnomAD/ClinVar variants to be annotated — the populated database
  is a prerequisite for that annotation step, not the target of this skill.
- The released mapping CSV has already been loaded and verified against the committed
  quality-control reports — re-populating from scratch is unnecessary.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PostgreSQL
  role: PostgreSQL v10 relational database in which the mapping database ('genes',
    'proteins', 'mappings' tables) is stored, with indexes on all fields used in lookups
- name: metadome.batch_load
  role: Batch-loads the released prebuild mapping CSV into the database via populate_database.sh
    inside a temporary docker-compose stack
  repo: https://github.com/cmbi/metadome
- name: BLAST
  role: Protein–Protein BLAST v2.2.31+ matches each GENCODE translation against the
    human Swiss-Prot canonical/isoform sequences; only identical matches are registered
    in the database
- name: Biopython
  role: translate() verifies that each transcript's cDNA translation is identical
    to the GENCODE translation, excluding mismatched transcripts before database registration
- name: ClustalW2
  role: v2.1 alignments of each identical match supply the per-nucleotide genomic-to-protein
    position mappings stored in the 'mappings' table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:fda8b6929e0506bfc596cd5355265bb0a4ae91757c99c7342db16c3d04bd731f
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
  - BLAST
  - Biopython
  - ClustalW2
  - MetaDome
  - PostgreSQL
  - Python
  - metadome.batch_load
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# relational-database-schema-population

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs and loads a PostgreSQL relational mapping database that links every coding genomic position to a GENCODE transcript, a Swiss-Prot protein residue and, where applicable, a Pfam domain consensus position. This indexed mapping database is the substrate on which all MetaDome variant aggregation and tolerance analyses run.

## When to use

Use when building or populating a relational mapping database that links genomic positions to protein residues (e.g., GENCODE transcripts to Swiss-Prot canonical/isoform sequences) with indexed lookup fields for downstream variant aggregation.

## When NOT to use

- Downstream analyses such as tolerance landscapes, meta-domain construction or meta-domain variant annotation — these consume the already-populated mapping database rather than populating it.
- Input is a VCF of gnomAD/ClinVar variants to be annotated — the populated database is a prerequisite for that annotation step, not the target of this skill.
- The released mapping CSV has already been loaded and verified against the committed quality-control reports — re-populating from scratch is unnecessary.

## Inputs

- GENCODE release 19 GRCh37.p13 Basic protein-coding transcript set (cDNA and translations)
- UniProtKB/Swiss-Prot Release 2016_09 human canonical and isoform sequence entries (FASTA)
- Released prebuild mapping database CSV, e.g. MetaDome_v2.0_GRCh37.p13_GENCODE-v19_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz

## Outputs

- PostgreSQL v10 database with 'genes' (transcript identifiers, sequence length), 'proteins' (identical Swiss-Prot matches) and 'mappings' (nucleotide genomic-to-protein positions) tables, indexed on all lookup fields
- Complete mapping of every coding genomic position to a transcript, a UniProt residue and a Pfam domain consensus position
- Quality-control report recording the recomputed mapping statistics for the release

## How to apply

Filter the GENCODE Basic protein-coding translations to those starting with an ATG start codon and ending with a stop codon, and confirm via Biopython's translate function that each cDNA translation is identical to the GENCODE translation, excluding mismatches. Run Protein–Protein BLAST v2.2.31+ of each surviving translation against the human Swiss-Prot canonical/isoform set and retain only identical sequence matches. Register transcript identifiers and sequence lengths in a PostgreSQL v10 'genes' table, matched Swiss-Prot entries in a 'proteins' table, and, for each identical match, every nucleotide's genomic-to-protein position (derived from a ClustalW2 v2.1 alignment) in a 'mappings' table, indexing all fields used in lookups. Alternatively, load a released prebuild CSV (e.g., MetaDome_v2.0_GRCh37.p13_GENCODE-v19_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz) with populate_database.sh, which brings up a temporary stack, loads through metadome.batch_load, and tears down again; provision disk headroom for tens of millions of rows per genome build plus their indexes.

## Related tools

- **PostgreSQL** (PostgreSQL v10 relational database in which the mapping database ('genes', 'proteins', 'mappings' tables) is stored, with indexes on all fields used in lookups)
- **metadome.batch_load** (Batch-loads the released prebuild mapping CSV into the database via populate_database.sh inside a temporary docker-compose stack) — https://github.com/cmbi/metadome
- **BLAST** (Protein–Protein BLAST v2.2.31+ matches each GENCODE translation against the human Swiss-Prot canonical/isoform sequences; only identical matches are registered in the database)
- **Biopython** (translate() verifies that each transcript's cDNA translation is identical to the GENCODE translation, excluding mismatched transcripts before database registration)
- **ClustalW2** (v2.1 alignments of each identical match supply the per-nucleotide genomic-to-protein position mappings stored in the 'mappings' table)

## Examples

```
./populate_database.sh --data-dir /path/to/the/directory/holding/the/csv --file MetaDome_v2.0_GRCh37.p13_GENCODE-v19_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz
```

## Evaluation signals

- Recomputed totals reproduce the published figures: 42,116 of 56,319 full-length protein-coding GENCODE Basic transcripts linked to 33,492 of the 42,130 Swiss-Prot human canonical or isoform sequences.
- The mappings table yields 32,595,355 unique genomic positions linked to 19,226,961 residues in Swiss-Prot protein sequences (published release statistics).
- Every retained transcript's cDNA translation is identical to its GENCODE translation under Biopython's translate function; no excluded sequence starts outside ATG or lacks a stop codon.
- All fields used in downstream lookups are indexed; the mapping table holds tens of millions of rows per build and the database plus indexes fit within the allocated disk.
- The committed quality-control reports (quality_control_report_GRCh37.p13.md / GRCh38.p14.md) confirm the released dataset reproduces the aggregation statistics and the tolerance track codon for codon.

## Limitations

- GENCODE, UniProt and Pfam are not read by the application itself; they are inputs to the prebuild pipelines, which fetch their own copies.
- METADOME_POSTGRES_DB_DIR must point to an empty directory the first time the database starts; the load is long and needs disk headroom for the database plus its indexes.
- Mapping coverage is limited to transcripts with identical Swiss-Prot sequence matches (42,116 of 56,319), so unmapped transcripts cannot contribute to downstream analyses.

## Evidence

- [methods] These mappings are created with Protein–Protein BLAST v2.2.31 + for each protein‐coding translation in the GENCODE Basic set: [evidence span withheld — anchor status: not_found]
