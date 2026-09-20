---
name: gene-prioritization-ranking-by-gda-score
description: Use when HPO terms (or other phenotype concepts) have already been mapped
  to UMLS CUIs and you need to retrieve matching DisGeNET GDAs from a triple store
  and output a gene list ranked by highest GDA score.
when_to_use_negative:
- Upstream phenotype terms have not yet been converted to CUIs — this skill consumes
  the union of CUIs, not raw HPO codes
- The goal is variant-level filtering (allele frequency, inheritance mode, in silico
  pathogenicity) rather than phenotype-based gene prioritization
- A different prioritization metric than the highest GDA score per gene is required,
  or the disease associations are not represented as an RDF triple store
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: VIBE
  role: Command-line tool whose GDA retrieval and ranking module executes the SPARQL
    queries, groups GDAs by NCBI gene identifier, and writes the ranked tab-delimited
    output
  repo: https://github.com/molgenis/vibe
- name: Apache Jena
  role: Used to build and query the TDB triple store (DisGeNET-RDF) from which matching
    GDAs are retrieved via SPARQL
- name: Java 8
  role: Runtime language in which VIBE 2.0, including the retrieval/ranking module,
    was programmed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:94d5790761d2c286c745b2a467ac3bd1ea71190e7bf7054f0de931a9e68f3663
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Apache Jena
  - Java 8
  - VIBE
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Gene prioritization ranking by GDA score

## Summary

Retrieves all gene-disease associations (GDAs) matching a union of CUIs from DisGeNET in a triple store, groups them by unique NCBI gene identifier, and ranks genes descending by their highest GDA score to produce a prioritized candidate gene list for genome diagnostics.

## When to use

Use when HPO terms (or other phenotype concepts) have already been mapped to UMLS CUIs and you need to retrieve matching DisGeNET GDAs from a triple store and output a gene list ranked by highest GDA score.

## When NOT to use

- Upstream phenotype terms have not yet been converted to CUIs — this skill consumes the union of CUIs, not raw HPO codes
- The goal is variant-level filtering (allele frequency, inheritance mode, in silico pathogenicity) rather than phenotype-based gene prioritization
- A different prioritization metric than the highest GDA score per gene is required, or the disease associations are not represented as an RDF triple store

## Inputs

- Union of UMLS CUIs produced by the upstream HPO-to-CUI mapping step
- Pre-built Apache Jena TDB triple store (VIBE HDT/TDB database built from DisGeNET-RDF, DisGeNET r6.0/r5.0, SIO v1.43, Orphadata HOOM r1.3)
- SPARQL queries for GDA retrieval from the main VIBE repository

## Outputs

- Tab-delimited gene ranking file (e.g. results.tsv) with columns: gene (NCBI), gene symbol (HGNC), highest GDA score, diseases (UMLS) with sources
- Genes ordered in descending order of highest GDA score, with GDA scores and PubMed identifiers of supporting literature grouped per CUI matched to each gene

## How to apply

Load the union of CUIs produced by the upstream HPO-to-CUI mapping step (performed via SPARQL against UMLS Metathesaurus data) together with the pre-built TDB triple store containing DisGeNET-RDF (DisGeNET r6.0 core, supplemented by DisGeNET r5.0 pda.ttl/phenotype.ttl/void.ttl, SIO v1.43, and Orphadata HOOM r1.3). Execute SPARQL queries (built with Apache Jena 3.12.0) against the triple store using the union of CUIs to retrieve all matching GDAs. Group the retrieved GDAs by unique NCBI gene identifier and assign each gene the highest GDA score within its group as its priority, ordering genes in descending order of that score. Write the result as a tab-delimited file (e.g. results.tsv) with columns for gene (NCBI), gene symbol (HGNC), highest GDA score, and diseases (UMLS) with supporting literature sources per matched CUI.

## Related tools

- **VIBE** (Command-line tool whose GDA retrieval and ranking module executes the SPARQL queries, groups GDAs by NCBI gene identifier, and writes the ranked tab-delimited output) — https://github.com/molgenis/vibe
- **Apache Jena** (Used to build and query the TDB triple store (DisGeNET-RDF) from which matching GDAs are retrieved via SPARQL)
- **Java 8** (Runtime language in which VIBE 2.0, including the retrieval/ranking module, was programmed)

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- Output file is tab-delimited and contains the gene (NCBI), highest GDA score, and diseases (UMLS) columns
- Genes are ordered by descending highest GDA score
- Every gene row carries the highest GDA score found among the GDAs grouped under its unique NCBI gene identifier
- Supporting GDA scores and PubMed identifiers are grouped per CUI matched to each gene
- Retrieval returns only GDAs matching at least one CUI from the union produced by the HPO-to-CUI mapping step

## Limitations

- VIBE shows a bias with over-represented genes (e.g. MECP2, KMT2D) at high positions in the ranked output, to be resolved in upcoming versions
- VIBE lacks exceptionally well prioritized genes near the bottom of the rank distribution
- Ranking depends on the completeness and literature bias of DisGeNET gene-disease associations; performance is complementary to other tools, so cases missed by VIBE may be found by tools such as Phenomizer, PhenoTips, hiPHIVE, PhenIX, PubCaseFinder, AMELIE, or GADO

## Evidence

- [methods] A union of resulting CUIs is then used to retrieve all matching GDAs (Gene‐Disease Associations). The GDAs are grouped by unique NCBI gene identifiers: [evidence span withheld — anchor status: not_found]
- [readme] gene (NCBI)	gene symbol (HGNC)	highest GDA score	diseases (UMLS) with sources: "gene (NCBI)	gene symbol (HGNC)	highest GDA score	diseases (UMLS) with sources"
- [methods] All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0): "All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0)"
- [methods] VIBE first maps the HPO search terms to CUIs (concept unique identifiers) from UMLS (Unified Medical Language System) using a SPARQL query: [evidence span withheld — anchor status: not_found]
