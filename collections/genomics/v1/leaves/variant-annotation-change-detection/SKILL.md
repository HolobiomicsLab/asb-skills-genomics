---
name: variant-annotation-change-detection
description: Use when variant annotations from a knowledge base (ClinVar, gnomAD,
  COSMIC, CIViC) may have been updated since last annotation, and you must identify
  which fields changed and whether reclassification is warranted.
when_to_use_negative:
- Input is raw sequence data or requires re-executing alignment/variant calling —
  VariantAlert targets knowledge-base annotation updates, not pipeline re-runs, which
  are 'often hampered by the unavailability of the raw original sequence data'.
- The annotation source requires manual file re-upload and external annotation (like
  iVar), so no automated REST re-query loop can be applied.
- Allele-frequency differences are below one order of magnitude — by design, e.g.,
  3.830e−06 to 3.979e−06, these must not be reported as changes.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0091
tools:
- name: VariantAlert
  role: Web application (Django/Python) that implements the daily re-annotation loop
    and change-detection/notification logic
  repo: https://github.com/next-crs4/VariantAlert
- name: MyVariant.info
  role: REST API re-queried daily to fetch fresh structured variant annotations used
    for the current snapshot
- name: PostgreSQL
  role: Database back-end storing saved queries, previous annotations, and detected
    differences
- name: Psycopg2
  role: Python library connecting the Django framework to PostgreSQL for retrieving
    prior annotations
- name: Docker
  role: Deployment platform for installing VariantAlert locally (bootstrapped via
    Makefile)
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/humu.24495
    title: 'VariantAlert: A web‐based tool to notify updates in genetic variant annotations'
  source_content_hash: sha256:feabcc117721a422e9de5f99cdcb4cfe2eb1215bfb64ac23f27c51b2937e4ef3
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
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 26
---

# variant-annotation-change-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects differences (added, removed, changed) between previous and current annotations of submitted variants by daily re-querying MyVariant.info, mimicking VariantAlert's change-detection rules and email-notification loop.

## When to use

Use when variant annotations from a knowledge base (ClinVar, gnomAD, COSMIC, CIViC) may have been updated since last annotation, and you must identify which fields changed and whether reclassification is warranted.

## When NOT to use

- Input is raw sequence data or requires re-executing alignment/variant calling — VariantAlert targets knowledge-base annotation updates, not pipeline re-runs, which are 'often hampered by the unavailability of the raw original sequence data'.
- The annotation source requires manual file re-upload and external annotation (like iVar), so no automated REST re-query loop can be applied.
- Allele-frequency differences are below one order of magnitude — by design, e.g., 3.830e−06 to 3.979e−06, these must not be reported as changes.

## Inputs

- Submitted variant lists (single variant identifiers or batch CSV/VCF files, one variant per row)
- Previous annotation snapshot (e.g., archived ClinVar May 2021 and May 2022 releases or stored prior MyVariant.info query results)
- Fresh MyVariant.info REST JSON annotation responses

## Outputs

- Per-variant structured JSON change report categorizing changes as added, removed, or changed, with old and new values
- Notification records (e.g., variant_notifications.) with a link to the affected variant, emulating the email alert
- Updated annotations stored in a PostgreSQL back-end for the next re-annotation cycle

## How to apply

Construct MyVariant.info REST queries (e.g., https://myvariant.info/v1/variant/chr1:g.35367G%3EA) for each variant — single variants or batch CSV/VCF rows — and fetch annotations as JSON. Compare each field between the stored previous snapshot and the fresh response using field-type-specific rules: fields stored as strings (e.g., transcript name, ClinVar CLNSIG classification) trigger an alert on any exact difference; allele frequencies trigger an alert only if they vary by at least 1 order of magnitude (so 3.830e−06 → 3.979e−06 is NOT notified); presence in COSMIC is detected when the annotation is absent in the older snapshot and present in the newer one. Emit a structured change report per variant listing changes by category (added, removed, changed) with old and new values, store the updated annotations server-side (VariantAlert uses PostgreSQL via Psycopg2), and generate a notification record with a link to the affected variant. Re-run periodically (VariantAlert re-sends all saved queries daily) to maintain the annotation-update cadence.

## Related tools

- **VariantAlert** (Web application (Django/Python) that implements the daily re-annotation loop and change-detection/notification logic) — https://github.com/next-crs4/VariantAlert
- **MyVariant.info** (REST API re-queried daily to fetch fresh structured variant annotations used for the current snapshot)
- **PostgreSQL** (Database back-end storing saved queries, previous annotations, and detected differences)
- **Psycopg2** (Python library connecting the Django framework to PostgreSQL for retrieving prior annotations)
- **Docker** (Deployment platform for installing VariantAlert locally (bootstrapped via Makefile))

## Examples

```
import requests; ann = requests.get('https://myvariant.info/v1/variant/chr1:g.35367G%3EA').(); changes = [f for f in ann if ann[f] != prev[f]]
```

## Evaluation signals

- A string-field change (e.g., ClinVar CLNSIG classification or transcript name) is flagged, while an allele-frequency shift from 3.830e−06 to 3.979e−06 produces NO alert (sub-order-of-magnitude rule).
- variant_notifications. exists, parses as valid JSON, and contains records only for variants with a change detected under the field-type rules.
- New COSMIC presence is detected exactly when the COSMIC field is absent in the older snapshot and present in the newer one.
- Each change report lists old and new values categorized as added, removed, or changed.
- Stored prior annotations retrieved from PostgreSQL match the snapshots compared, and updated annotations are persisted after each detected change.

## Limitations

- Only changes in annotation accessible via REST (MyVariant.info resources such as ClinVar, gnomAD, COSMIC, CIViC) are detected; new sources like MasterMind or DisGeNET are planned but not yet integrated.
- Privacy policies may prevent uploading genetic data to public platforms, so local deployment is the suggested procedure for sensitive data.
- Databases update at different cadences: ClinVar is updated at each release, while more stable annotation sources such as gnomAD are upgraded less frequently, affecting alert frequency.

## Evidence
