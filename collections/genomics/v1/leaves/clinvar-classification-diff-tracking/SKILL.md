---
name: clinvar-classification-diff-tracking
description: Use when a variant list must be periodically re-annotated and changes
  such as a ClinVar classification update, a ≥1-order-of-magnitude gnomAD allele-frequency
  shift, or newly reported COSMIC presence must be flagged.
when_to_use_negative:
- Allele frequencies differing by less than one order of magnitude (e.g., 3.830e-06
  vs 3.979e-06) — by design this must not produce an alert
- Re-execution of the sequence analysis pipeline (alignment/variant calling) or reference
  genome updates — this skill only tracks annotation/knowledge-base updates
- Untrusted data destined for a public platform where privacy policies prevent uploading
  genetic data — prefer a local deployment of the tool instead
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: VariantAlert
  role: Django/Python web application that daily re-annotates submitted variants via
    MyVariant.info, compares results to identify added/removed/changed annotations,
    and emails users on detection
  repo: https://github.com/next-crs4/VariantAlert
- name: MyVariant.info
  role: REST API source of continuously updated variant annotations (ClinVar CLNSIG,
    gnomAD AF, COSMIC, CIViC) re-queried daily for each saved query
- name: Docker
  role: Deployment platform used to install the tool locally (Makefile bootstrap),
    enabling privacy-preserving in-house diff tracking
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/humu.24495
    title: 'VariantAlert: A web‐based tool to notify updates in genetic variant annotations'
  source_content_hash: sha256:1a4ed740c6ab2f94a54ca1e11cafaced0fbf38f5c0d4c9a0b538cdcfb1094dec
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
  - Python
  - VariantAlert
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# clinvar-classification-diff-tracking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects changes in a variant's annotations (ClinVar CLNSIG classification, gnomAD allele frequency, COSMIC presence) between an older and a newer annotation snapshot and emits an email/JSON change report per variant, as implemented by VariantAlert.

## When to use

Use when a variant list must be periodically re-annotated and changes such as a ClinVar classification update, a ≥1-order-of-magnitude gnomAD allele-frequency shift, or newly reported COSMIC presence must be flagged.

## When NOT to use

- Allele frequencies differing by less than one order of magnitude (e.g., 3.830e-06 vs 3.979e-06) — by design this must not produce an alert
- Re-execution of the sequence analysis pipeline (alignment/variant calling) or reference genome updates — this skill only tracks annotation/knowledge-base updates
- Untrusted data destined for a public platform where privacy policies prevent uploading genetic data — prefer a local deployment of the tool instead

## Inputs

- Variant identifiers (hgvs/CHR-POS-REF-ALT) submitted singly or in batches as CSV or VCF files
- Current and previous annotation snapshots (MyVariant.info REST responses or archived ClinVar May 2021 / May 2022 releases)

## Outputs

- Structured JSON change report per variant, listing changes by category (added, removed, changed) with old and new values
- Email notification to the user when an annotation change is detected
- Updated annotations stored and differences highlighted on the web site

## How to apply

Load the variant identifier and the two annotation snapshots (MyVariant.info REST responses or archived ClinVar May 2021 vs May 2022 releases) with Python. For string fields such as the ClinVar CLNSIG classification or transcript name, apply an exact-difference comparison: any difference is reported (e.g., 'Likely pathogenic' → 'Pathogenic'). For gnomAD allele frequencies, apply the order-of-magnitude rule — report only if the frequency varies by at least 1 order of magnitude, so 3.830e-06 to 3.979e-06 must NOT be flagged. Detect newly reported COSMIC presence when the COSMIC field is absent in the older snapshot and present in the newer one. Emit a structured JSON change report per variant categorizing changes as added, removed, or changed with old and new values, and notify the user by email; optionally pre-filter to rare variants (gnomAD AF < 0.01) with CLNSIG 'Pathogenic' or 'Likely pathogenic' per the May 2022 release.

## Related tools

- **VariantAlert** (Django/Python web application that daily re-annotates submitted variants via MyVariant.info, compares results to identify added/removed/changed annotations, and emails users on detection) — https://github.com/next-crs4/VariantAlert
- **MyVariant.info** (REST API source of continuously updated variant annotations (ClinVar CLNSIG, gnomAD AF, COSMIC, CIViC) re-queried daily for each saved query)
- **Docker** (Deployment platform used to install the tool locally (Makefile bootstrap), enabling privacy-preserving in-house diff tracking)

## Examples

```
git clone https://github.com/next-crs4/VariantAlert.git
```

## Evaluation signals

- A pair of gnomAD allele frequencies differing by less than one order of magnitude (3.830e-06 vs 3.979e-06) produces no allele-frequency alert
- A difference in a string field (ClinVar CLNSIG, transcript name) between snapshots is always reported
- The JSON change report lists changes by category (added, removed, changed) with both old and new values for each variant
- COSMIC 'newly reported' is flagged only when the COSMIC field is absent in the older snapshot and present in the newer one
- On real data, the fraction of families with at least one annotation change between the 2021 and 2022 ClinVar releases should be on the order of the reported 41%

## Limitations

- String-field comparison reports any difference, so minor ClinVar classification shifts (e.g., 'Likely pathogenic' → 'Pathogenic') are flagged separately from new Pathogenic variants; 73 families (6%) showed only such minor changes
- No changelog is available for the software
- Privacy policies may prevent uploading genetic data to the hosted web service; local deployment is the suggested procedure
- Additional annotation sources (e.g., Genomenon's MasterMind, DisGeNET) are planned but not yet integrated

## Evidence

- [methods] extracting the variants classified as "Pathogenic" or "Likely pathogenic" (CLNSIG annotation of ClinVar) according to the May 2022 release: [evidence span withheld — anchor status: not_found]
