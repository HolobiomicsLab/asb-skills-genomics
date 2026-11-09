---
name: cosmic-presence-detection
description: Use when comparing two annotation snapshots of the same variant (e.g.,
  MyVariant.info responses or archived ClinVar May 2021 vs May 2022 releases) to flag
  variants that gained a COSMIC annotation between the two time points.
when_to_use_negative:
- The target change is in a string field (transcript name, ClinVar CLNSIG) where exact-difference
  comparison — not presence/absence — is the correct rule
- The target change is a gnomAD allele frequency variation, which requires the 1-order-of-magnitude
  threshold rule rather than a binary presence check
- Only one annotation snapshot is available (a presence change cannot be assessed
  without the older snapshot)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: VariantAlert
  role: Web application (Django/Python, PostgreSQL backend) that daily re-annotates
    submitted variants via MyVariant.info, runs the change-detection rules including
    COSMIC presence detection, and emails the user on detected changes
  repo: https://github.com/next-crs4/VariantAlert
- name: MyVariant.info
  role: External REST resource queried daily to obtain updated variant annotations,
    including COSMIC presence, that are compared against previously stored snapshots
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/humu.24495
    title: 'VariantAlert: A web‐based tool to notify updates in genetic variant annotations'
  source_content_hash: sha256:a203619573d112ffeb0a78ac0e505b0f93a70cf44efed0c2975c8e297784bc23
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
  - Python
  - VariantAlert
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cosmic-presence-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects newly reported presence of a variant in the COSMIC catalogue of somatic mutations in cancer by checking whether the COSMIC annotation field is absent in the older annotation snapshot and present in the newer one, as part of VariantAlert's daily re-annotation change detection.

## When to use

Use when comparing two annotation snapshots of the same variant (e.g., MyVariant.info responses or archived ClinVar May 2021 vs May 2022 releases) to flag variants that gained a COSMIC annotation between the two time points.

## When NOT to use

- The target change is in a string field (transcript name, ClinVar CLNSIG) where exact-difference comparison — not presence/absence — is the correct rule
- The target change is a gnomAD allele frequency variation, which requires the 1-order-of-magnitude threshold rule rather than a binary presence check
- Only one annotation snapshot is available (a presence change cannot be assessed without the older snapshot)

## Inputs

- Variant identifier list (single variant or batch CSV/VCF files, one variant per row)
- Older annotation snapshot per variant (MyVariant.info REST response or archived ClinVar May 2021 release record)
- Newer annotation snapshot per variant (MyVariant.info REST response or archived ClinVar May 2022 release record)

## Outputs

- Per-variant structured JSON change report listing detected changes by category (added, removed, changed) with old and new values
- Email notification to the submitting user when a COSMIC presence change (or other annotation change) is detected

## How to apply

Load the variant identifier and its two annotation snapshots (MyVariant.info REST responses or the archived ClinVar May 2021 and May 2022 releases) using Python. For each variant, check the COSMIC annotation field: emit an 'added' change event only when the field is absent (or empty) in the older snapshot and present in the newer one — this is a binary presence check, unlike string fields (e.g., transcript name, ClinVar CLNSIG) that use exact-difference comparison, or gnomAD allele frequencies that use the 1-order-of-magnitude rule. Include the change in a structured JSON change report per variant, categorized as added, removed, or changed with old and new values. A COSMIC presence addition triggers the VariantAlert email notification, alongside other detected changes such as a ClinVar classification change or a gnomAD allele-frequency shift of at least one order of magnitude (e.g., 3.830e-06 to 3.979e-06 must NOT be flagged).

## Related tools

- **VariantAlert** (Web application (Django/Python, PostgreSQL backend) that daily re-annotates submitted variants via MyVariant.info, runs the change-detection rules including COSMIC presence detection, and emails the user on detected changes) — https://github.com/next-crs4/VariantAlert
- **MyVariant.info** (External REST resource queried daily to obtain updated variant annotations, including COSMIC presence, that are compared against previously stored snapshots)

## Examples

```
cosmic_added = [v for v in variants if not old[v].get('cosmic') and new[v].get('cosmic')]  # report as 'added' change with old=None, new=COSMIC value
```

## Evaluation signals

- A variant whose COSMIC field is absent in the May 2021 snapshot and present in the May 2022 snapshot produces an 'added' change event in the JSON report
- A variant already present in COSMIC in both snapshots produces no COSMIC alert (no duplicate 'added' events)
- Validation pair: gnomAD allele frequencies 3.830e-06 vs 3.979e-06 must produce no allele-frequency alert, while a ClinVar string-field difference must be reported — confirming the COSMIC rule is applied alongside, not conflated with, other rules
- Every emitted change report entry carries the category (added/removed/changed) and old/new values for the COSMIC field
- A detected COSMIC presence addition results in an email notification and the updated annotations being stored on the web site

## Limitations

- The check is binary presence/absence only: a change in COSMIC content while the field remains present is not captured by this rule
- Change detection depends on the underlying resource's release cadence; annotation sources such as COSMIC and ClinVar update at different frequencies than more stable sources like gnomAD
- New annotation sources (e.g., MasterMind, DisGeNET) are planned but not yet integrated, so coverage is limited to currently supported resources
- Privacy policies may prevent uploading genetic data to the public web service; a local Docker deployment is suggested in that case

## Evidence
