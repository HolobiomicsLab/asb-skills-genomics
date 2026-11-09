---
name: hpo-code-to-disease-concept-mapping
description: Use when patient symptoms are already encoded as HPO codes (e.g. HP:0002996)
  and must be translated into disease concepts to retrieve gene-disease associations
  for phenotype-based candidate gene prioritization.
when_to_use_negative:
- The patient phenotypes are free-text names rather than HPO codes — they must first
  be matched to HPO v2018-03-08 to obtain corresponding codes
- You need candidate variants filtered by allele frequency, inheritance mode, or pathogenicity
  — VIBE only provides phenotype-based candidate genes, not variant-level filtering
- Privacy or availability rules forbid external web services — VIBE is designed for
  local installation precisely because external services such as AMELIE 'cannot always
  be used since privacy and availability are not guaranteed'
edam_operation: http://edamontology.org/operation_0282
tools:
- name: VIBE
  role: Performs the HPO-to-CUI/ORDO mapping via SPARQL over the HDT triple store
    and returns ranked gene-disease associations
  repo: https://github.com/molgenis/vibe
- name: Apache Jena
  role: Builds the TDB triple store from DisGeNET-RDF, HOOM, SIO, and HPO sources
    that the SPARQL mapping queries run against
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:d85afddfc199ad6574a63a6fdc083155a1298bbf4b145a24107c6a94830337b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Apache Jena
  - HPO v2018-03-08
  - Java 8
  - VIBE
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# hpo-code-to-disease-concept-mapping

## Summary

Maps patient HPO term codes onto disease concepts (UMLS CUIs and ORDO terms via HOOM) and retrieves ranked gene-disease associations from DisGeNET-RDF, as implemented inside the VIBE v2.0 command-line tool for phenotype-based gene prioritization.

## When to use

Use when patient symptoms are already encoded as HPO codes (e.g. HP:0002996) and must be translated into disease concepts to retrieve gene-disease associations for phenotype-based candidate gene prioritization.

## When NOT to use

- The patient phenotypes are free-text names rather than HPO codes — they must first be matched to HPO v2018-03-08 to obtain corresponding codes
- You need candidate variants filtered by allele frequency, inheritance mode, or pathogenicity — VIBE only provides phenotype-based candidate genes, not variant-level filtering
- Privacy or availability rules forbid external web services — VIBE is designed for local installation precisely because external services such as AMELIE 'cannot always be used since privacy and availability are not guaranteed'

## Inputs

- Patient case HPO codes (e.g. HP:0002996, HP:0001377), one or more per run via repeated -p flags
- Local VIBE HDT knowledge-base archive (DisGeNET r6.0 RDF supplemented with DisGeNET r5.0 pda.ttl/phenotype.ttl/void.ttl, SIO v1.43, HOOM r1.3)
- HPO.owl ontology file (required only when ontology traversal via -w/-m/-n is used)
- Java 8 runtime

## Outputs

- Tab-delimited ranked gene list: NCBI gene identifier, gene symbol (HGNC), highest GDA score, and pipe-separated disease (UMLS) associations with GDA scores and PubMed evidence IDs
- One ranked gene list per patient case for downstream merge/processing with vibe-suppl Python and R scripts

## How to apply

Supply the case's HPO codes to VIBE v2.0 via repeated -p arguments (e.g. -p HP:0002996), running on Java 8 against the local HDT triple store built with Apache Jena v3.12.0 from DisGeNET r6.0 RDF, pda.ttl/phenotype.ttl/void.ttl from DisGeNET r5.0, SIO v1.43, Orphadata HOOM r1.3, and HPO v2018-03-08. VIBE first maps the HPO search terms to UMLS CUIs with a SPARQL query, matches HPO to ORDO according to HOOM, and then uses the union of resulting CUIs to retrieve all matching gene-disease associations (GDAs). By default the run uses only the user-defined phenotypes with no ontology traversal; to expand the phenotypic search space, add -w with the HPO OWL file plus -n (children or distance) and -m (a maximum distance number), as in '-n distance -m 1'. GDAs are grouped by unique NCBI gene identifiers and genes are ranked by highest GDA score, yielding a tab-delimited output of NCBI gene ID, HGNC symbol, highest GDA score, and pipe-separated UMLS disease associations with sources.

## Related tools

- **VIBE** (Performs the HPO-to-CUI/ORDO mapping via SPARQL over the HDT triple store and returns ranked gene-disease associations) — https://github.com/molgenis/vibe
- **Apache Jena** (Builds the TDB triple store from DisGeNET-RDF, HOOM, SIO, and HPO sources that the SPARQL mapping queries run against)

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- Each per-case output is a tab-delimited gene list ranked in descending order of highest GDA score
- The union of CUIs derived from the input HPO codes yields non-empty GDA retrieval for every case (308 causal genes from 305 cases are recallable by rank 20)
- Gene identifiers in the output are NCBI gene IDs paired with HGNC symbols, consistent with the HGNC-to-NCBI conversion data used across the benchmark
- Disease associations appear as UMLS CUIs (plus ORDO-derived matches) with GDA scores and PubMed evidence IDs, matching the documented output schema

## Limitations

- VIBE shows a bias with over-represented genes (e.g. MECP2, KMT2D) at high positions, which the authors aim to resolve in upcoming versions
- VIBE lacks exceptionally well prioritized genes near the bottom of the rank distribution, and high complementarity exists among phenotype prioritization tools
- Ontology traversal via -m/-n changes the phenotypic search space and carries a documented caveat (GitHub issue #25); the benchmark itself ran at default settings without traversal
- The underlying DisGeNET evidence comes mostly from literature and curated variant-disease sources, so mapping quality depends on that knowledge base (r6.0 with r5.0 supplements)

## Evidence

- [methods] HPO terms mapped to CUIs via SPARQL: [evidence span withheld — anchor status: not_found]
- [methods] Three-step mapping pipeline: "(a) HPO to CUI matching according to UMLS Metathesaurus, (b) HPO to ORDO matching according to HOOM, (c) HPO to the gene‐diseases associations according to DisGeNET PDAs"
- [methods] CUI union retrieves GDAs ranked per gene: [evidence span withheld — anchor status: not_found]
- [readme] Optional ontology traversal flags: "Using the user-defined phenotypes and phenotypes that are related to them with a maximum distance of 1"
- [methods] HPO codes matched to release v2018-03-08: "The HPO terms from these cases were then matched to the HPO (release v2018‐03‐08) to obtain their corresponding codes"
