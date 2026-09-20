---
name: genomic-variant-identifier-mapping
description: Use when variant lists (single variants or CSV/VCF batches) must be converted
  into REST API queries against annotation services like MyVariant.info, or when extending
  the pipeline with a new API-based annotation source such as DisGeNET.
when_to_use_negative:
- The workflow requires annotations produced by an external tool and manually re-uploaded
  files (like iVar), rather than automated API-based re-annotation
- Privacy policies prevent uploading genetic data to public platforms and no local
  deployment of the software has been set up
- The input is not a variant list but raw sequence data requiring full re-execution
  of the alignment/variant-calling pipeline
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: VariantAlert
  role: Django/Python web application whose client methods forward user requests to
    REST annotation services and manage responses; the framework into which new annotation-source
    modules plug
  repo: https://github.com/next-crs4/VariantAlert
- name: MyVariant.info
  role: External REST API resource that aggregates structured variant annotations
    (with links to gnomAD, COSMIC, ClinVar, CIViC); the reference integration pattern
- name: DisGeNET
  role: Planned new annotation source integrated via a pluggable module following
    the same REST API interface contract, queried with genomic HGVS identifiers for
    gene-disease annotations
- name: MasterMind
  role: Genomenon's REST-accessible annotation source for which a parallel adapter
    can be implemented against the same interface contract
- name: Django
  role: Python-based web framework in which the annotation-source modules and REST
    client methods are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/humu.24495
    title: 'VariantAlert: A web‐based tool to notify updates in genetic variant annotations'
  source_content_hash: sha256:41839ca78fe1d1d5d693a508b937aea8b739c7c0a2acd8b6b180ad2efc41e963
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# genomic-variant-identifier-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps user-submitted variant identifiers (e.g., genomic HGVS like chr1:g.35367G>A, or CSV/VCF batch rows) into queries forwarded to external REST annotation services such as MyVariant.info, following VariantAlert's API-based integration pattern that new annotation sources (DisGeNET, MasterMind) can reuse.

## When to use

Use when variant lists (single variants or CSV/VCF batches) must be converted into REST API queries against annotation services like MyVariant.info, or when extending the pipeline with a new API-based annotation source such as DisGeNET.

## When NOT to use

- The workflow requires annotations produced by an external tool and manually re-uploaded files (like iVar), rather than automated API-based re-annotation
- Privacy policies prevent uploading genetic data to public platforms and no local deployment of the software has been set up
- The input is not a variant list but raw sequence data requiring full re-execution of the alignment/variant-calling pipeline

## Inputs

- Single variant identifiers in genomic HGVS format (e.g., chr1:g.35367G>A)
- Batch variant lists as CSV or VCF files, one variant per row
- Saved queries retrieved from the VariantAlert database (PostgreSQL back-end)

## Outputs

- Structured JSON annotation records matching the MyVariant.info JSON annotation convention
- REST API query requests forwarded to annotation services (MyVariant.info, DisGeNET, MasterMind)

## How to apply

Inspect the VariantAlert (Django/Python) codebase to locate the client methods that forward user requests—coming from the user interface or retrieved from the PostgreSQL database—to MyVariant.info and manage the REST web service responses. Express each input variant in genomic HGVS format (e.g., chr1:g.35367G>A) as accepted by the service, whether submitted as a single variant or in batches via CSV or VCF files where each row represents a different variant. To add a source like DisGeNET, implement a pluggable annotation-source module exposing the same interface contract, mapping HGVS identifiers to the new REST API's gene-disease annotation queries, and optionally a parallel adapter for Genomenon's MasterMind. Store returned annotations as structured JSON records matching the MyVariant.info JSON convention so VariantAlert's change-detection rules can compare them; periodically re-send all saved queries to keep annotations current.

## Related tools

- **VariantAlert** (Django/Python web application whose client methods forward user requests to REST annotation services and manage responses; the framework into which new annotation-source modules plug) — https://github.com/next-crs4/VariantAlert
- **MyVariant.info** (External REST API resource that aggregates structured variant annotations (with links to gnomAD, COSMIC, ClinVar, CIViC); the reference integration pattern)
- **DisGeNET** (Planned new annotation source integrated via a pluggable module following the same REST API interface contract, queried with genomic HGVS identifiers for gene-disease annotations)
- **MasterMind** (Genomenon's REST-accessible annotation source for which a parallel adapter can be implemented against the same interface contract)
- **Django** (Python-based web framework in which the annotation-source modules and REST client methods are implemented)

## Examples

```
git clone https://github.com/next-crs4/VariantAlert.git
```

## Evaluation signals

- The module returns a structured annotation record (JSON) for every test variant
- Returned fields are parseable by VariantAlert's change-detection rules: any difference is reported for string fields (e.g., transcript name)
- Allele-frequency changes are flagged only when they span at least 1 order of magnitude (e.g., 3.830e−06 to 3.979e−06 is not notified)
- VariantAlert periodically re-sends all saved queries and correctly notifies the user by email when a change (added, removed, changed annotation) is detected

## Limitations

- New annotation sources (MasterMind, DisGeNET) were planned but not yet integrated at publication time
- Privacy policies may prevent uploading genetic data to public platforms, so a local deployment of VariantAlert is the suggested procedure
- No changelog is available for the VariantAlert software
- Re-annotation depends on external REST services remaining available; re-execution of whole pipelines is often hampered by unavailability of raw original sequence data

## Evidence
