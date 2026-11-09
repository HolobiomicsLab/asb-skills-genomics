---
name: sparql-construct-querying
description: Use when an analysis consumes RDF triple stores (TDB/HDT) and you need
  graph-shaped transformations or identifier mappings, e.g. extracting a minimized
  TTL dataset or mapping HPO terms to UMLS CUIs.
when_to_use_negative:
- The data source is not RDF/SPARQL-queryable (e.g. flat benchmark TSVs or obo files
  used in the vibe-suppl benchmarking scripts)
- You need variant-level filtering (allele frequency, inheritance mode, in silico
  pathogenicity) rather than phenotype-to-gene-disease-association retrieval
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Apache Jena 3.12.0
  role: Builds the TDB triple store from the combined RDF sources and serves as the
    SPARQL query engine over it
  repo: https://github.com/molgenis/vibe
- name: VIBE 2.0
  role: Java 8 application that executes the SPARQL queries (HPO-to-CUI mapping, GDA
    retrieval) against the TDB/HDT database
  repo: https://github.com/molgenis/vibe
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:c77130693a7f15cdf8ccf739c46aaa3bff05b27bfb19c4b1588c583055f9d31f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Apache Jena
  - Apache Jena 3.12.0
  - Java
  - VIBE
  - VIBE 2.0
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# sparql-construct-querying

## Summary

Query RDF knowledge bases with SPARQL to retrieve or transform graph data: in VIBE, SPARQL queries against a TDB triple store (built with Apache Jena 3.12.0) both minimize the combined DisGeNET-RDF/HOOM/SIO dataset into TTL and map HPO search terms to UMLS CUIs during gene prioritization.

## When to use

Use when an analysis consumes RDF triple stores (TDB/HDT) and you need graph-shaped transformations or identifier mappings, e.g. extracting a minimized TTL dataset or mapping HPO terms to UMLS CUIs.

## When NOT to use

- The data source is not RDF/SPARQL-queryable (e.g. flat benchmark TSVs or obo files used in the vibe-suppl benchmarking scripts)
- You need variant-level filtering (allele frequency, inheritance mode, in silico pathogenicity) rather than phenotype-to-gene-disease-association retrieval

## Inputs

- TDB triple store combining DisGeNET-RDF (r6.0 + r5.0 supplements pda.ttl, phenotype.ttl, void.ttl), SIO v1.43, and Orphadata HOOM r1.3
- HDT-packaged VIBE database with sha256 checksum
- HPO search terms (e.g. HP:0002996) supplied on the command line
- HPO OWL file (hp.owl) for ontology traversal options

## Outputs

- Minimized dataset in TTL format extracted by SPARQL CONSTRUCT
- Final TDB triple store rebuilt from the minimized TTL set (distributed as HDT)
- Ranked gene list (NCBI gene ID, HGNC symbol, highest GDA score, diseases with sources) in TSV output

## How to apply

First combine the RDF sources — DisGeNET r6.0 plus pda.ttl, phenotype.ttl, and void.ttl from DisGeNET r5.0, SIO v1.43, and Orphadata HOOM r1.3 — into a TDB triple store built with Apache Jena 3.12.0. For dataset minimization, execute a SPARQL CONSTRUCT query on the triple store to obtain a minimized dataset in TTL format, then rebuild the final TDB store (packaged as an HDT file with a sha256 checksum) from that TTL set. At runtime, VIBE maps HPO search terms to UMLS CUIs using a SPARQL query, takes the union of resulting CUIs to retrieve matching gene-disease associations, and groups these by unique NCBI gene identifiers to rank genes by highest GDA score. All SPARQL queries and pre-/post-processing algorithms are kept in the main VIBE repository so they can be inspected or reused.

## Related tools

- **Apache Jena 3.12.0** (Builds the TDB triple store from the combined RDF sources and serves as the SPARQL query engine over it) — https://github.com/molgenis/vibe
- **VIBE 2.0** (Java 8 application that executes the SPARQL queries (HPO-to-CUI mapping, GDA retrieval) against the TDB/HDT database) — https://github.com/molgenis/vibe

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- The final TDB store rebuilds from the minimized TTL set without errors and is fully queryable offline
- The distributed database is packaged as an HDT file accompanied by a sha256 checksum that validates
- The SPARQL CONSTRUCT output is valid TTL containing only the minimized triples expected from the combined sources
- HPO search terms resolve to UMLS CUIs via the SPARQL query, and returned GDAs group by unique NCBI gene identifiers with a single highest GDA score per gene
- All SPARQL queries used are reproducible from the main VIBE repository

## Limitations

- Ontology traversal options (-m/-n with -w HPO OWL file) have documented caveats (molgenis/vibe issue #25), especially when using -n
- VIBE shows a bias with over-represented genes (MECP2, KMT2D) at high ranks, attributed to the underlying data and to be resolved in upcoming versions
- VIBE lacks exceptionally well prioritized genes near the bottom of the rank distribution

## Evidence

- [methods] TDB triple store built with Jena and CONSTRUCT minimization: "All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0)"
- [methods] SPARQL CONSTRUCT yields minimized TTL dataset: "On this triple store, a SPARQL construct query is executed to obtain a minimized dataset in TTL format"
- [methods] SPARQL query maps HPO terms to UMLS CUIs: [evidence span withheld — anchor status: not_found]
- [methods] Queries kept in repository: [evidence span withheld — anchor status: not_found]
- [readme] TDB used in benchmark runs: [evidence span withheld — anchor status: not_found]
