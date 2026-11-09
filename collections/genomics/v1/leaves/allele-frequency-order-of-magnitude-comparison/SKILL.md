---
name: allele-frequency-order-of-magnitude-comparison
description: Use when comparing population allele frequencies (e.g., gnomAD AF) between
  the previous and current annotation snapshots of a submitted variant, to decide
  whether the shift is large enough to notify the user or should be silently ignored.
when_to_use_negative:
- The two allele-frequency values differ by less than 1 order of magnitude (e.g.,
  3.830e-06 to 3.979e-06) — the rule explicitly suppresses such sub-threshold shifts
- Comparing string-valued annotation fields (e.g., ClinVar CLNSIG classification,
  transcript name) — use exact-difference comparison instead, reporting any difference
- Detecting first-time presence of an annotation (e.g., new COSMIC entry) — check
  absent-in-old/present-in-new rather than magnitude change
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: VariantAlert
  role: Web application (Django/Python) implementing the daily re-annotation loop
    and the order-of-magnitude change-detection rule over stored variant queries
  repo: https://github.com/next-crs4/VariantAlert
- name: MyVariant.info
  role: External REST annotation resource periodically re-queried to obtain fresh
    gnomAD allele frequencies for comparison against stored prior annotations
- name: Python
  role: Language used to load variant identifiers and snapshots, fetch MyVariant.info
    responses, and apply the comparison rules per field type
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/humu.24495
    title: 'VariantAlert: A web‐based tool to notify updates in genetic variant annotations'
  source_content_hash: sha256:41f29bfb1dbd0c3bf5417729f0ac8cbbc8cf7d40a9a0f076f973d2b90bc3b042
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
  - MyVariant.info
  - PostgreSQL
  - Psycopg2
  - Python
  - VariantAlert
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# allele-frequency-order-of-magnitude-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

VariantAlert's field-type-aware change-detection rule for allele frequencies: a difference in gnomAD allele frequency between the previous and current annotation snapshots is reported only if it spans at least 1 order of magnitude, filtering out negligible drift that should not trigger user email notifications.

## When to use

Use when comparing population allele frequencies (e.g., gnomAD AF) between the previous and current annotation snapshots of a submitted variant, to decide whether the shift is large enough to notify the user or should be silently ignored.

## When NOT to use

- The two allele-frequency values differ by less than 1 order of magnitude (e.g., 3.830e-06 to 3.979e-06) — the rule explicitly suppresses such sub-threshold shifts
- Comparing string-valued annotation fields (e.g., ClinVar CLNSIG classification, transcript name) — use exact-difference comparison instead, reporting any difference
- Detecting first-time presence of an annotation (e.g., new COSMIC entry) — check absent-in-old/present-in-new rather than magnitude change

## Inputs

- Two annotation snapshots per variant (previous and current JSON responses from MyVariant.info REST queries, e.g. https://myvariant.info/v1/variant/chr1:g.35367G%3EA)
- Submitted variant list (single variant or batch CSV/VCF file, one variant per row)
- Archived annotation releases (e.g., ClinVar May 2021 and May 2022) for offline reconstruction

## Outputs

- Per-variant structured change report listing detected changes by category (added, removed, changed) with old and new values
- Notification records (e.g., variant_notifications.) for variants with a detected change, emulating the daily email alert

## How to apply

Load the two MyVariant.info annotation snapshots (current and previous) for each variant identifier (e.g., chr1:g.35367G>A). For allele-frequency fields, convert both values to logs and flag the change only when the ratio spans at least 1 order of magnitude — i.e., |log10(new/old)| >= 1; the documented example pair 3.830e-06 vs 3.979e-06 must NOT be notified. For fields stored as strings (e.g., the ClinVar CLNSIG classification or transcript name), apply exact-difference comparison and report any difference. Detect newly added annotations (e.g., first presence in COSMIC) when the field is absent in the older snapshot and present in the newer one, and apply normal comparison to other values such as pathogenicity scores. Emit a structured change report (added/removed/changed, with old and new values) only for variants with a detected change, which drives the daily email notification loop backed by PostgreSQL/Psycopg2.

## Related tools

- **VariantAlert** (Web application (Django/Python) implementing the daily re-annotation loop and the order-of-magnitude change-detection rule over stored variant queries) — https://github.com/next-crs4/VariantAlert
- **MyVariant.info** (External REST annotation resource periodically re-queried to obtain fresh gnomAD allele frequencies for comparison against stored prior annotations)
- **Python** (Language used to load variant identifiers and snapshots, fetch MyVariant.info responses, and apply the comparison rules per field type)

## Examples

```
python -c "import math; f1,f2=3.830e-06,3.979e-06; print('AF change' if abs(math.log10(f2/f1))>=1 else 'no alert')"
```

## Evaluation signals

- A pair of allele frequencies differing by less than one order of magnitude (3.830e-06 vs 3.979e-06) must produce no allele-frequency alert
- A change in a string field (e.g., ClinVar CLNSIG classification) must always be reported, regardless of magnitude heuristics
- A frequency ratio of >= 10x (|log10(new/old)| >= 1) must be flagged as a change
- variant_notifications. must parse as valid JSON and contain records only for variants with a change detected under the field-type rules
- A newly present annotation field (absent in the older snapshot, present in the newer) must be reported under the 'added' category

## Limitations

- The 1-order-of-magnitude threshold is a fixed heuristic; smaller but potentially meaningful frequency shifts are deliberately not notified
- Only annotation changes from resources accessed via MyVariant.info (gnomAD, COSMIC, ClinVar, CIViC, etc.) are detected; planned sources such as Genomenon's MasterMind and DisGeNET are not yet integrated
- Privacy policies may prevent uploading genetic data to the public web service; local deployment via Docker is the suggested procedure in those cases

## Evidence

- [methods] Order-of-magnitude threshold on allele frequencies: "for allele frequencies, variations of at least 1 order of magnitude are reported (e.g., a change from 3.830e−06 to 3.979e−06 would not be notified)"
- [methods] String-field exact comparison: "for fields stored as string (e.g., the transcript name) any difference is reported"
- [abstract] Daily re-annotation and email notification on change: "the user is notified by email and updated annotations are stored on the web‐site"
- [methods] Periodic re-query of saved queries: "VariantAlert periodically re‐send to the MyVariant.info service all the queries saved in the database, to keep the annotations updated"
