---
name: sparql-gene-disease-association-retrieval
description: Use when HPO patient phenotypes have already been mapped to UMLS CUIs
  and a ranked candidate gene list must be produced from a DisGeNET-based RDF triple
  store.
when_to_use_negative:
- Upstream HPO-to-CUI mapping has not been performed — this skill consumes CUIs, not
  raw HPO terms (the tool internally maps HPO terms to CUIs first)
- No local semantic dataset is available and dependence on external web services is
  required — VIBE is designed for local, stand-alone use in routine genome diagnostics
- The goal is variant-level prioritization (e.g. filtering on allele frequency, inheritance
  mode, or pathogenicity) rather than phenotype-based candidate gene prioritization
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3062
- http://edamontology.org/topic_0091
tools:
- name: VIBE
  role: Command-line tool implementing CUI-based SPARQL GDA retrieval, grouping by
    NCBI gene ID, and ranking by highest GDA score
  repo: https://github.com/molgenis/vibe
- name: Apache Jena
  role: Builds the TDB triple store (v3.12.0) queried via SPARQL to retrieve GDAs
    from DisGeNET-RDF
- name: Java 8
  role: Runtime/implementation language for VIBE 2.0 executing the retrieval and ranking
    workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:4feb64ddf01c5c2b374b57e70e14262e8b94f90c0f150ce64b062820af240e66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Apache Jena
  - Java 8
  - VIBE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3062
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# sparql-gene-disease-association-retrieval

## Summary

Retrieves gene-disease associations (GDAs) from a pre-built semantic triple store using the union of CUIs produced by upstream HPO-to-CUI mapping, then ranks genes by their highest GDA score. This is the core gene-prioritization step of VIBE (v2.0), implemented in Java 8 with SPARQL queries over DisGeNET-derived data.

## When to use

Use when HPO patient phenotypes have already been mapped to UMLS CUIs and a ranked candidate gene list must be produced from a DisGeNET-based RDF triple store.

## When NOT to use

- Upstream HPO-to-CUI mapping has not been performed — this skill consumes CUIs, not raw HPO terms (the tool internally maps HPO terms to CUIs first)
- No local semantic dataset is available and dependence on external web services is required — VIBE is designed for local, stand-alone use in routine genome diagnostics
- The goal is variant-level prioritization (e.g. filtering on allele frequency, inheritance mode, or pathogenicity) rather than phenotype-based candidate gene prioritization

## Inputs

- Union of CUIs (UMLS concept unique identifiers) from the HPO-to-CUI mapping step
- Pre-built Apache Jena TDB triple store (VIBE 2.0) or downloaded HDT dataset combining DisGeNET r6.0 RDF, DisGeNET r5.0 pda.ttl/phenotype.ttl/void.ttl, SIO v1.43, and Orphadata HOOM r1.3
- HPO term(s) supplied as -p arguments (e.g. HP:0002996)
- Optional HPO OWL file (hp.owl) when using ontology traversal options -m/-n

## Outputs

- Tab-delimited gene prioritization file with columns: gene (NCBI), gene symbol (HGNC), highest GDA score, and diseases (UMLS) with sources
- Genes ordered by descending highest GDA score
- All GDA scores and PubMed identifiers of supporting literature grouped per CUI matched to each gene

## How to apply

Load the union of CUIs produced by the upstream HPO-to-CUI SPARQL mapping step together with the pre-built semantic data store (a TDB triple store built with Apache Jena v3.12.0 in VIBE 2.0; newer releases distribute an HDT dataset). Execute SPARQL queries against the store using the union of CUIs to retrieve all matching GDAs from DisGeNET (r6.0 RDF core, supplemented with r5.0 pda.ttl/phenotype.ttl/void.ttl, SIO v1.43, and Orphadata HOOM r1.3). Group the retrieved GDAs by unique NCBI gene identifier and assign each gene the highest GDA score within its group as its priority, so ranking is driven by the strongest single association per gene rather than an aggregate. Write a tab-delimited output file with genes in descending order of highest GDA score, carrying all GDA scores and PubMed identifiers of supporting literature grouped per CUI matched to each gene. Optional ontology traversal options (-m/-n with -w for an HPO OWL file) can expand the phenotypic search space via child or distance-based traversal before retrieval.

## Related tools

- **VIBE** (Command-line tool implementing CUI-based SPARQL GDA retrieval, grouping by NCBI gene ID, and ranking by highest GDA score) — https://github.com/molgenis/vibe
- **Apache Jena** (Builds the TDB triple store (v3.12.0) queried via SPARQL to retrieve GDAs from DisGeNET-RDF)
- **Java 8** (Runtime/implementation language for VIBE 2.0 executing the retrieval and ranking workflow)

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -w hp.owl -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- Output file is tab-delimited and contains the gene (NCBI), highest GDA score, and diseases (UMLS) columns
- Genes appear in descending order of highest GDA score
- Each gene carries all GDA scores and PubMed identifiers of supporting literature grouped per matched CUI
- Every retrieved GDA traces back to one of the input CUIs from the HPO-to-CUI mapping union
- Benchmark ranking behavior matches the published evaluation (305 patient cases; median of 25 permutations used to stabilize ranks)

## Limitations

- VIBE shows a bias with over-represented genes (e.g. MECP2, KMT2D) at high positions in the ranking, which the authors aim to resolve in upcoming versions
- VIBE lacks exceptionally well prioritized genes near the bottom of the rank distribution
- GDAs come mostly from literature (DisGeNET), so recall depends on the coverage of DisGeNET-RDF and the HPO-to-CUI mapping quality
- Caution is required with the -n ontology traversal option (see github.com/molgenis/vibe/issues/25), especially when setting a maximum distance

## Evidence

- [methods] Union of CUIs drives retrieval and ranking: [evidence span withheld — anchor status: not_found]
- [methods] HPO-to-CUI mapping precedes retrieval: [evidence span withheld — anchor status: not_found]
- [methods] Triple store construction: "All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0)"
- [methods] SPARQL queries published in repo: "All SPARQL queries and algorithms for data pre‐ and post‐processing can be found in the main VIBE repository"
- [readme] CLI invocation with HDT dataset and HPO terms: "java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377"
