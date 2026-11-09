---
name: hpo-to-cui-ontology-mapping
description: Use when HPO-coded patient phenotypes (e.g. HP:0002996, HP:0001377) must
  be translated into UMLS CUI sets as a prerequisite for retrieving gene-disease associations
  or ranking candidate genes.
when_to_use_negative:
- When phenotypes are given as free-text names rather than HPO codes — first convert
  them via hp.obo/hp.owl (as done in the VIBE benchmark where phenotype names were
  matched to HPO release v2018-03-08)
- When the goal is direct phenotype-to-gene ranking via a different algorithm (e.g.
  Phenomizer, hiPHIVE, PhenIX, GADO) rather than ontology-based CUI lookup feeding
  DisGeNET GDAs
- When external web services are acceptable — VIBE's local triple-store approach exists
  precisely because external services cannot always be used since privacy and availability
  are not guaranteed
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0089
tools:
- name: VIBE
  role: Executes the SPARQL HPO-to-CUI/UMLS/ORDO mapping over the local triple store
    and writes the CUI map; command-line Java 8 tool
  repo: https://github.com/molgenis/vibe
- name: Apache Jena 3.12.0
  role: Builds the TDB triple store from DisGeNET RDF, SIO, HOOM and phenotype data
    that the SPARQL mapping query runs against
  repo: https://github.com/molgenis/vibe
- name: TestNG 7.0.0
  role: Unit test suite validating the mapping implementation, compiled with Apache
    Maven 3.3.9
  repo: https://github.com/molgenis/vibe
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:c36e30ab2585dd6f4a6c321b4f001e6c707794972e43fed85203472b7cbc7eaa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Apache Jena
  - Apache Jena 3.12.0
  - Apache Maven
  - Java 8
  - TestNG
  - TestNG 7.0.0
  - VIBE
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# hpo-to-cui-ontology-mapping

## Summary

Maps patient symptoms encoded as Human Phenotype Ontology (HPO) codes to UMLS concept unique identifiers (CUIs) using an optimized SPARQL query with three parallel matching branches, as implemented in VIBE 2.0 for phenotype-based gene prioritization.

## When to use

Use when HPO-coded patient phenotypes (e.g. HP:0002996, HP:0001377) must be translated into UMLS CUI sets as a prerequisite for retrieving gene-disease associations or ranking candidate genes.

## When NOT to use

- When phenotypes are given as free-text names rather than HPO codes — first convert them via hp.obo/hp.owl (as done in the VIBE benchmark where phenotype names were matched to HPO release v2018-03-08)
- When the goal is direct phenotype-to-gene ranking via a different algorithm (e.g. Phenomizer, hiPHIVE, PhenIX, GADO) rather than ontology-based CUI lookup feeding DisGeNET GDAs
- When external web services are acceptable — VIBE's local triple-store approach exists precisely because external services cannot always be used since privacy and availability are not guaranteed

## Inputs

- HPO phenotype codes supplied as search terms (e.g. HP:0002996, HP:0001377)
- Prebuilt TDB triple store (DisGeNET r6.0 RDF + pda.ttl/phenotype.ttl/void.ttl from DisGeNET r5.0 + SIO v1.43 + Orphadata HOOM r1.3)
- Optional HPO OWL file (e.g. hp.owl) for child/distance-based ontology traversal

## Outputs

- Per-HPO-code CUI sets written to a tab-delimited file (hpo_to_cui_map.tsv)
- Union of CUIs used downstream to retrieve matching GDAs grouped by NCBI gene identifiers

## How to apply

Load the prebuilt TDB triple store (DisGeNET r6.0 RDF supplemented with pda.ttl, phenotype.ttl and void.ttl from DisGeNET r5.0, SIO v1.43, and Orphadata HOOM r1.3, built with Apache Jena v3.12.0) and read the sample HPO codes as search terms. Execute the optimized SPARQL query that branches into three paths: (a) HPO-to-CUI matching via the UMLS Metathesaurus, (b) HPO-to-ORDO matching via HOOM, and (c) HPO-to-disease matching via DisGeNET PDAs. Compute the union of the CUIs returned by the three branches so that a phenotype reachable through any ontology route contributes mappings. Optionally expand the phenotypic search space by supplying an HPO OWL file with the -w option together with -n (children or distance-based ontology traversal) and -m (maximum distance) to include related/child HPO terms. Write the per-HPO-code CUI sets to a tab-delimited output file such as hpo_to_cui_map.tsv.

## Related tools

- **VIBE** (Executes the SPARQL HPO-to-CUI/UMLS/ORDO mapping over the local triple store and writes the CUI map; command-line Java 8 tool) — https://github.com/molgenis/vibe
- **Apache Jena 3.12.0** (Builds the TDB triple store from DisGeNET RDF, SIO, HOOM and phenotype data that the SPARQL mapping query runs against) — https://github.com/molgenis/vibe
- **TestNG 7.0.0** (Unit test suite validating the mapping implementation, compiled with Apache Maven 3.3.9) — https://github.com/molgenis/vibe

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- Compare produced CUI sets against UMLS crosswalk references for each queried HPO code
- Confirm the TestNG v7.0.0 unit test suite passes (built with Apache Maven v3.3.9)
- Verify the output is tab-delimited with one row per HPO code and CUI sets covering all three branches (UMLS Metathesaurus, HOOM ORDO, DisGeNET PDAs)
- Downstream check: the union of CUIs retrieves matching GDAs that group by unique NCBI gene identifiers and yield a ranked gene list

## Limitations

- VIBE shows a bias with over-represented genes (e.g. MECP2, KMT2D) at high positions, to be resolved in upcoming versions
- VIBE lacks exceptionally well prioritized genes near the bottom of the rank distribution compared with other tools
- Ontology traversal via -n/-m requires supplying an HPO OWL file; the README warns to keep issue #25 in mind, especially when using -n
- Phenotype-gene matching embedded in variant prioritization applies only to SNVs and indels, excluding structural variation, CGH, Sanger sequencing, cytogenetics, RNA-sequencing and metabolomics

## Evidence

- [methods] VIBE first maps the HPO search terms to CUIs: [evidence span withheld — anchor status: not_found]
- [methods] three matching branches: "(a) HPO to CUI matching according to UMLS Metathesaurus, (b) HPO to ORDO matching according to HOOM, (c) HPO to the gene‐diseases associations according to DisGeNET PDAs"
- [methods] union of CUIs and GDA ranking: [evidence span withheld — anchor status: not_found]
- [readme] ontology traversal options: "−n to select child or distance based ontology traversal, −w to supply an HPO OWL file when using the ‐m and ‐n options to increase phenotypic search space"
- [methods] triple store construction: "All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0)"
