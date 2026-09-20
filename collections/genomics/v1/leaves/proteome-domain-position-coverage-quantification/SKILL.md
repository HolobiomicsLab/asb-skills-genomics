---
name: proteome-domain-position-coverage-quantification
description: Use when determining which Pfam domain families have ≥2 homologous domain
  instances across the human proteome, or when computing the fraction of proteome
  positions covered by annotated homologous domains.
when_to_use_negative:
- When haplotype context or protein-protein interaction information must be preserved
  — the article notes that aggregation via meta-domains loses such specific context
- When only one homologous domain instance exists for a Pfam family — single-instance
  families are not suitable for meta-domain construction
- When the input is raw variant data (VCF) rather than a populated mapping database;
  domain-position coverage cannot be computed without the mapping-database annotations
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: PostgreSQL
  role: Stores the MetaDome mapping database, including the 'interpro_domains' table
    queried to group Pfam-A domain annotations by family identifier
- name: Python
  role: Implements the grouping/aggregation of domain instances by Pfam family, the
    HMMER alignment step, and the coverage-fraction computation
  repo: https://github.com/cmbi/metadome
- name: MetaDome
  role: Provides the mapping database and meta-domain concept whose family counts
    (3,334) and position coverage (43.6%) this skill reconstructs
  repo: https://github.com/cmbi/metadome
- name: HMMER 3.1b2
  role: Aligns retrieved domain-instance sequences against the corresponding Pfam
    HMM into Stockholm-format MSA files per meta-domain
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:d58abca9cfa48669878fae4084aa97c1a0b5c0776d581f72bce96d15da60ac9b
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
  - HMMER 3.1b2
  - MetaDome
  - PostgreSQL
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# proteome-domain-position-coverage-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how much of a proteome's protein positions fall inside annotated homologous domains and identifies Pfam families with two or more homologous domain occurrences as candidates for meta-domain construction, as done in MetaDome (3,334 meta-domains covering 43.6% of mapping-database positions).

## When to use

Use when determining which Pfam domain families have ≥2 homologous domain instances across the human proteome, or when computing the fraction of proteome positions covered by annotated homologous domains.

## When NOT to use

- When haplotype context or protein-protein interaction information must be preserved — the article notes that aggregation via meta-domains loses such specific context
- When only one homologous domain instance exists for a Pfam family — single-instance families are not suitable for meta-domain construction
- When the input is raw variant data (VCF) rather than a populated mapping database; domain-position coverage cannot be computed without the mapping-database annotations

## Inputs

- MetaDome prebuilt mapping database CSV (e.g. MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz) loaded into PostgreSQL
- Pfam-A domain annotations for all Swiss-Prot human sequences in the 'interpro_domains' table of the PostgreSQL mapping database
- Pfam HMM profiles corresponding to each qualifying Pfam family identifier

## Outputs

- meta_domain_families.tsv: per-family meta-domain table of Pfam families with ≥2 homologous domain instances
- Stockholm-format MSA files per meta-domain produced with HMMER 3.1b2
- Coverage statistic: fraction of mapping-database positions within an annotated homologous protein domain (reported as 43.6%)

## How to apply

Load the Pfam-A domain annotations for all Swiss-Prot human sequences from the 'interpro_domains' table of the MetaDome PostgreSQL mapping database, which stores a complete mapping between genomic positions, protein positions, and domain annotations. Group domain instances by Pfam family identifier and count homologous occurrences across the human proteome, retaining only families with at least two homologs as meta-domains. For each qualifying meta-domain, retrieve all domain-instance sequences, fetch the corresponding Pfam HMM, and align the instances with HMMER 3.1b2 into Stockholm-format MSA files. Compute the fraction of mapping-database positions that fall within an annotated homologous protein domain and write the per-family table to meta_domain_families.tsv. Critically, incorporate every protein-coding transcript rather than only the longest ones — this raised the constructed meta-domain count from 2,750 to 3,334 — and validate against those reported figures.

## Related tools

- **PostgreSQL** (Stores the MetaDome mapping database, including the 'interpro_domains' table queried to group Pfam-A domain annotations by family identifier)
- **Python** (Implements the grouping/aggregation of domain instances by Pfam family, the HMMER alignment step, and the coverage-fraction computation) — https://github.com/cmbi/metadome
- **MetaDome** (Provides the mapping database and meta-domain concept whose family counts (3,334) and position coverage (43.6%) this skill reconstructs) — https://github.com/cmbi/metadome
- **HMMER 3.1b2** (Aligns retrieved domain-instance sequences against the corresponding Pfam HMM into Stockholm-format MSA files per meta-domain)

## Examples

```
SELECT pfam_id, COUNT(*) AS n FROM interpro_domains GROUP BY pfam_id HAVING COUNT(*) >= 2;
```

## Evaluation signals

- The number of meta-domain families written to meta_domain_families.tsv must equal 3,334
- The computed fraction of mapping-database positions within annotated homologous domains must match the reported 43.6%
- Every retained Pfam family has at least two homologous domain instances in the human proteome
- Including only the longest transcripts yields 2,750 meta-domains; including all protein-coding transcripts must yield the higher count of 3,334
- Each per-family MSA is in Stockholm format and was generated by aligning all domain-instance sequences to the family's Pfam HMM with HMMER 3.1b2

## Limitations

- Counts depend on dataset versions: the article reports 3,334 families and 43.6% coverage for its Swiss-Prot/Pfam release, while newer prebuilds (e.g. Pfam-37.4, UniProt-2025-01) will shift these numbers
- Using only the longest transcripts per gene undercounts meta-domains (2,750 vs 3,334), so transcript-selection strategy directly biases the result
- Aggregating variation via meta-domains loses specific context such as haplotype information and protein-protein interactions, and only encapsulates general biological or molecular functions attributed to the domain

## Evidence
