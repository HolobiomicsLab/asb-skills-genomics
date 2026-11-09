---
name: biomedical-ontology-harmonization
description: Use when multiple RDF/ontology sources (e.g. DisGeNET-RDF, HOOM, SIO)
  must be merged into one queryable triple store so HPO-coded patient symptoms can
  be mapped to gene-disease associations offline.
when_to_use_negative:
- When a single ontology or knowledge source suffices and no cross-source harmonization
  (e.g. HPO-to-ORDO or HPO-to-UMLS CUI mapping) is needed
- When external web services (e.g. AMELIE) are acceptable and a locally installed,
  privacy-preserving knowledge base is not required
- When the input is already a flat gene-phenotype prioritization matrix that does
  not need RDF/SPARQL integration
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: Apache Jena 3.12.0
  role: Builds the TDB triple store that combines all downloaded knowledge sources
    and executes the SPARQL CONSTRUCT query for the minimized TTL dataset
- name: VIBE
  role: Consumes the resulting HDT knowledge base and maps HPO search terms to UMLS
    CUIs and gene-disease associations via SPARQL queries
  repo: https://github.com/molgenis/vibe
- name: DisGeNET-RDF
  role: Core RDF data source providing gene-disease and variant-disease associations
    loaded into the triple store
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:c6f2af763fb41071ff218157905b416ae30ba6835d215e632c5dab40e48693d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Apache Jena
  - Apache Jena 3.12.0
  - DisGeNET-RDF
  - Java
  - VIBE
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# biomedical-ontology-harmonization

## Summary

Combines heterogeneous biomedical knowledge sources (DisGeNET-RDF gene-disease associations, Orphadata HOOM phenotype-disease mappings, and the SIO ontology) into a single TDB triple store built with Apache Jena v3.12.0, yielding a locally queryable knowledge base for phenotype-based gene prioritization.

## When to use

Use when multiple RDF/ontology sources (e.g. DisGeNET-RDF, HOOM, SIO) must be merged into one queryable triple store so HPO-coded patient symptoms can be mapped to gene-disease associations offline.

## When NOT to use

- When a single ontology or knowledge source suffices and no cross-source harmonization (e.g. HPO-to-ORDO or HPO-to-UMLS CUI mapping) is needed
- When external web services (e.g. AMELIE) are acceptable and a locally installed, privacy-preserving knowledge base is not required
- When the input is already a flat gene-phenotype prioritization matrix that does not need RDF/SPARQL integration

## Inputs

- DisGeNET r6.0 RDF representation (core data source)
- pda.ttl, phenotype.ttl, void.ttl supplement files from DisGeNET r5.0
- SIO (Semanticscience Integrated Ontology) v1.43
- Orphadata HOOM (HPO-ORDO Ontological Module) r1.3
- HPO OWL file (e.g. hp.owl, release v2018-03-08 used in benchmark)
- HPO search terms (e.g. HP:0002996, HP:0001377) supplied via -p flags

## Outputs

- TDB triple store built with Apache Jena v3.12.0
- Minimized dataset in TTL format (via SPARQL CONSTRUCT query)
- Final downloadable TDB/HDT knowledge base with sha256 checksum
- Ranked gene list with highest GDA score and diseases (UMLS CUIs) with sources (TSV)

## How to apply

Download the RDF representation of DisGeNET r6.0 as the core source, supplement it with pda.ttl, phenotype.ttl, and void.ttl from DisGeNET r5.0, and add SIO v1.43 (used by DisGeNET to semantically harmonize its gene-disease associations) plus Orphadata HOOM r1.3 for additional phenotype-disease associations. Load all sources into a TDB triple store built with Apache Jena v3.12.0, then run a SPARQL CONSTRUCT query over the store to extract a minimized dataset in TTL format; rebuild the final TDB store from this minimized TTL set. Distribute the resulting store as an HDT file accompanied by a sha256 checksum, following the VIBE repository convention. At query time, harmonize vocabularies along three axes: HPO to CUIs via UMLS Metathesaurus (SPARQL query), HPO to ORDO via HOOM, and HPO to gene-disease associations via DisGeNET PDAs, optionally expanding the search space with child- or distance-based ontology traversal using an HPO OWL file.

## Related tools

- **Apache Jena 3.12.0** (Builds the TDB triple store that combines all downloaded knowledge sources and executes the SPARQL CONSTRUCT query for the minimized TTL dataset)
- **VIBE** (Consumes the resulting HDT knowledge base and maps HPO search terms to UMLS CUIs and gene-disease associations via SPARQL queries) — https://github.com/molgenis/vibe
- **DisGeNET-RDF** (Core RDF data source providing gene-disease and variant-disease associations loaded into the triple store)

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -w hp.owl -n distance -m 1 -o results.tsv -p HP:0002996
```

## Evaluation signals

- The final TDB store builds with Apache Jena v3.12.0 without errors and is fully queryable offline via SPARQL
- The SPARQL CONSTRUCT query returns a minimized TTL dataset from which the final TDB/HDT store is rebuilt
- The distributed database is packaged as an HDT file accompanied by a sha256 checksum that verifies
- HPO input terms (e.g. HP:0002996) resolve through the three harmonization routes: HPO-to-CUI via UMLS Metathesaurus, HPO-to-ORDO via HOOM, and HPO-to-GDA via DisGeNET PDAs
- Running VIBE against the built store returns gene (NCBI), gene symbol (HGNC), highest GDA score, and diseases (UMLS) with sources in the expected output format

## Limitations

- The knowledge base is tied to fixed dataset releases (DisGeNET r6.0 with r5.0 supplements, SIO v1.43, HOOM r1.3, HPO v2018-03-08), so re-harmonization is needed when sources update
- When using the -n option to expand the search space via ontology traversal, known issues apply (see GitHub issue #25 in the VIBE repository)
- VIBE shows a bias with over-represented genes (e.g. MECP2, KMT2D) at high rank positions, which the authors aim to resolve in upcoming versions

## Evidence

- [methods] All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0): "All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0)"
- [methods] We also included SIO (Semanticscience Integrated Ontology, v1.43), used by DisGeNET to semantically harmonize its gene‐disease associations: [evidence span withheld — anchor status: not_found]
- [methods] On this triple store, a SPARQL construct query is executed to obtain a minimized dataset in TTL format: "On this triple store, a SPARQL construct query is executed to obtain a minimized dataset in TTL format"
- [readme] Download and extract [HDT][hdt_download] ([sha256 checksum](./database/checksums/database.sha256)): "Download and extract [HDT][hdt_download] ([sha256 checksum](./database/checksums/database.sha256))"
