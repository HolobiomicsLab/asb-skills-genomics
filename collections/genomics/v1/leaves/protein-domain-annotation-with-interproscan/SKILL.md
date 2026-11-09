---
name: protein-domain-annotation-with-interproscan
description: Use when building or reproducing a protein-domain inventory over a curated
  FASTA set (e.g. Swiss-Prot sequences matched to GENCODE Basic translations) so that
  residues can later be projected onto Pfam domain consensus positions for meta-domain
  variant aggregation.
when_to_use_negative:
- You only need the finished mapping database rather than a fresh domain annotation
  — download the prebuilt release (e.g. MetaDome_v2.0_GRCh38.p14_..._Pfam-37.4_prebuild-mapping-database.csv.gz
  from Zenodo record 19376150) instead of re-annotating
- Your input is raw GENCODE transcript translations that have not yet been filtered
  and BLAST-matched to Swiss-Prot sequences — run the GENCODE-to-Swiss-Prot Protein–Protein
  BLAST mapping step first
- You need non-Pfam domain resources or Gene Ontology annotation the paper does not
  describe as loaded into MetaDome's 'interpro_domains' table
tools:
- name: InterProScan
  role: Annotates Pfam protein domain instances on the matched Swiss-Prot FASTA sequences
    (v5.20-59.0)
- name: Pfam
  role: Provides the Pfam-A v30.0 domain library used by InterProScan to define domain
    instances
- name: PostgreSQL
  role: Stores the resulting domain annotations in the 'interpro_domains' table of
    the MetaDome mapping database
- name: Python
  role: Parses InterProScan output and loads each domain annotation into the database
    (MetaDome is developed in Python v3.5.1 with Biopython for translation checks)
  repo: https://github.com/cmbi/metadome
- name: MetaDome
  role: Target application whose mapping database and meta-domain aggregation consume
    the annotated domain inventory
  repo: https://github.com/cmbi/metadome
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:98c95b84ef1536b5eeb8276f19a8bb0ba030f51b216d176ce28bddee938e44f6
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# protein-domain-annotation-with-interproscan

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates protein domain instances on matched Swiss-Prot sequences by running InterProScan v5.20-59.0 with the Pfam-A v30.0 library, then loading each domain annotation into the 'interpro_domains' table of the MetaDome PostgreSQL mapping database. This establishes the Pfam domain inventory (71,419 domains on 30,406 Swiss-Prot sequences) that MetaDome's meta-domain aggregation depends on.

## When to use

Use when building or reproducing a protein-domain inventory over a curated FASTA set (e.g. Swiss-Prot sequences matched to GENCODE Basic translations) so that residues can later be projected onto Pfam domain consensus positions for meta-domain variant aggregation.

## When NOT to use

- You only need the finished mapping database rather than a fresh domain annotation — download the prebuilt release (e.g. MetaDome_v2.0_GRCh38.p14_..._Pfam-37.4_prebuild-mapping-database.csv.gz from Zenodo record 19376150) instead of re-annotating
- Your input is raw GENCODE transcript translations that have not yet been filtered and BLAST-matched to Swiss-Prot sequences — run the GENCODE-to-Swiss-Prot Protein–Protein BLAST mapping step first
- You need non-Pfam domain resources or Gene Ontology annotation the paper does not describe as loaded into MetaDome's 'interpro_domains' table

## Inputs

- FASTA file of matched human Swiss-Prot canonical and isoform sequences (from the 'proteins' table of the MetaDome mapping database)
- InterProScan v5.20-59.0 binary/container
- Pfam-A v30.0 domain library (HMMs)
- PostgreSQL v10 mapping database with 'proteins' and 'interpro_domains' tables

## Outputs

- InterProScan output file with one record per Pfam domain instance
- 'interpro_domains' table populated in the PostgreSQL mapping database, keyed by Swiss-Prot sequence entries
- Domain inventory counts (total Pfam domains and distinct annotated sequences)

## How to apply

Retrieve the matched Swiss-Prot canonical and isoform sequences (the identical-sequence matches to GENCODE Basic translations recorded in the 'proteins' table) as FASTA, after applying MetaDome's filters (sequences must start with an ATG start codon, end with a stop codon, and have a cDNA translation that Biopython confirms matches the GENCODE translation). Run InterProScan v5.20-59.0 over these sequences restricted to the Pfam-A v30.0 domain library, parsing the output so each row is one domain instance keyed by its Swiss-Prot sequence entry. Load the parsed annotations into the 'interpro_domains' table of the PostgreSQL v10 mapping database, which also stores the genomic-to-protein position mapping. Finally, tally the domain inventory: total Pfam domain instances and the number of distinct Swiss-Prot sequences carrying at least one domain, and check against the reference counts. The resulting inventory also identifies which Pfam families have two or more homologs (3,334 families in the paper) and are therefore suitable for meta-domain construction.

## Related tools

- **InterProScan** (Annotates Pfam protein domain instances on the matched Swiss-Prot FASTA sequences (v5.20-59.0))
- **Pfam** (Provides the Pfam-A v30.0 domain library used by InterProScan to define domain instances)
- **PostgreSQL** (Stores the resulting domain annotations in the 'interpro_domains' table of the MetaDome mapping database)
- **Python** (Parses InterProScan output and loads each domain annotation into the database (MetaDome is developed in Python v3.5.1 with Biopython for translation checks)) — https://github.com/cmbi/metadome
- **MetaDome** (Target application whose mapping database and meta-domain aggregation consume the annotated domain inventory) — https://github.com/cmbi/metadome

## Examples

```
interproscan.sh -i swissprot_matched.fasta -f TSV -appl Pfam
```

## Evaluation signals

- Recomputed domain inventory matches the reported 71,419 Pfam domain instances on 30,406 of the Swiss-Prot human sequences within expected tolerance
- The parsed InterProScan annotation file contains exactly one record per domain instance
- Every row in 'interpro_domains' resolves to an existing entry in the 'proteins' table (foreign-key integrity over Swiss-Prot sequence entries)
- Only domains from the specified Pfam-A library (v30.0) appear in the loaded table, matching the InterProScan -appl Pfam restriction
- Pfam families with two or more homologs identified from the inventory match the reported count of 3,334 meta-domain-suitable families

## Limitations

- Only Swiss-Prot sequences successfully mapped to GENCODE Basic translations carry domains: 33,492 of 42,130 matched sequences yield domains on 30,406 — unmapped transcripts are excluded from analysis
- Only the transcripts that are mapped to a Swiss-Prot protein can be used in the analysis; sequences failing the start/stop codon or cDNA-translation-match checks are excluded upstream
- Meta-domain aggregation over homologous domains loses specific context such as haplotype information or protein-protein interactions

## Evidence
