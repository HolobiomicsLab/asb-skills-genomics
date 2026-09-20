---
name: annotation-difference-tracking
description: Use when a stored list of variants (single variants or CSV/VCF batch
  files) must be re-annotated daily from external REST resources and changes — e.g.
  ClinVar classification, gnomAD allele frequency, presence in COSMIC/CIViC — must
  be detected and notified.
when_to_use_negative:
- The goal is re-execution of the whole pipeline from raw sequence data — this requires
  the original data and intensive compute; VariantAlert only re-queries annotation
  APIs on already-called variants.
- You need annotation sources not yet integrated, such as Genomenon's MasterMind or
  DisGeNET — these are planned but not implemented.
- A one-shot annotation is sufficient and no ongoing change monitoring or notification
  is required.
edam_operation: http://edamontology.org/operation_0331
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3168
tools:
- name: VariantAlert
  role: Web application (Django/Python, PostgreSQL via Psycopg2) that performs daily
    re-annotation of submitted variants and notifies users by email when an annotation
    change is detected
  repo: https://github.com/next-crs4/VariantAlert
- name: MyVariant.info
  role: External REST resource re-queried daily to obtain updated variant annotations
    (linking in turn to gnomAD, COSMIC, ClinVar, CIViC)
- name: PostgreSQL
  role: Database back-end storing variant lists, saved queries, and previous annotation
    results used for comparison
- name: Psycopg2
  role: Library connecting the Django web framework to the PostgreSQL back-end for
    storing/retrieving prior annotations
- name: Docker
  role: Platform for easy local installation and deployment of VariantAlert via a
    Makefile bootstrap
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/humu.24495
    title: 'VariantAlert: A web‐based tool to notify updates in genetic variant annotations'
  source_content_hash: sha256:87a5a96a0e992680070167a301b7869835bdbba222e961b9310a33803d69ff03
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
  - Docker
  - MyVariant.info
  - PostgreSQL
  - Psycopg2
  - Python
  - VariantAlert
  edam_operation: http://edamontology.org/operation_0331
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 23
---

# annotation-difference-tracking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Periodic re-annotation of submitted variant lists via MyVariant.info REST queries, with field-type-aware change detection (strings, allele frequencies, pathogenicity scores) so that only meaningful annotation changes are reported and notified.

## When to use

Use when a stored list of variants (single variants or CSV/VCF batch files) must be re-annotated daily from external REST resources and changes — e.g. ClinVar classification, gnomAD allele frequency, presence in COSMIC/CIViC — must be detected and notified.

## When NOT to use

- The goal is re-execution of the whole pipeline from raw sequence data — this requires the original data and intensive compute; VariantAlert only re-queries annotation APIs on already-called variants.
- You need annotation sources not yet integrated, such as Genomenon's MasterMind or DisGeNET — these are planned but not implemented.
- A one-shot annotation is sufficient and no ongoing change monitoring or notification is required.

## Inputs

- Submitted variant lists: a single variant or batch CSV or VCF files, where each row represents a different variant
- Prior stored annotations (variant id, previous query results, differences between last two query results) from the PostgreSQL back-end
- Freshly re-fetched MyVariant.info annotation responses (JSON) for all saved queries

## Outputs

- Notification records (emulating the email notification) listing variants with detected annotation changes (added, removed, changed)
- Updated annotations stored server-side / on the web-site
- Highlighted differences between previous and current annotations, computed under the field-type rules

## How to apply

Load the submitted variant list (one variant per row, single or batch CSV/VCF) and build MyVariant.info REST query URLs, e.g. https://myvariant.info/v1/variant/chr1:g.35367G%3EA, fetching annotations as JSON (e.g. via the MyVariant.py wrapper). Retrieve stored prior annotations (variant id, previous query results, differences between the last two query results) from the PostgreSQL back-end via Psycopg2, then periodically re-send all saved queries to MyVariant.info to obtain fresh annotations, mimicking VariantAlert's daily cadence. Apply the per-field change rules: for fields stored as strings (e.g. the transcript name) report any difference; for allele frequencies report only variations of at least 1 order of magnitude (a change from 3.830e-06 to 3.979e-06 is NOT notified); apply normal comparison to other values such as pathogenicity scores. For each variant where an annotation is added, removed, or changed, write a notification record (emulating the email notification), store the updated annotations, and mark the highlighted differences. Deployable locally via Docker (Makefile bootstrap) when privacy policies prevent uploading genetic data to public platforms.

## Related tools

- **VariantAlert** (Web application (Django/Python, PostgreSQL via Psycopg2) that performs daily re-annotation of submitted variants and notifies users by email when an annotation change is detected) — https://github.com/next-crs4/VariantAlert
- **MyVariant.info** (External REST resource re-queried daily to obtain updated variant annotations (linking in turn to gnomAD, COSMIC, ClinVar, CIViC))
- **PostgreSQL** (Database back-end storing variant lists, saved queries, and previous annotation results used for comparison)
- **Psycopg2** (Library connecting the Django web framework to the PostgreSQL back-end for storing/retrieving prior annotations)
- **Docker** (Platform for easy local installation and deployment of VariantAlert via a Makefile bootstrap)

## Examples

```
curl https://myvariant.info/v1/variant/chr1:g.35367G%3EA
```

## Evaluation signals

- Notification records exist only for variants with a change detected under the field-type rules — no records for unchanged variants
- Changes in string fields (e.g. transcript names) are flagged
- Sub-order-of-magnitude allele-frequency shifts are correctly NOT notified (e.g. 3.830e-06 to 3.979e-06 produces no notification)
- Updated annotations are stored and differences are highlighted for each changed variant
- Notification output parses as valid JSON with records containing current and previous annotation values

## Limitations

- For allele frequencies, variations of less than 1 order of magnitude (e.g. 3.830e-06 to 3.979e-06) are silently not notified
- Only annotation sources accessible via REST API (currently MyVariant.info) are supported; new sources such as MasterMind and DisGeNET are planned but not integrated
- The hosted web service requires uploading genetic data — a local Docker deployment is the suggested procedure when privacy policies prevent this
- Competing tools like iVar require external annotation and manual file re-upload, limiting full automation

## Evidence

- [methods] Submitted variants are daily re‐annotated and if a change in an annotation is detected (added, removed, changed) due to the update of the underlying resource the user is notified by email: "Submitted variants are daily re‐annotated and if a change in an annotation is detected (added, removed, changed) due to the update of the underlying resource the user is notified by email"
- [methods] allele-frequency magnitude rule: "for allele frequencies, variations of at least 1 order of magnitude are reported (e.g., a change from 3.830e−06 to 3.979e−06 would not be notified)"
- [methods] string field rule: "for fields stored as string (e.g., the transcript name) any difference is reported"
- [readme] daily cadence and email notification: "the annotation is repeated daily and if a change is detected the user is notified by email."
- [methods] periodic re-query of saved queries: "VariantAlert periodically re‐send to the MyVariant.info service all the queries saved in the database, to keep the annotations updated"
