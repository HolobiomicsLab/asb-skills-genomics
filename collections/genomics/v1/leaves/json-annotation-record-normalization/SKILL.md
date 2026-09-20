---
name: json-annotation-record-normalization
description: Use when integrating a new REST annotation source into an automated re-annotation
  pipeline and its responses must be mapped into a canonical structured JSON record
  schema for change detection.
when_to_use_negative:
- The annotation source is not accessible via a REST application programming interface
  (e.g., iVar-style workflows where annotations must be performed externally and files
  manually re-uploaded)
- Privacy policies prevent uploading genetic data to public platforms and no local
  deployment of the annotation service is possible
- The goal is raw data reprocessing (sequence alignment/variant calling) rather than
  annotation lookup — VariantAlert only re-annotates submitted variants
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: VariantAlert
  role: Target application whose JSON annotation convention and change-detection rules
    the normalized records must conform to; source of the client interface pattern
    to replicate
  repo: https://github.com/next-crs4/VariantAlert
- name: MyVariant.info
  role: Reference REST annotation service defining the JSON annotation record convention
    that new annotation-source modules must match
- name: DisGeNET
  role: New REST annotation source whose gene-disease annotations are mapped to the
    normalized JSON record schema
- name: Django
  role: Python web framework in which the annotation-source adapter module and client
    methods are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/humu.24495
    title: 'VariantAlert: A web‐based tool to notify updates in genetic variant annotations'
  source_content_hash: sha256:fffbb8eac2b40873b3772feb311a649cc7abbcae14f2f1e9bcfa060a2f6a9e63
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
  - MasterMind
  - MyVariant.info
  - Python
  - VariantAlert
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5909090909090909
n_claims: 22
---

# -annotation-record-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalizes annotation records returned by external REST annotation services (e.g., MyVariant.info, DisGeNET) into the structured JSON convention used by VariantAlert's change-detection engine, so that annotations from new pluggable sources can be compared and diffed consistently.

## When to use

Use when integrating a new REST annotation source into an automated re-annotation pipeline and its responses must be mapped into a canonical structured JSON record schema for change detection.

## When NOT to use

- The annotation source is not accessible via a REST application programming interface (e.g., iVar-style workflows where annotations must be performed externally and files manually re-uploaded)
- Privacy policies prevent uploading genetic data to public platforms and no local deployment of the annotation service is possible
- The goal is raw data reprocessing (sequence alignment/variant calling) rather than annotation lookup — VariantAlert only re-annotates submitted variants

## Inputs

- Variant lists with variants in genomic HGVS format (e.g., chr1:g.35367G>A)
- JSON responses from external REST annotation services (MyVariant.info, DisGeNET, MasterMind)

## Outputs

- Structured JSON annotation records matching the MyVariant.info annotation convention
- Normalized annotation records parseable by VariantAlert's change-detection rules

## How to apply

Inspect the existing client methods (in VariantAlert's Django/Python codebase) that forward user requests to MyVariant.info and manage REST responses, then implement the new annotation-source module exposing the same interface contract. Map variant identifiers in genomic HGVS format (e.g., chr1:g.35367G>A) to the target service's REST API queries (e.g., DisGeNET gene-disease annotations). Store the returned annotations as structured JSON records matching the MyVariant.info JSON annotation convention so downstream comparison rules apply. Rationale: VariantAlert periodically re-sends all saved queries to keep annotations updated, so records must be schema-consistent across sources for the diff logic to fire correctly.

## Related tools

- **VariantAlert** (Target application whose JSON annotation convention and change-detection rules the normalized records must conform to; source of the client interface pattern to replicate) — https://github.com/next-crs4/VariantAlert
- **MyVariant.info** (Reference REST annotation service defining the JSON annotation record convention that new annotation-source modules must match)
- **DisGeNET** (New REST annotation source whose gene-disease annotations are mapped to the normalized JSON record schema)
- **Django** (Python web framework in which the annotation-source adapter module and client methods are implemented)

## Examples

```
git clone https://github.com/next-crs4/VariantAlert.git
```

## Evaluation signals

- The module returns structured annotation records for every test variant in a small validation set
- All returned record fields are parseable by VariantAlert's change-detection rules
- String-field differences (e.g., transcript name) are reported for any change, matching the exact-difference comparison convention
- Allele-frequency changes are flagged only at >= 1 order of magnitude (e.g., 3.830e−06 to 3.979e−06 is not notified), consistent with the tool's threshold
- Normalized records are stored as JSON matching the MyVariant.info annotation convention and diffable against prior annotations on the website

## Limitations

- Only annotation sources accessible via REST API can be integrated; MasterMind and DisGeNET were planned but not yet integrated at time of publication
- Differences are only meaningful if the new source's fields can be mapped to the JSON annotation convention; string fields use exact-difference comparison and allele frequencies use a 1-order-of-magnitude threshold
- No changelog is available for the software, complicating version tracking of adapter changes

## Evidence
