---
name: rest-api-variant-annotation-querying
description: Use when you need to fetch current structured annotations (allele frequencies,
  ClinVar CLNSIG classifications, COSMIC presence) for one or more variants from a
  REST API, or when building a re-annotation loop that must repeatedly re-query saved
  variant queries.
when_to_use_negative:
- When privacy policies prevent uploading genetic data to public platforms — the article
  suggests a local deployment of VariantAlert instead of relying on the hosted service
- When annotations must be produced offline or the source requires manual file re-upload
  (e.g., iVar-style workflows) rather than automated API re-querying
- When raw sequence data re-analysis is needed (reference genome or variant-calling
  updates) — this skill only covers knowledge-base annotation updates, not pipeline
  re-execution
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3275
tools:
- name: VariantAlert
  role: Django/Python web application whose client methods forward user requests to
    MyVariant.info, manage REST responses, and periodically re-send all saved queries
    for daily re-annotation
  repo: https://github.com/next-crs4/VariantAlert
- name: MyVariant.info
  role: External REST resource that aggregates and serves structured variant annotations,
    providing links to gnomAD, COSMIC, ClinVar, and CIViC
- name: PostgreSQL
  role: Database back-end storing saved queries and previous annotation results so
    they can be periodically re-sent to the REST service
- name: Psycopg2
  role: Library connecting the Django web framework to the PostgreSQL back-end for
    retrieving saved queries and prior annotations
- name: Docker
  role: Platform used for easy installation and deployment of the querying service
    (Makefile bootstrap)
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1002/humu.24495
    title: 'VariantAlert: A web‐based tool to notify updates in genetic variant annotations'
  source_content_hash: sha256:5c07f6f43f92cdcb9895ea2cdbbaf501c92cbf20d77a8b346fbdd10d475e6803
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/next-crs4/VariantAlert
  repo_url: https://github.com/next-crs4/VariantAlert
  tools:
  - DisGeNET
  - Django
  - Docker
  - MasterMind
  - MyVariant.info
  - PostgreSQL
  - Psycopg2
  - Python
  - VariantAlert
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3275
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# rest-api-variant-annotation-querying

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Query structured, continuously updated variant annotations from external REST resources such as MyVariant.info (which links to gnomAD, COSMIC, ClinVar, CIViC) using genomic HGVS identifiers, forming the annotation backbone for daily variant re-annotation and change detection in VariantAlert.

## When to use

Use when you need to fetch current structured annotations (allele frequencies, ClinVar CLNSIG classifications, COSMIC presence) for one or more variants from a REST API, or when building a re-annotation loop that must repeatedly re-query saved variant queries.

## When NOT to use

- When privacy policies prevent uploading genetic data to public platforms — the article suggests a local deployment of VariantAlert instead of relying on the hosted service
- When annotations must be produced offline or the source requires manual file re-upload (e.g., iVar-style workflows) rather than automated API re-querying
- When raw sequence data re-analysis is needed (reference genome or variant-calling updates) — this skill only covers knowledge-base annotation updates, not pipeline re-execution

## Inputs

- Variant identifiers in genomic HGVS format (e.g., chr1:g.35367G>A), single or in CSV/VCF batch files with one variant per row
- MyVariant.info REST endpoint URL (e.g., https://myvariant.info/v1/variant/chr1:g.35367G%3EA)
- Prior stored annotations (variant id, previous query results) from PostgreSQL for re-annotation loops

## Outputs

- Structured JSON annotation records per variant (fields such as gnomAD allele frequency, ClinVar CLNSIG, COSMIC presence, transcript name)
- Saved query records persisted in a PostgreSQL back-end for periodic re-querying
- Diffable annotation snapshots suitable for change detection (added, removed, changed categories)

## How to apply

Construct MyVariant.info REST query URLs from variant identifiers in genomic HGVS format (e.g., chr1:g.35367G>A, URL-encoded as chr1:g.35367G%3EA) and fetch responses as JSON, e.g. via the MyVariant.py wrapper. Variants can be submitted as a single identifier or in batches as CSV or VCF files with one variant per row; from a persistent service, store the queries in a PostgreSQL back-end (accessed via Psycopg2) so they can be periodically re-sent to keep annotations current. Map each returned JSON annotation record to downstream change-detection rules: report any difference for fields stored as strings (e.g., transcript name), flag allele-frequency variations only when they differ by at least 1 order of magnitude (3.830e-06 to 3.979e-06 is not notified), and treat newly appearing or removed annotation fields (e.g., COSMIC presence) as added/removed changes. When extending to additional sources such as DisGeNET or Genomenon's MasterMind, implement a module that forwards requests and manages REST responses through the same interface VariantAlert uses for MyVariant.info, returning structured JSON records parseable by the same change-detection rules.

## Related tools

- **VariantAlert** (Django/Python web application whose client methods forward user requests to MyVariant.info, manage REST responses, and periodically re-send all saved queries for daily re-annotation) — https://github.com/next-crs4/VariantAlert
- **MyVariant.info** (External REST resource that aggregates and serves structured variant annotations, providing links to gnomAD, COSMIC, ClinVar, and CIViC)
- **PostgreSQL** (Database back-end storing saved queries and previous annotation results so they can be periodically re-sent to the REST service)
- **Psycopg2** (Library connecting the Django web framework to the PostgreSQL back-end for retrieving saved queries and prior annotations)
- **Docker** (Platform used for easy installation and deployment of the querying service (Makefile bootstrap))

## Examples

```
import myvariant; mv = myvariant.MyVariantInfo(); ann = mv.getvariant('chr1:g.35367G>A', as_=True)
```

## Evaluation signals

- Each queried variant returns a JSON annotation record whose fields are parseable by change-detection rules: any difference reported for string fields, allele-frequency changes flagged only at >= 1 order of magnitude
- A pair of allele frequencies differing by less than one order of magnitude (3.830e-06 vs 3.979e-06) produces no alert, while a ClinVar string-field (CLNSIG) difference is reported
- Saved queries persisted in PostgreSQL can be re-sent verbatim to MyVariant.info and responses compared against the stored previous results
- Batch submissions (CSV/VCF, one variant per row) yield one structured annotation record per variant row
- Extension modules (e.g., DisGeNET) return structured JSON records matching the MyVariant.info annotation convention for every test variant

## Limitations

- New annotation sources such as Genomenon's MasterMind and DisGeNET are planned but not yet integrated in the original tool
- Allele-frequency changes below 1 order of magnitude are deliberately not reported, so subtle frequency shifts are silent
- A local deployment is the suggested procedure when privacy policies could prevent upload of genetic data to public platforms
- Stability of update cadence varies by source: databases like ClinVar are updated at each release while more stable sources such as gnomAD are upgraded less frequently

## Evidence
