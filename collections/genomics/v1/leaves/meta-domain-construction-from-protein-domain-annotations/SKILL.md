---
name: meta-domain-construction-from-protein-domain-annotations
description: Use when building per-family aggregates of Pfam domain annotations from
  a positional mapping database (e.g. PostgreSQL 'interpro_domains' table) to define
  homologous-domain groups for variant-tolerance aggregation.
when_to_use_negative:
- Pfam families with only a single homologous domain instance in the human proteome
  — these cannot form a meta-domain since aggregation requires at least two homologs
- Aggregation of variation outside annotated protein domains, where no Pfam consensus
  positions exist to project variants onto
edam_operation: http://edamontology.org/operation_0253
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0621
tools:
- name: MetaDome
  role: Provides the meta-domain concept and the mapping database from which homologous
    Pfam domain instances are grouped
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL 10
  role: Stores the mapping database whose 'interpro_domains' table supplies Pfam-A
    domain annotations for grouping
- name: Python 3.5.1
  role: Implements the meta-domain aggregation and MSA-generation logic
  repo: https://github.com/cmbi/metadome
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:a74f1d7ebf429b5f25083c143220acbc0b3bd67e647ab30c6eb0bbc089e9a170
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
  edam_operation: http://edamontology.org/operation_0253
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# meta-domain-construction-from-protein-domain-annotations

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs Pfam meta-domains by grouping every Pfam family with two or more homologous domain instances across the human proteome and aligning all domain-instance sequences to the family HMM, enabling aggregation of variation data over homologous domains.

## When to use

Use when building per-family aggregates of Pfam domain annotations from a positional mapping database (e.g. PostgreSQL 'interpro_domains' table) to define homologous-domain groups for variant-tolerance aggregation.

## When NOT to use

- Pfam families with only a single homologous domain instance in the human proteome — these cannot form a meta-domain since aggregation requires at least two homologs
- Aggregation of variation outside annotated protein domains, where no Pfam consensus positions exist to project variants onto

## Inputs

- Pfam-A domain annotations for all Swiss-Prot human sequences from the 'interpro_domains' table of the PostgreSQL mapping database
- Pfam HMM profiles corresponding to each qualifying Pfam family identifier
- Domain-instance protein sequences for every homologous occurrence (all protein-coding transcripts, not only the longest)

## Outputs

- Per-family meta-domain table written to meta_domain_families.tsv
- Stockholm-format MSA files per meta-domain produced with HMMER 3.1b2
- Computed fraction of mapping-database positions within annotated homologous protein domains (expected 43.6%)

## How to apply

Load Pfam-A domain annotations for all Swiss-Prot human sequences from the 'interpro_domains' table of the PostgreSQL mapping database, then group domain instances by Pfam family identifier and count homologous occurrences across the human proteome, retaining only families with at least two homologs as meta-domains. For each qualifying meta-domain, retrieve all domain-instance sequences, fetch the corresponding Pfam HMM, and align the instances with HMMER 3.1b2 into Stockholm-format MSA files so that variants sit on Pfam domain consensus positions. Incorporating every protein-coding transcript rather than only the longest ones raises the meta-domain count from 2,750 to 3,334, so the transcript set must cover all full-length protein-coding GENCODE Basic translations. Finally, compute the fraction of mapping-database positions that fall within an annotated homologous protein domain and write the per-family meta-domain table to meta_domain_families.tsv.

## Related tools

- **MetaDome** (Provides the meta-domain concept and the mapping database from which homologous Pfam domain instances are grouped) — https://github.com/cmbi/metadome
- **PostgreSQL 10** (Stores the mapping database whose 'interpro_domains' table supplies Pfam-A domain annotations for grouping)
- **Python 3.5.1** (Implements the meta-domain aggregation and MSA-generation logic) — https://github.com/cmbi/metadome

## Examples

```
python -c "from metadome.domain.annotation import build_meta_domains; build_meta_domains(min_homologs=2, out='meta_domain_families.tsv')"
```

## Evaluation signals

- Derived meta-domain family count must equal the reported 3,334 Pfam families with two or more homologs
- Computed fraction of mapping-database positions within annotated homologous domains must match the reported 43.6%
- Count rises from 2,750 to 3,334 when all protein-coding transcripts (56,319 GENCODE Basic) are included instead of only the longest ones
- Each retained family must have at least two homologous domain instances; families with one homolog must be excluded
- Every meta-domain MSA must be in Stockholm format and generated with HMMER 3.1b2

## Limitations

- Aggregating variation via meta-domains loses specific context such as haplotype information or protein-protein interactions
- Aggregation via meta-domains only encapsulates general biological or molecular functions attributed to the domain

## Evidence
