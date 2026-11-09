---
name: genome-annotation-mapping-statistics
description: Use when reproducing or auditing the GENCODE-to-Swiss-Prot transcript
  mapping underlying MetaDome, or when building a genomic-position-to-protein-residue
  mapping database from cDNA/FASTA translations using full-length identical BLAST
  matches.
when_to_use_negative:
- When the analysis requires homologous (non-identical) or cross-species domain mapping
  — this skill retains only 100% identical full-length sequence matches.
- When using newer reference versions (e.g., GRCh38/GENCODE v45 with UniProt 2025-01
  and Pfam 37.4) and expecting the published 2019 counts — the 42,116/56,319 figures
  are specific to GENCODE v19 and Swiss-Prot 2016_09.
- When transcripts already carry official GENCODE–UniProt cross-references and no
  position-level genomic-to-protein mapping table is needed.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: BLAST
  role: Protein–Protein BLAST v2.2.31+ runs each GENCODE Basic protein-coding translation
    against Swiss-Prot human canonical/isoform sequences to find identical sequence
    matches
- name: Biopython
  role: translate function verifies that each transcript's cDNA translation is identical
    to the GENCODE translation; mismatches are excluded
- name: ClustalW2
  role: v2.1 alignments built for each identical match to derive nucleotide genomic-to-protein
    position mappings stored in the 'mappings' table
- name: PostgreSQL
  role: v10 relational database storing the mapping between genomic positions, protein
    positions, transcripts ('genes' table) and matched Swiss-Prot sequences ('proteins'
    table)
- name: MetaDome
  role: Application whose mapping criteria, batch_load routine (metadome.batch_load
    via populate_database.sh), and database schema define the transcript-to-Swiss-Prot
    mapping being reproduced
  repo: https://github.com/cmbi/metadome
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:0fb7c6e0e56e305042598e6043e8491378d809b82e6d5a8cf733ba9b203b5af7
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# genome-annotation-mapping-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the MetaDome mapping of full-length protein-coding GENCODE Basic transcripts onto UniProtKB/Swiss-Prot human canonical/isoform sequences via Protein–Protein BLAST identical-sequence matches, storing genomic-to-protein position maps in a PostgreSQL database and validating the published counts (42,116 of 56,319 transcripts; 33,492 of 42,130 Swiss-Prot sequences).

## When to use

Use when reproducing or auditing the GENCODE-to-Swiss-Prot transcript mapping underlying MetaDome, or when building a genomic-position-to-protein-residue mapping database from cDNA/FASTA translations using full-length identical BLAST matches.

## When NOT to use

- When the analysis requires homologous (non-identical) or cross-species domain mapping — this skill retains only 100% identical full-length sequence matches.
- When using newer reference versions (e.g., GRCh38/GENCODE v45 with UniProt 2025-01 and Pfam 37.4) and expecting the published 2019 counts — the 42,116/56,319 figures are specific to GENCODE v19 and Swiss-Prot 2016_09.
- When transcripts already carry official GENCODE–UniProt cross-references and no position-level genomic-to-protein mapping table is needed.

## Inputs

- GENCODE release 19 GRCh37.p13 Basic protein-coding transcript set (cDNA with translations; 56,319 transcripts for 19,728 human genes)
- UniProtKB/Swiss-Prot Release 2016_09 human canonical and isoform sequence entries (42,130 sequences)
- Prebuilt mapping database CSV (e.g., MetaDome_v2.0_GRCh37.p13_GENCODE-v19_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz from Zenodo record 19376150)

## Outputs

- PostgreSQL 'genes', 'proteins', and 'mappings' tables containing transcript/protein registrations and nucleotide genomic-to-protein position mappings
- Mapping statistics: 42,116/56,319 transcripts mapped, 33,492/42,130 Swiss-Prot sequences matched, 32,595,355 unique genomic positions, 19,226,961 linked residues
- Quality-control report comparing recomputed totals against the published figures

## How to apply

Download the GENCODE release 19 GRCh37.p13 Basic protein-coding transcript set and UniProtKB/Swiss-Prot Release 2016_09 human canonical and isoform entries (42,130 sequences), then exclude GENCODE translations that do not start with an ATG start codon (encoding methionine) or end with a stop codon. Verify each remaining transcript by translating its cDNA with Biopython's translate function and exclude any transcript whose cDNA translation is not identical to the GENCODE translation. Run Protein–Protein BLAST v2.2.31+ for each protein-coding translation against the Swiss-Prot human canonical/isoform set and retain only identical (full-length 100% identity) sequence matches. Register transcript identifiers and sequence lengths in a PostgreSQL 'genes' table and each identical Swiss-Prot match in a 'proteins' table, indexing all fields used in lookups; for each match build a ClustalW2 v2.1 alignment and store each nucleotide's genomic-to-protein position mapping in a 'mappings' table. Recompute the statistics — transcripts mapped of 56,319, Swiss-Prot sequences matched of 42,130, unique genomic positions and linked residues — and write them to a quality-control report; accept only if the totals reproduce 42,116 of 56,319 transcripts linked to 33,492 of 42,130 Swiss-Prot sequences, with 32,595,355 unique genomic positions linked to 19,226,961 residues.

## Related tools

- **BLAST** (Protein–Protein BLAST v2.2.31+ runs each GENCODE Basic protein-coding translation against Swiss-Prot human canonical/isoform sequences to find identical sequence matches)
- **Biopython** (translate function verifies that each transcript's cDNA translation is identical to the GENCODE translation; mismatches are excluded)
- **ClustalW2** (v2.1 alignments built for each identical match to derive nucleotide genomic-to-protein position mappings stored in the 'mappings' table)
- **PostgreSQL** (v10 relational database storing the mapping between genomic positions, protein positions, transcripts ('genes' table) and matched Swiss-Prot sequences ('proteins' table))
- **MetaDome** (Application whose mapping criteria, batch_load routine (metadome.batch_load via populate_database.sh), and database schema define the transcript-to-Swiss-Prot mapping being reproduced) — https://github.com/cmbi/metadome

## Examples

```
./populate_database.sh --data-dir /path/to/csv --file MetaDome_v2.0_GRCh37.p13_GENCODE-v19_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz
```

## Evaluation signals

- Recomputed mapping counts reproduce the published totals: 42,116 of 56,319 transcripts (79.4%) and 33,492 of 42,130 Swiss-Prot human canonical/isoform sequences.
- Resulting mappings contain exactly 32,595,355 unique genomic positions linked to 19,226,961 residues in Swiss-Prot protein sequences.
- Every accepted transcript starts with ATG, ends with a stop codon, and its Biopython cDNA translation is identical to the GENCODE translation.
- Each 'mappings' row traces to a ClustalW2 v2.1 alignment of an identical-sequence BLAST match, and all lookup fields in the 'genes' and 'proteins' tables are indexed.
- Quality-control report written alongside the release documents the recomputed aggregation statistics.

## Limitations

- Version-specific: the 42,116/56,319 and 33,492/42,130 figures hold only for GENCODE release 19 (GRCh37.p13) against Swiss-Prot Release 2016_09; newer releases (e.g., GRCh38, GENCODE v45, UniProt 2025-01, Pfam 37.4) will give different totals.
- Only identical-sequence matches are retained, so transcripts lacking a full-length Swiss-Prot match (14,203 of 56,319) are excluded and appear grayed out in downstream MetaDome analyses.
- Loading the mapping table is a long operation — tens of millions of rows per genome build — requiring substantial disk headroom for the database plus its indexes.

## Evidence

- [methods] blast_step: [evidence span withheld — anchor status: not_found]
- [methods] cdna_check: [evidence span withheld — anchor status: not_found]
