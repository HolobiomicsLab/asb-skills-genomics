---
name: annotation-source-interface-abstraction
description: Use when extending a variant re-annotation pipeline (e.g. VariantAlert,
  Django/Python) with a new external annotation database accessible via REST API,
  by implementing a module that mirrors the existing MyVariant.info client interface.
when_to_use_negative:
- The goal is to re-annotate variants using the already-integrated sources (gnomAD,
  COSMIC, ClinVar, CIViC) — use VariantAlert's existing daily re-annotation workflow
  instead of adding a new module.
- The new annotation source has no REST API (e.g. requires manual file upload or offline
  annotation, like iVar) — the interface-abstraction pattern depends on programmatic
  API access.
- Variant identifiers are not mappable to the target API's query scheme (e.g. non-HGVS
  identifiers that cannot be converted to genomic HGVS format).
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: VariantAlert
  role: Web application whose existing MyVariant.info REST client defines the interface
    pattern that the new annotation-source module must follow
  repo: https://github.com/next-crs4/VariantAlert
- name: MyVariant.info
  role: Reference external annotation source whose client and JSON annotation convention
    serve as the template for new modules
- name: DisGeNET
  role: Target annotation database to integrate as a new pluggable REST-API source
    for gene-disease annotations
- name: MasterMind
  role: Optional additional annotation source (Genomenon's) to integrate via a parallel
    adapter against its REST API
- name: Django
  role: Python web framework hosting the client methods that forward user requests
    and manage REST web service responses
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/humu.24495
    title: 'VariantAlert: A web‐based tool to notify updates in genetic variant annotations'
  source_content_hash: sha256:f61e31672d90e69e55f4ad4499b3dc1d2fc40cd02a7f986851a2f541d4331a17
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# annotation-source-interface-abstraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extends VariantAlert's annotation pipeline by adding new external annotation sources (e.g. DisGeNET, Genomenon's MasterMind) as pluggable modules that follow the same REST-API client interface VariantAlert already uses for MyVariant.info.

## When to use

Use when extending a variant re-annotation pipeline (e.g. VariantAlert, Django/Python) with a new external annotation database accessible via REST API, by implementing a module that mirrors the existing MyVariant.info client interface.

## When NOT to use

- The goal is to re-annotate variants using the already-integrated sources (gnomAD, COSMIC, ClinVar, CIViC) — use VariantAlert's existing daily re-annotation workflow instead of adding a new module.
- The new annotation source has no REST API (e.g. requires manual file upload or offline annotation, like iVar) — the interface-abstraction pattern depends on programmatic API access.
- Variant identifiers are not mappable to the target API's query scheme (e.g. non-HGVS identifiers that cannot be converted to genomic HGVS format).

## Inputs

- Variant list(s) with variant identifiers in genomic HGVS format (e.g. chr1:g.35367G>A), submitted as single variants or CSV/VCF batch files
- REST API endpoints of the external annotation resources (MyVariant.info, DisGeNET, Genomenon's MasterMind)
- VariantAlert Django/Python codebase with the existing MyVariant.info REST client methods

## Outputs

- Pluggable annotation-source module (e.g. DisGeNET adapter) implementing the same interface as the MyVariant.info client
- Structured JSON annotation records conforming to the MyVariant.info JSON annotation convention, stored in the VariantAlert database
- Optional parallel adapter module for Genomenon's MasterMind REST API

## How to apply

First inspect the VariantAlert (Django/Python) codebase to locate the client methods that forward user requests to MyVariant.info and manage the REST web service responses; this client defines the interface contract the new module must satisfy. Implement a DisGeNET annotation-source module exposing that same interface, mapping variant identifiers in genomic HGVS format (e.g. chr1:g.35367G>A) to DisGeNET REST API queries for gene-disease annotations; optionally add a parallel adapter for Genomenon's MasterMind under the same contract. Run the module on a small set of test variants and store returned annotations as structured JSON records matching the MyVariant.info JSON annotation convention, so VariantAlert's change-detection rules can consume them. Validate that every test variant yields a structured record whose fields are parseable by the change-detection rules: any difference in string fields is reported, and allele-frequency changes are flagged only at >= 1 order of magnitude (e.g. 3.830e-06 to 3.979e-06 is not notified).

## Related tools

- **VariantAlert** (Web application whose existing MyVariant.info REST client defines the interface pattern that the new annotation-source module must follow) — https://github.com/next-crs4/VariantAlert
- **MyVariant.info** (Reference external annotation source whose client and JSON annotation convention serve as the template for new modules)
- **DisGeNET** (Target annotation database to integrate as a new pluggable REST-API source for gene-disease annotations)
- **MasterMind** (Optional additional annotation source (Genomenon's) to integrate via a parallel adapter against its REST API)
- **Django** (Python web framework hosting the client methods that forward user requests and manage REST web service responses)

## Evaluation signals

- The new module returns a structured annotation record for every variant in the test set.
- Record fields are parseable by VariantAlert's change-detection rules: any difference on string fields (e.g. transcript name) is reported.
- Allele-frequency changes are flagged only at >= 1 order of magnitude (e.g. 3.830e-06 to 3.979e-06 must NOT be notified).
- The adapter exposes the same interface contract as the existing MyVariant.info client methods, so it drops into the daily re-annotation and periodic re-query loop without code changes elsewhere.
- Stored annotations follow the MyVariant.info JSON annotation convention so differences can be compared and highlighted.

## Limitations

- New annotation sources (MasterMind, DisGeNET) are described in the article as planned, not yet integrated — the pattern must be validated against the real APIs.
- Notifications for allele-frequency fields are suppressed for changes smaller than 1 order of magnitude, so subtle updates from the new source will not trigger emails.
- No changelog is available for the VariantAlert software, so tracking codebase changes that may affect the client interface requires manual inspection.

## Evidence

- [abstract] VariantAlert provides daily re‐annotation of variants using external resources accessed through application programming interface: "VariantAlert provides daily re‐annotation of variants using external resources accessed through application programming interface"
- [methods] REST client methods: "methods have been developed to forward user requests (coming from the user interface or retrieved from the database) and to manage the responses of the REST web service"
