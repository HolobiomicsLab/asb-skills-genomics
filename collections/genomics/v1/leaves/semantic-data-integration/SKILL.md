---
name: semantic-data-integration
description: Use when several RDF/OWL knowledge sources (e.g. DisGeNET-RDF, SIO, HOOM)
  must be merged into one locally queryable triple store (TDB/HDT) to support offline
  SPARQL-based gene-disease association retrieval in a diagnostic pipeline.
when_to_use_negative:
- The task is benchmarking or comparing prioritization tools (e.g. ranking causal
  genes across Phenomizer, hiPHIVE, PhenIX, GADO) rather than constructing the knowledge
  base itself.
- Up-to-date, beyond-snapshot associations are required — the store is a static build
  from DisGeNET r6.0/r5.0, SIO v1.43, and HOOM r1.3 and must be rebuilt when sources
  change.
- The inputs are not RDF/ontology files (e.g. plain TSV phenotype lists or VCFs);
  those are consumed downstream by VIBE, not integrated here.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_3361
tools:
- name: Apache Jena
  role: Builds the combined TDB triple store from the downloaded RDF sources (v3.12.0)
- name: VIBE
  role: 'Consumer and validator of the final triple store: runs SPARQL queries (HPO-to-CUI
    mapping, GDA retrieval) against the packaged HDT to confirm the store is queryable
    offline'
  repo: https://github.com/molgenis/vibe
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:f6c784b03ec5267bd7d16bcc8ff8537f58d9f2a286df1dd434ed888eb0c23e3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Apache Jena
  - Java
  - VIBE
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# semantic-data-integration

## Summary

Combines multiple RDF/ontology knowledge sources (DisGeNET-RDF, SIO, Orphadata HOOM) into a single Apache Jena TDB triple store, minimized via a SPARQL CONSTRUCT query and packaged as an HDT file, providing the locally queryable knowledge base behind VIBE's phenotype-based gene prioritization.

## When to use

Use when several RDF/OWL knowledge sources (e.g. DisGeNET-RDF, SIO, HOOM) must be merged into one locally queryable triple store (TDB/HDT) to support offline SPARQL-based gene-disease association retrieval in a diagnostic pipeline.

## When NOT to use

- The task is benchmarking or comparing prioritization tools (e.g. ranking causal genes across Phenomizer, hiPHIVE, PhenIX, GADO) rather than constructing the knowledge base itself.
- Up-to-date, beyond-snapshot associations are required — the store is a static build from DisGeNET r6.0/r5.0, SIO v1.43, and HOOM r1.3 and must be rebuilt when sources change.
- The inputs are not RDF/ontology files (e.g. plain TSV phenotype lists or VCFs); those are consumed downstream by VIBE, not integrated here.

## Inputs

- DisGeNET r6.0 RDF representation (core gene-disease/variant-disease associations)
- DisGeNET r5.0 supplement files: pda.ttl, phenotype.ttl, void.ttl
- SIO (Semanticscience Integrated Ontology) v1.43
- Orphadata HOOM (The HPO-ORDO Ontological Module) r1.3

## Outputs

- Apache Jena TDB triple store combining all sources
- Minimized dataset in TTL format extracted via SPARQL CONSTRUCT
- Final downloadable knowledge base distributed as an HDT file with sha256 checksum

## How to apply

Download the RDF representation of DisGeNET r6.0 as the core source and supplement it with pda.ttl, phenotype.ttl, and void.ttl from DisGeNET r5.0; add the SIO (Semanticscience Integrated Ontology) v1.43 — used by DisGeNET to semantically harmonize gene-disease associations — and Orphadata HOOM (HPO-ORDO Ontological Module) r1.3 for extra phenotype-disease associations. Load all sources into a single TDB triple store built with Apache Jena (v3.12.0). On that triple store, execute a SPARQL CONSTRUCT query to extract a minimized dataset in TTL format, then rebuild the final TDB from the minimized TTL set — this is the downloadable database VIBE consumes. For operational use, package the store as an HDT file (e.g. vibe-<db-version>.hdt) with a sha256 checksum, per the repository distribution convention. Decision point: if parallel benchmark runs are needed (as in vibe-suppl), copy the TDB directory once per bash script (TDB0/, TDB1/, ...) rather than sharing one instance.

## Related tools

- **Apache Jena** (Builds the combined TDB triple store from the downloaded RDF sources (v3.12.0))
- **VIBE** (Consumer and validator of the final triple store: runs SPARQL queries (HPO-to-CUI mapping, GDA retrieval) against the packaged HDT to confirm the store is queryable offline) — https://github.com/molgenis/vibe

## Examples

```
java -jar vibe-with-dependencies.jar -d -t vibe-5.0.0.hdt -w hp.owl -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- The TDB store builds with Apache Jena without errors and a SPARQL CONSTRUCT query returns a valid minimized TTL dataset.
- The rebuilt final TDB is fully queryable offline — e.g. running VIBE with HPO terms (-p HP:0002996) against the HDT returns ranked genes with GDA scores.
- The distributed database is packaged as an HDT file accompanied by a sha256 checksum that verifies after download/extract.
- All expected source vocabularies are present: UMLS CUIs, ORDO terms via HOOM, and NCBI gene identifiers grouped from DisGeNET GDAs.

## Limitations

- The build mixes DisGeNET r6.0 as core source with pda.ttl, phenotype.ttl, and void.ttl supplement files taken from DisGeNET r5.0, so supplement content may lag the core release.
- For parallel workloads each TDB directory must be duplicated per process (TDB0/, TDB1/, ...); sharing a single TDB across concurrent runs is not supported.
- The store is a static snapshot; external sources such as AMELIE cannot always be used since privacy and availability are not guaranteed, which is one rationale for the local build.

## Evidence

- [methods] TDB combined store build: "All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0)"
- [methods] SPARQL minimization step: "On this triple store, a SPARQL construct query is executed to obtain a minimized dataset in TTL format"
- [methods] SIO rationale: [evidence span withheld — anchor status: not_found]
- [methods] HOOM supplement: [evidence span withheld — anchor status: not_found]
- [readme] Distribution convention: "Download and extract [HDT][hdt_download] ([sha256 checksum](./database/checksums/database.sha256))"
- [readme] Parallel TDB copies: [evidence span withheld — anchor status: not_found]
