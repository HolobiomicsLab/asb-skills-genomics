---
name: transcript-protein-sequence-mapping
description: Use when you need to link transcript cDNA/translation sets (e.g., GENCODE
  GTF/FASTA) to curated protein sequences (Swiss-Prot canonical + isoforms) and build
  per-nucleotide genomic-to-protein position mappings.
when_to_use_negative:
- You only need approximate homology (non-identical matches) between transcripts and
  proteins — this skill strictly retains identical sequence matches only
- Your variant-analysis task starts from an already-built mapping database (e.g.,
  the MetaDome v2.0 prebuild CSV from Zenodo record 19376150) — load it with populate_database.sh
  instead of rebuilding
- Input transcripts lack complete CDS annotations or fail start/stop codon sanity
  checks upstream of mapping — they will be excluded, so the workflow is wrong for
  partial or UDN/incomplete models
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_0622
tools:
- name: BLAST
  role: Protein–Protein BLAST v2.2.31+ aligns each GENCODE Basic protein-coding translation
    against human Swiss-Prot canonical/isoform sequences; identical matches are retained
- name: Biopython
  role: translate function verifies that each transcript's cDNA translates exactly
    to the GENCODE-annotated protein before mapping
- name: ClustalW2
  role: v2.1 builds the alignment per identical match from which nucleotide-to-protein-position
    mappings are derived
- name: PostgreSQL
  role: v10 relational database storing the mapping between genomic positions, protein
    positions, and domain annotations in 'genes', 'proteins', and 'mappings' tables
    with indexes on lookup fields
  repo: https://github.com/cmbi/metadome
- name: MetaDome
  role: web server whose underlying mapping database is the output of this skill;
    source code and prebuild pipelines live in the repository
  repo: https://github.com/cmbi/metadome
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:2c291858d36f05cd5206488904eb02a085ab1c81723e76b4a9184cbbc01224bc
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
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# transcript-protein-sequence-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce the MetaDome pipeline that maps full-length protein-coding GENCODE Basic transcripts to UniProtKB/Swiss-Prot human canonical/isoform sequences via strict sequence-identity filtering and Protein–Protein BLAST, storing genomic-to-protein position mappings in PostgreSQL.

## When to use

Use when you need to link transcript cDNA/translation sets (e.g., GENCODE GTF/FASTA) to curated protein sequences (Swiss-Prot canonical + isoforms) and build per-nucleotide genomic-to-protein position mappings.

## When NOT to use

- You only need approximate homology (non-identical matches) between transcripts and proteins — this skill strictly retains identical sequence matches only
- Your variant-analysis task starts from an already-built mapping database (e.g., the MetaDome v2.0 prebuild CSV from Zenodo record 19376150) — load it with populate_database.sh instead of rebuilding
- Input transcripts lack complete CDS annotations or fail start/stop codon sanity checks upstream of mapping — they will be excluded, so the workflow is wrong for partial or UDN/incomplete models

## Inputs

- GENCODE release 19 (GRCh37.p13) Basic protein-coding transcript set (GTF + cDNA/translation FASTA), 56,319 full-length transcripts for 19,728 genes
- UniProtKB/Swiss-Prot Release 2016_09 human canonical and isoform sequence entries (42,130 sequences, FASTA)

## Outputs

- PostgreSQL relational database with 'genes', 'proteins', and 'mappings' tables linking genomic positions to transcript and Swiss-Prot protein residues
- Quality-control report with recomputed mapping statistics (transcripts mapped, Swiss-Prot sequences matched, unique genomic positions, linked residues)
- ClustalW2 v2.1 alignments for each identical transcript-to-Swiss-Prot match

## How to apply

Download the GENCODE release 19 GRCh37.p13 Basic protein-coding transcript set (56,319 full-length transcripts for 19,728 genes) and UniProtKB/Swiss-Prot Release 2016_09 human canonical and isoform sequences (42,130 entries). Apply quality filters: exclude GENCODE translations that do not begin with an ATG start codon or end with a stop codon, then verify each remaining transcript by translating its cDNA with Biopython's translate function and dropping any transcript whose translation is not identical to the GENCODE translation. Run Protein–Protein BLAST v2.2.31+ for each protein-coding translation against the human Swiss-Prot canonical/isoform set and retain only identical sequence matches (79.4% of transcripts). Register transcript identifiers and sequence lengths in a PostgreSQL 'genes' table and matched Swiss-Prot proteins in a 'proteins' table, indexing all lookup fields, then build a ClustalW2 v2.1 alignment per match and store each nucleotide's genomic-to-protein position mapping in the 'mappings' table. Recompute summary statistics into a quality-control report and accept only if they reproduce the published totals.

## Related tools

- **BLAST** (Protein–Protein BLAST v2.2.31+ aligns each GENCODE Basic protein-coding translation against human Swiss-Prot canonical/isoform sequences; identical matches are retained)
- **Biopython** (translate function verifies that each transcript's cDNA translates exactly to the GENCODE-annotated protein before mapping)
- **ClustalW2** (v2.1 builds the alignment per identical match from which nucleotide-to-protein-position mappings are derived)
- **PostgreSQL** (v10 relational database storing the mapping between genomic positions, protein positions, and domain annotations in 'genes', 'proteins', and 'mappings' tables with indexes on lookup fields) — https://github.com/cmbi/metadome
- **MetaDome** (web server whose underlying mapping database is the output of this skill; source code and prebuild pipelines live in the repository) — https://github.com/cmbi/metadome

## Examples

```
blastp -query gencode_v19_basic_translations.fasta -db swissprot_human_canonical_isoforms -outfmt 6 -out identical_matches.tsv
```

## Evaluation signals

- Mapping statistics reproduce published totals: 42,116 of 56,319 transcripts for 19,728 genes linked to 33,492 of 42,130 Swiss-Prot sequences
- Resulting mappings contain 32,595,355 unique genomic positions linked to 19,226,961 residues in Swiss-Prot protein sequences
- Every retained transcript passes both the start/stop codon check and the Biopython cDNA-translation identity check
- All fields used in lookups ('genes', 'proteins', 'mappings' tables) are indexed; the mapping table holds tens of millions of rows per genome build
- Only transcripts mapped to a Swiss-Prot protein are usable in downstream MetaDome analyses; unmapped transcripts appear in gray in the web server

## Limitations

- Strict identity requirement means transcripts whose translation diverges from any Swiss-Prot canonical/isoform sequence (20.6% here) remain unmapped and unusable in MetaDome analyses
- Aggregation via meta-domains loses specific context such as haplotype information or protein-protein interactions
- The mapping database load is long-running and needs disk headroom for the database plus its indexes (tens of millions of rows per genome build)

## Evidence

- [methods] Biopython cDNA translation identity check: [evidence span withheld — anchor status: not_found]
- [methods] Protein–Protein BLAST against GENCODE Basic set: [evidence span withheld — anchor status: not_found]
