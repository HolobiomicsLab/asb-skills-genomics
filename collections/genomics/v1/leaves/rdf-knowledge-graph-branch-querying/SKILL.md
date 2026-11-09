---
name: rdf-knowledge-graph-branch-querying
description: Use when patient symptoms are encoded as HPO codes (e.g. HP:0002996,
  HP:0001377) and must be translated into disease/concept identifiers (CUIs) via an
  RDF knowledge base to enable downstream gene prioritization.
when_to_use_negative:
- Input phenotypes are free text rather than HPO codes — the module expects HPO IDs
  already normalized against a specific HPO release (v2018-03-08 in the benchmark).
- The analysis does not need ontology-backed phenotype-to-disease translation (e.g.
  variants already filtered and prioritized by other evidence types).
- The RDF triple store has not been built — running the branch query without the TDB
  constructed by Apache Jena v3.12.0 from the DisGeNET/SIO/HOOM sources will fail.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0634
tools:
- name: VIBE 2.0
  role: Hosts the HPO-to-CUI mapping module; executes the three-branch SPARQL query
    and writes hpo_to_cui_map.tsv
  repo: https://github.com/molgenis/vibe
- name: Apache Jena 3.12.0
  role: Builds and serves the TDB triple store that the SPARQL query runs against
- name: Java 8
  role: Language VIBE (version 2.0) is programmed in, required to run the jar
- name: TestNG 7.0.0
  role: Unit test suite validating the mapping behavior
- name: Apache Maven 3.3.9
  role: Build tool compiling VIBE and its TestNG suite
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:7178f35602752baac59f46511acf162e2bbe3589b6cd1e60b4f5f5cd971d3bb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Apache Jena
  - Apache Jena 3.12.0
  - Apache Maven
  - Apache Maven 3.3.9
  - Java 8
  - TestNG
  - TestNG 7.0.0
  - VIBE
  - VIBE 2.0
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0634
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# rdf-knowledge-graph-branch-querying

## Summary

Maps HPO phenotype codes to UMLS CUIs by executing a single optimized SPARQL query against an Apache Jena TDB triple store that branches into three matching paths (UMLS Metathesaurus, HOOM HPO-ORDO, and DisGeNET PDAs), then unions the returned CUIs so phenotype-to-concept coverage is maximized.

## When to use

Use when patient symptoms are encoded as HPO codes (e.g. HP:0002996, HP:0001377) and must be translated into disease/concept identifiers (CUIs) via an RDF knowledge base to enable downstream gene prioritization.

## When NOT to use

- Input phenotypes are free text rather than HPO codes — the module expects HPO IDs already normalized against a specific HPO release (v2018-03-08 in the benchmark).
- The analysis does not need ontology-backed phenotype-to-disease translation (e.g. variants already filtered and prioritized by other evidence types).
- The RDF triple store has not been built — running the branch query without the TDB constructed by Apache Jena v3.12.0 from the DisGeNET/SIO/HOOM sources will fail.

## Inputs

- HPO phenotype codes (e.g. HP:0002996, HP:0001377)
- TDB triple store built from DisGeNET r6.0 RDF plus pda.ttl, phenotype.ttl, void.ttl (DisGeNET r5.0), SIO v1.43, and Orphadata HOOM r1.3
- Optional HPO OWL file (hp.owl) for ontology traversal via -w

## Outputs

- Per-HPO-code CUI sets written to a tab-delimited file (hpo_to_cui_map.tsv)
- Union of CUIs from the three query branches, used downstream to retrieve matching GDAs

## How to apply

Build a TDB triple store with Apache Jena v3.12.0 from DisGeNET r6.0 RDF supplemented with pda.ttl, phenotype.ttl and void.ttl from DisGeNET r5.0, SIO v1.43, and Orphadata HOOM r1.3. Read each input HPO code as a search term and execute an optimized SPARQL query whose graph pattern branches into three paths: (a) HPO-to-CUI matching via the UMLS Metathesaurus, (b) HPO-to-ORDO matching via HOOM, and (c) HPO-to-disease matching via DisGeNET PDAs. Compute the union of the CUIs returned by the three branches, since each branch contributes phenotype-disease associations the others miss (e.g. rare diseases covered by Orphanet/HOOM but not literature-based DisGeNET). Write the per-HPO-code CUI sets to a tab-delimited file (hpo_to_cui_map.tsv) so the mapping module stays an interchangeable step in a composable diagnostic pipeline. Optionally widen the phenotypic search space using the -n (child or distance based ontology traversal) and -m options together with an HPO OWL file supplied via -w.

## Related tools

- **VIBE 2.0** (Hosts the HPO-to-CUI mapping module; executes the three-branch SPARQL query and writes hpo_to_cui_map.tsv) — https://github.com/molgenis/vibe
- **Apache Jena 3.12.0** (Builds and serves the TDB triple store that the SPARQL query runs against)
- **Java 8** (Language VIBE (version 2.0) is programmed in, required to run the jar)
- **TestNG 7.0.0** (Unit test suite validating the mapping behavior)
- **Apache Maven 3.3.9** (Build tool compiling VIBE and its TestNG suite)

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -w hp.owl -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- Produced CUI sets match UMLS crosswalk references for each input HPO code
- TestNG v7.0.0 unit test suite passes when built with Apache Maven v3.3.9
- Output hpo_to_cui_map.tsv is tab-delimited with one CUI set per supplied HPO code
- Union covers concepts from all three branches — CUIs attributable to UMLS Metathesaurus, HOOM/ORDO, and DisGeNET PDAs are all present where applicable
- Downstream, the union of CUIs retrieves non-empty matching GDAs grouped by unique NCBI gene identifiers

## Limitations

- Coverage depends on the snapshot versions baked into the TDB (DisGeNET r6.0, HOOM r1.3, HPO v2018-03-08); newer codes or associations are absent.
- VIBE shows a bias with over-represented genes (e.g. MECP2, KMT2D) at high positions in the ranked output, acknowledged to be resolved in upcoming versions.
- The three branches may disagree; ontology traversal options (-n/-m) widen the search space and the README warns to keep issue #25 in mind, especially when using -n.
- The benchmark may be representative of clinical practice only to a degree and does not demonstrate individual strengths of the tested tools.

## Evidence

- [methods] VIBE first maps the HPO search terms to CUIs (concept unique identifiers) from UMLS (Unified Medical Language System) using a SPARQL query: [evidence span withheld — anchor status: not_found]
- [methods] (a) HPO to CUI matching according to UMLS Metathesaurus, (b) HPO to ORDO matching according to HOOM, (c) HPO to the gene‐diseases associations according to DisGeNET PDAs: "(a) HPO to CUI matching according to UMLS Metathesaurus, (b) HPO to ORDO matching according to HOOM, (c) HPO to the gene‐diseases associations according to DisGeNET PDAs"
- [methods] All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0): "All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0)"
- [readme] Open a terminal and run VIBE. `java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377`: "java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377"
- [methods] −n to select child or distance based ontology traversal, −w to supply an HPO OWL file when using the ‐m and ‐n options to increase phenotypic search space: "−n to select child or distance based ontology traversal, −w to supply an HPO OWL file when using the ‐m and ‐n options to increase phenotypic search space"
