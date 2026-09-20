---
name: pubmed-evidence-aggregation-per-disease
description: Use when patient symptoms have been encoded as HPO terms and mapped to
  UMLS CUIs, and the goal is to prioritize candidate genes by aggregating literature-
  and curation-backed gene-disease associations (DisGeNET GDAs) with their PubMed
  evidence.
when_to_use_negative:
- Input is raw variant calls or a VCF without phenotype information — this skill only
  prioritizes genes from phenotype-derived CUIs, not from variant-level annotations
- No HPO-coded patient phenotypes (and thus no upstream HPO-to-CUI mapping) are available,
  so there is no CUI union to query the triple store with
- A phenotype prioritizer requiring external services or variant data (e.g. Exomiser-style
  VCF-based runs) is what the task actually needs; VIBE is a stand-alone phenotype-to-gene
  module
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: VIBE
  role: Implements the CUI-union SPARQL retrieval of DisGeNET GDAs, grouping by NCBI
    gene ID, highest-GDA-score ranking, and tab-delimited output with PubMed evidence
    per CUI
  repo: https://github.com/molgenis/vibe
- name: Apache Jena 3.12.0
  role: Builds and backs the TDB triple store over which the GDA-retrieval SPARQL
    queries are executed
- name: Java 8
  role: Runtime language in which VIBE 2.0 (including this retrieval/ranking module)
    is programmed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:5eddfd12f0ef793620f26484341eaa21f8f4f86c4d98480bef00f34a78a3cc01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Apache Jena
  - Apache Jena 3.12.0
  - Java 8
  - VIBE
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# pubmed-evidence-aggregation-per-disease

## Summary

VIBE's GDA retrieval and ranking step: a union of CUIs produced by upstream HPO-to-CUI mapping is used in SPARQL queries against a DisGeNET-based triple store to retrieve all matching Gene-Disease Associations (GDAs), which are then grouped per NCBI gene and ranked by highest GDA score, carrying the supporting PubMed identifiers per matched disease (UMLS CUI).

## When to use

Use when patient symptoms have been encoded as HPO terms and mapped to UMLS CUIs, and the goal is to prioritize candidate genes by aggregating literature- and curation-backed gene-disease associations (DisGeNET GDAs) with their PubMed evidence.

## When NOT to use

- Input is raw variant calls or a VCF without phenotype information — this skill only prioritizes genes from phenotype-derived CUIs, not from variant-level annotations
- No HPO-coded patient phenotypes (and thus no upstream HPO-to-CUI mapping) are available, so there is no CUI union to query the triple store with
- A phenotype prioritizer requiring external services or variant data (e.g. Exomiser-style VCF-based runs) is what the task actually needs; VIBE is a stand-alone phenotype-to-gene module

## Inputs

- Union of CUIs (UMLS concept unique identifiers) from the upstream HPO-to-CUI mapping step
- Pre-built Apache Jena TDB triple store of DisGeNET-RDF (r6.0 core, supplemented with r5.0 pda.ttl, phenotype.ttl, void.ttl, SIO v1.43, Orphadata HOOM r1.3); later versions use a downloaded HDT dataset
- Patient phenotype input as HPO term IDs (e.g. HP:0002996), optionally with an HPO OWL file (hp.owl) when ontology traversal is enabled

## Outputs

- Tab-delimited gene prioritization file with columns: gene (NCBI), gene symbol (HGNC), highest GDA score, diseases (UMLS) with sources
- Per-gene list of all GDA scores and PubMed identifiers of supporting literature, grouped per matched CUI
- Genes ordered in descending order of highest GDA score

## How to apply

Load the union of CUIs produced by the upstream HPO-to-CUI mapping (HPO search terms are mapped to UMLS concept unique identifiers via a SPARQL query) together with the pre-built DisGeNET-RDF triple store (VIBE v2.0 uses an Apache Jena 3.12.0 TDB, later distributed as an HDT dataset). Execute SPARQL queries using the union of CUIs to retrieve all matching GDAs from DisGeNET r6.0 (supplemented with DisGeNET r5.0 pda.ttl and SIO v1.43 harmonization). Group the retrieved GDAs by unique NCBI gene identifier and assign each gene the highest GDA score within its group as its priority score; optionally expand the search space beforehand via child- or distance-based ontology traversal (-n children|distance with -m and the HPO OWL file via -w). Write the ranked result as a tab-delimited file listing genes in descending order of highest GDA score, with gene symbol (HGNC), all GDA scores, the diseases (UMLS CUIs) matched to that gene, and the PubMed identifiers of supporting literature grouped per CUI.

## Related tools

- **VIBE** (Implements the CUI-union SPARQL retrieval of DisGeNET GDAs, grouping by NCBI gene ID, highest-GDA-score ranking, and tab-delimited output with PubMed evidence per CUI) — https://github.com/molgenis/vibe
- **Apache Jena 3.12.0** (Builds and backs the TDB triple store over which the GDA-retrieval SPARQL queries are executed)
- **Java 8** (Runtime language in which VIBE 2.0 (including this retrieval/ranking module) is programmed)

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- Output file is tab-delimited and contains the expected columns: gene (NCBI), gene symbol (HGNC), highest GDA score, diseases (UMLS) with sources
- Genes appear in strictly descending order of their highest GDA score
- Every gene in the output has a group of retrieved GDAs keyed by its unique NCBI gene identifier, and the priority equals the maximum GDA score within the group
- All GDAs retrieved by the CUI-union SPARQL queries are represented — each matched disease (UMLS CUI) carries its associated GDA scores and PubMed identifiers of supporting literature
- All HPO inputs map into the CUI union used for retrieval (or ontology traversal options -n/-m/-w were correctly applied via the HPO OWL file)

## Limitations

- VIBE shows a bias with over-represented genes (e.g. MECP2, KMT2D) occupying high positions in the ranking, which the authors aim to resolve in upcoming versions
- VIBE lacks exceptionally well prioritized genes near the bottom of the rank distribution, and performance is highly complementary to other phenotype prioritizers — results should not be relied on alone
- Caution is required when using the -n ontology-traversal option (see vibe repo issue #25), as it changes the effective phenotype search space and thus the CUI union
- The accompanying benchmark scripts (vibe-suppl) assume exact intended usage; misuse may pass weak internal validation and produce unexpected behavior

## Evidence

- [methods] CUI mapping precedes GDA retrieval: [evidence span withheld — anchor status: not_found]
- [methods] CUI union to grouped, ranked GDAs: [evidence span withheld — anchor status: not_found]
- [methods] TDB triple store backing the queries: "All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0)"
- [readme] README output schema with PubMed-sourced evidence columns: "gene (NCBI)	gene symbol (HGNC)	highest GDA score	diseases (UMLS) with sources"
- [methods] DisGeNET as the GDA data source: [evidence span withheld — anchor status: not_found]
