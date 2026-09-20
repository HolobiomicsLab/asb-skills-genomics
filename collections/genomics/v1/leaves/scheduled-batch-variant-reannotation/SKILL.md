---
name: scheduled-batch-variant-reannotation
description: Use when submitted variant lists (single variants or batch CSV/VCF files)
  must be re-annotated on a recurring schedule and users notified only when annotations
  actually change.
when_to_use_negative:
- Privacy policies prevent uploading genetic data to public platforms and no local
  deployment of the tool is possible — a local deployment is the suggested procedure
  in this case
- A one-time static annotation is sufficient with no need to detect changes across
  resource updates (e.g., knowledge-base releases)
- The update requires re-executing the whole primary analysis pipeline from raw sequence
  data, which is often hampered by unavailability of the original data and is outside
  this annotation-only scope
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0219
tools:
- name: VariantAlert
  role: Orchestrates daily re-annotation of submitted variant lists, change detection,
    server-side storage, and email notification
  repo: https://github.com/next-crs4/VariantAlert
- name: MyVariant.info
  role: External REST annotation resource that is periodically re-queried for fresh
    variant annotations
- name: PostgreSQL
  role: Database back-end storing submitted variants, saved queries, and prior annotations
- name: Psycopg2
  role: Library connecting the Django web framework to the PostgreSQL back-end
- name: Docker
  role: Platform for easy installation and deployment of the tool (Makefile bootstrap)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/humu.24495
    title: 'VariantAlert: A web‐based tool to notify updates in genetic variant annotations'
  source_content_hash: sha256:9fc46e6e0b6edcaefb8d53cef81e2ff8792365bc3f44746335c2ddfa70b39aae
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# scheduled-batch-variant-reannotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for periodically re-annotating user-submitted variant lists (single variants or CSV/VCF batches) against external REST resources such as MyVariant.info on a daily cadence, notifying the user only when an annotation is added, removed, or changed so that reclassification-relevant updates are caught automatically.

## When to use

Use when submitted variant lists (single variants or batch CSV/VCF files) must be re-annotated on a recurring schedule and users notified only when annotations actually change.

## When NOT to use

- Privacy policies prevent uploading genetic data to public platforms and no local deployment of the tool is possible — a local deployment is the suggested procedure in this case
- A one-time static annotation is sufficient with no need to detect changes across resource updates (e.g., knowledge-base releases)
- The update requires re-executing the whole primary analysis pipeline from raw sequence data, which is often hampered by unavailability of the original data and is outside this annotation-only scope

## Inputs

- Variant list submitted as a single variant identifier
- Batch variant list as CSV file (one variant per row)
- Batch variant list as VCF file (one variant per row)
- Stored prior annotations and inter-query differences from the PostgreSQL back-end

## Outputs

- Freshly re-annotated variant records in JSON format from MyVariant.info
- Updated annotations stored server-side with differences highlighted
- Notification record per changed variant (in the tool: an email with a direct link to the affected variant)

## How to apply

Load the submitted variant list (one variant per row for CSV/VCF batches) and construct MyVariant.info REST query URLs (e.g., https://myvariant.info/v1/variant/chr1:g.35367G%3EA), fetching annotations as JSON via the MyVariant.py wrapper. Retrieve prior annotations and their stored differences from the PostgreSQL back-end through Psycopg2, then periodically re-send all saved queries to MyVariant.info to mimic the daily re-annotation cadence. Apply change-detection rules per field type: report any difference for string fields (e.g., transcript name), report allele-frequency variations of at least 1 order of magnitude (3.830e−06 → 3.979e−06 is NOT notified), and apply normal comparison to other values such as pathogenicity scores. For each variant with a detected change (added, removed, changed), store updated annotations server-side, highlight the differences, and emit a notification — in VariantAlert this is an email with a direct link to the affected variant, delivered by a Django web application backed by PostgreSQL and deployed via Docker with a Makefile bootstrap.

## Related tools

- **VariantAlert** (Orchestrates daily re-annotation of submitted variant lists, change detection, server-side storage, and email notification) — https://github.com/next-crs4/VariantAlert
- **MyVariant.info** (External REST annotation resource that is periodically re-queried for fresh variant annotations)
- **PostgreSQL** (Database back-end storing submitted variants, saved queries, and prior annotations)
- **Psycopg2** (Library connecting the Django web framework to the PostgreSQL back-end)
- **Docker** (Platform for easy installation and deployment of the tool (Makefile bootstrap))

## Examples

```
import myvariant; mv = myvariant.MyVariantInfo(); ann = mv.getvariant("chr1:g.35367G>A")  # REST fetch of the current annotation for change detection
```

## Evaluation signals

- Notification records exist only for variants with a change detected under the field-type rules (annotation added, removed, or changed)
- String-field changes (e.g., transcript name) are flagged, while allele-frequency shifts below 1 order of magnitude (e.g., 3.830e−06 → 3.979e−06) are not notified
- Allele-frequency changes of at least 1 order of magnitude do trigger a notification
- Every re-annotation cycle re-sends all queries saved in the database, matching the daily cadence
- Each changed variant's updated annotations are stored server-side with differences highlighted and a direct link to the affected variant in the notification

## Limitations

- Privacy policies may prevent uploading genetic data to public platforms; local deployment is the suggested procedure
- The 1-order-of-magnitude allele-frequency threshold suppresses notifications for smaller real AF changes
- Annotation sources update at different rates (e.g., ClinVar at each release, more stable sources like gnomAD less frequently), so change yield varies over time
- Additional annotation sources (e.g., Genomenon's MasterMind, DisGeNET) are planned but not yet integrated

## Evidence
