---
name: pfam-domain-family-homolog-grouping
description: Use when Pfam-A domain annotations for Swiss-Prot human sequences must
  be reduced to homologous domain families (≥2 occurrences) as a precursor to meta-domain
  MSA construction or genetic tolerance landscape computation.
when_to_use_negative:
- Pfam families with only a single domain occurrence in the human proteome — they
  cannot be aggregated into a meta-domain
- Analyses restricted to a single gene or protein where no cross-gene domain homology
  aggregation is intended
- Workflows that only need raw genomic-position-to-residue mappings; use the prebuilt
  mapping database directly instead of regrouping families
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0199
tools:
- name: MetaDome
  role: Provides the mapping database and meta-domain construction code that performs
    the Pfam family grouping and downstream MSA building
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL 10
  role: Relational database storing the mapping database whose interpro_domains table
    is queried to group domain instances by Pfam family and count homologous occurrences
- name: Python 3.5.1
  role: Implementation language for the grouping, database access, and meta-domain
    construction scripts
  repo: https://github.com/cmbi/metadome
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:08235175f627bff7f7056305a5b28bea94aa2af64c7b38b8c287aa20cd86839b
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
  - MetaDome
  - PostgreSQL
  - PostgreSQL 10
  - Python
  - Python 3.5.1
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# pfam-domain-family-homolog-grouping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Groups Pfam-A domain instances annotated in the MetaDome PostgreSQL mapping database by Pfam family identifier, retaining only families with two or more homologous domain occurrences in the human proteome so that variant information can be aggregated into meta-domains.

## When to use

Use when Pfam-A domain annotations for Swiss-Prot human sequences must be reduced to homologous domain families (≥2 occurrences) as a precursor to meta-domain MSA construction or genetic tolerance landscape computation.

## When NOT to use

- Pfam families with only a single domain occurrence in the human proteome — they cannot be aggregated into a meta-domain
- Analyses restricted to a single gene or protein where no cross-gene domain homology aggregation is intended
- Workflows that only need raw genomic-position-to-residue mappings; use the prebuilt mapping database directly instead of regrouping families

## Inputs

- Pfam-A domain annotations for Swiss-Prot human sequences (interpro_domains table of the MetaDome PostgreSQL mapping database)
- MetaDome prebuild mapping database CSV (e.g. MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz)
- Pfam HMMs corresponding to each qualifying Pfam family identifier
- GENCODE Basic protein-coding transcript-to-Swiss-Prot mappings

## Outputs

- Per-family meta-domain table (meta_domain_families.tsv) of Pfam families with ≥2 homologous domain instances
- Stockholm-format multiple sequence alignments per meta-domain, built from domain-instance sequences against the Pfam HMM
- Count of qualifying meta-domain families (expected 3,334 with all transcripts included)

## How to apply

Load Pfam-A domain annotations for all Swiss-Prot human sequences from the 'interpro_domains' table of the PostgreSQL mapping database (e.g. a MetaDome v2.0 prebuild CSV loaded via populate_database.sh). Group domain instances by Pfam family identifier and count homologous occurrences across the human proteome, discarding families with fewer than two instances since aggregation requires multiple aligned homologs. For each qualifying family, retrieve all domain-instance sequences and the corresponding Pfam HMM (three-step process) to build the meta-domain. Crucially, include every protein-coding transcript rather than only the longest ones — this raised the number of constructed meta-domains from 2,750 to 3,334. Validate by checking the derived family count equals 3,334 and that the fraction of mapping-database positions inside annotated homologous domains matches 43.6%.

## Related tools

- **MetaDome** (Provides the mapping database and meta-domain construction code that performs the Pfam family grouping and downstream MSA building) — https://github.com/cmbi/metadome
- **PostgreSQL 10** (Relational database storing the mapping database whose interpro_domains table is queried to group domain instances by Pfam family and count homologous occurrences)
- **Python 3.5.1** (Implementation language for the grouping, database access, and meta-domain construction scripts) — https://github.com/cmbi/metadome

## Examples

```
./populate_database.sh --data-dir /path/to/prebuild --file MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz
```

## Evaluation signals

- The number of Pfam families retained (≥2 homologous instances) must equal 3,334 when all protein-coding transcripts are included (2,750 when only the longest transcripts are used)
- The computed fraction of mapping-database positions falling within an annotated homologous protein domain must match 43.6%
- Sanity check upstream inputs: 71,419 Pfam domains linked to 30,406 Swiss-Prot sequences in the mapping database
- Every retained family must contain at least two homologous domain instances; no single-instance families may appear in meta_domain_families.tsv
- Each per-family MSA must be in Stockholm format and align domain-instance sequences against the correct corresponding Pfam HMM

## Limitations

- Families with a single domain occurrence are excluded, so only 43.6% of mapping-database positions fall within annotated homologous domains
- Aggregation via meta-domains loses specific context such as haplotype information or protein-protein interactions and only encapsulates general biological or molecular functions attributed to the domain
- Coverage is limited to transcripts mappable to Swiss-Prot proteins: 42,116 of 56,319 GENCODE Basic transcripts linked to 33,492 of 42,130 Swiss-Prot canonical or isoform sequences

## Evidence
