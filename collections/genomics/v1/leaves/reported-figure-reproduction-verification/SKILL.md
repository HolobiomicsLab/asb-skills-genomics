---
name: reported-figure-reproduction-verification
description: Use when a reconstructed module must be validated by reproducing reported
  quantitative results, e.g. a claimed count of constructed meta-domains (3,334) or
  a coverage fraction (43.6%) derived from Pfam domain annotations.
when_to_use_negative:
- The goal is novel discovery rather than verifying that a reconstructed pipeline
  reproduces already-published figures
- The working dataset differs from the paper's reference data (e.g. a different Pfam
  release than the one tied to the reported 3,334 count), so the reported numbers
  cannot be expected to match
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaDome
  role: Source of the mapping database and prebuilt pipelines whose outputs the reported
    figures validate
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL 10
  role: Stores the mapping database; the 'interpro_domains' table is queried to count
    homologous Pfam domain occurrences per family
- name: Python 3.5.1
  role: Glue language for grouping Pfam families, computing the coverage fraction,
    and writing meta_domain_families.tsv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:a2211325b2c5260083a0e65352fc6ec4a4090b318c8557313f90d964cb07bde8
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# reported-figure-reproduction-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Independently recompute headline statistics from a paper's own data pipeline and compare against the reported numbers — here, the MetaDome meta-domain family count (3,334 Pfam families with ≥2 homologous instances) and the domain-position coverage fraction (43.6%).

## When to use

Use when a reconstructed module must be validated by reproducing reported quantitative results, e.g. a claimed count of constructed meta-domains (3,334) or a coverage fraction (43.6%) derived from Pfam domain annotations.

## When NOT to use

- The goal is novel discovery rather than verifying that a reconstructed pipeline reproduces already-published figures
- The working dataset differs from the paper's reference data (e.g. a different Pfam release than the one tied to the reported 3,334 count), so the reported numbers cannot be expected to match

## Inputs

- PostgreSQL mapping database 'interpro_domains' table with Pfam-A domain annotations for Swiss-Prot human sequences
- Prebuilt MetaDome mapping database CSV (e.g. MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz from Zenodo record 19376150)
- Pfam HMM profiles for each qualifying Pfam family

## Outputs

- meta_domain_families.tsv — per-family meta-domain table
- Validated meta-domain family count (target: 3,334)
- Computed fraction of positions within annotated homologous protein domains (target: 43.6%)

## How to apply

Load Pfam-A domain annotations for all Swiss-Prot human sequences from the 'interpro_domains' table of the PostgreSQL mapping database, then group domain instances by Pfam family identifier and retain only families with at least two homologous occurrences across the human proteome. For each qualifying meta-domain, retrieve all domain-instance sequences, fetch the corresponding Pfam HMM, and align them with HMMER 3.1b2 into Stockholm-format MSA files, since the paper increases the meta-domain count from 2,750 to 3,334 by incorporating every protein-coding transcript rather than only the longest ones. Compute the fraction of mapping-database positions falling within an annotated homologous protein domain and write the per-family table to meta_domain_families.tsv. Treat the derived family count (must equal 3,334) and the computed coverage fraction (must match 43.6%) as pass/fail validation criteria; any deviation indicates a filtering or transcript-inclusion mismatch in the reconstruction.

## Related tools

- **MetaDome** (Source of the mapping database and prebuilt pipelines whose outputs the reported figures validate) — https://github.com/cmbi/metadome
- **PostgreSQL 10** (Stores the mapping database; the 'interpro_domains' table is queried to count homologous Pfam domain occurrences per family)
- **Python 3.5.1** (Glue language for grouping Pfam families, computing the coverage fraction, and writing meta_domain_families.tsv)

## Examples

```
psql -d metadome -c "SELECT pfam_id, COUNT(*) FROM interpro_domains GROUP BY pfam_id HAVING COUNT(*) >= 2;"
```

## Evaluation signals

- Derived meta-domain family count equals the reported 3,334 (the count obtained when including every protein-coding transcript, not the 2,750 longest-transcript-only figure)
- Computed fraction of mapping-database positions inside annotated homologous protein domains matches the reported 43.6%
- Every retained Pfam family has at least two homologous domain instances in the human proteome (no singleton families in meta_domain_families.tsv)
- Per-family MSA files are Stockholm-format and were produced with HMMER 3.1b2 alignment of the retrieved domain-instance sequences against the family's Pfam HMM
- Upstream consistency check: 71,419 Pfam domains linked to 30,406 Swiss-Prot sequences in the mapping database

## Limitations

- The reported counts are tied to a specific genome build and reference-data version (e.g. GRCh38.p14 with GENCODE-v45, UniProt-2025-01, Pfam-37.4); other versions will yield different family counts
- Mapping the prebuilt database is a long load — the mapping table holds tens of millions of rows per build and needs disk headroom for the database plus its indexes
- If the figure being verified is only a proposed future direction or discussion-level claim rather than a data-derived result, no reproduction target exists

## Evidence

- [other] Group domain instances by Pfam family identifier and count homologous occurrences across the human proteome, retaining only families with at least two homologs as meta-domains.: [evidence span withheld — anchor status: not_found]
- [other] by incorporating every protein-coding transcript rather than only the longest ones, the number of constructed meta-domains was increased from 2,750 to 3,334: [evidence span withheld — anchor status: not_found]
