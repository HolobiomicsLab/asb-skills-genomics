---
name: rdf-triple-store-querying
description: Use when patient symptoms encoded as HPO codes must be translated to
  UMLS CUIs and used to retrieve and rank gene-disease associations from a local RDF/SPARQL
  knowledge base.
when_to_use_negative:
- The task requires structural variant prioritization or data sources beyond SNV/indel
  variant-gene evidence — the article notes phenotype-gene matching embedded in variant
  prioritization only applies to SNVs and indels
- The knowledge base must be queried through an external web service — VIBE's design
  goal is local, offline installation precisely because external services' privacy
  and availability are not guaranteed
- No RDF/SPARQL triple store exists and building one is out of scope — this skill
  presumes a prebuilt Jena TDB/HDT database
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Apache Jena
  role: Builds the TDB triple store from combined RDF sources and executes the SPARQL
    queries (v3.12.0)
- name: VIBE
  role: Java 8 application that runs the HPO-to-CUI mapping and GDA retrieval/ranking
    SPARQL queries against the triple store
  repo: https://github.com/molgenis/vibe
- name: Java 8
  role: Runtime requirement for the VIBE jar that performs the triple store querying
provenance:
  source_task_ids:
  - task_007
  - task_005
  - task_006
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:ff5bf9b9b65935b05640526b523704e41e73c7ff61a4ce26578086bd037e7f0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Apache Jena
  - Apache Maven
  - Java
  - Java 8
  - TestNG
  - VIBE
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# rdf-triple-store-querying

## Summary

Builds a local RDF triple store from combined knowledge sources (DisGeNET-RDF, SIO, HOOM) and queries it with SPARQL to map patient HPO terms to CUIs and retrieve gene-disease associations, as done in VIBE for phenotype-based gene prioritization.

## When to use

Use when patient symptoms encoded as HPO codes must be translated to UMLS CUIs and used to retrieve and rank gene-disease associations from a local RDF/SPARQL knowledge base.

## When NOT to use

- The task requires structural variant prioritization or data sources beyond SNV/indel variant-gene evidence — the article notes phenotype-gene matching embedded in variant prioritization only applies to SNVs and indels
- The knowledge base must be queried through an external web service — VIBE's design goal is local, offline installation precisely because external services' privacy and availability are not guaranteed
- No RDF/SPARQL triple store exists and building one is out of scope — this skill presumes a prebuilt Jena TDB/HDT database

## Inputs

- HPO phenotype codes supplied via -p flags (e.g. HP:0002996, HP:0001377)
- Local HDT/TDB triple store file built with Apache Jena (DisGeNET-RDF r6.0 + pda.ttl, phenotype.ttl, void.ttl from r5.0, SIO v1.43, Orphadata HOOM r1.3)
- HPO OWL file (hp.owl) supplied via -w when using ontology traversal options

## Outputs

- Tab-delimited results.tsv listing gene (NCBI), gene symbol (HGNC), highest GDA score, and diseases (UMLS) with sources, ordered by descending highest GDA score
- Per-HPO-code CUI sets from the mapping stage (hpo_to_cui_map.tsv)
- Minimized TTL dataset extracted via SPARQL CONSTRUCT query (for store rebuilding)

## How to apply

Load the prebuilt triple store — DisGeNET r6.0 RDF supplemented with pda.ttl, phenotype.ttl and void.ttl from DisGeNET r5.0, SIO v1.43, and Orphadata HOOM r1.3 — built with Apache Jena v3.12.0 (distributed as an HDT file with a sha256 checksum). Read the input HPO codes (e.g. HP:0002996, HP:0001377) and execute a SPARQL query that branches into three matching paths: HPO-to-CUI via UMLS Metathesaurus, HPO-to-ORDO via HOOM, and HPO-to-gene-disease via DisGeNET PDAs; compute the union of returned CUIs. Run a second SPARQL query against the store using the union of CUIs to retrieve all matching GDAs, then group GDAs by unique NCBI gene identifier and assign each gene the highest GDA score within its group. Optionally expand the phenotypic search space using ontology traversal options: -n children or -n distance with -m <maximum distance> and -w hp.owl. A SPARQL CONSTRUCT query can also be run on the store to obtain a minimized TTL dataset for redistribution.

## Related tools

- **Apache Jena** (Builds the TDB triple store from combined RDF sources and executes the SPARQL queries (v3.12.0))
- **VIBE** (Java 8 application that runs the HPO-to-CUI mapping and GDA retrieval/ranking SPARQL queries against the triple store) — https://github.com/molgenis/vibe
- **Java 8** (Runtime requirement for the VIBE jar that performs the triple store querying)

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- The output file is tab-delimited and contains the expected columns: gene (NCBI), gene symbol (HGNC), highest GDA score, and diseases (UMLS) with sources
- Genes in the output are ordered by descending highest GDA score
- Each gene's GDA group corresponds to a unique NCBI gene identifier, with supporting PubMed identifiers grouped per matched CUI
- Returned CUI sets match UMLS crosswalk references and the TestNG v7.0.0 unit test suite (built with Apache Maven v3.3.9) passes
- The HDT database matches its sha256 checksum before querying

## Limitations

- VIBE lacks exceptionally well prioritized genes near the bottom of the rank distribution
- VIBE shows a bias with over-represented genes (e.g. MECP2, KMT2D) at high positions, to be resolved in upcoming versions
- Using the -n (ontology traversal) option requires an HPO OWL file and caution; the README flags a known issue when using -n (github.com/molgenis/vibe/issues/25)
- The store must be built offline from specific release combinations (DisGeNET r6.0 core, r5.0 supplements, SIO v1.43, HOOM r1.3); mismatched versions may break SPARQL matching paths

## Evidence

- [methods] All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0): "All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0)"
- [methods] On this triple store, a SPARQL construct query is executed to obtain a minimized dataset in TTL format: "On this triple store, a SPARQL construct query is executed to obtain a minimized dataset in TTL format"
- [methods] VIBE first maps the HPO search terms to CUIs from UMLS using a SPARQL query: [evidence span withheld — anchor status: not_found]
- [methods] A union of resulting CUIs is then used to retrieve all matching GDAs. The GDAs are grouped by unique NCBI gene identifiers: [evidence span withheld — anchor status: not_found]
- [readme] README quickstart invocation of VIBE against the HDT dataset with HPO codes: "java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377"
