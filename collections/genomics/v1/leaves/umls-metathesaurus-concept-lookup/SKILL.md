---
name: umls-metathesaurus-concept-lookup
description: Use when patient symptoms are encoded as HPO codes and must be translated
  to UMLS CUIs to seed downstream gene-disease association retrieval, e.g. in the
  VIBE prioritization pipeline.
when_to_use_negative:
- The phenotypes are already provided as UMLS CUIs or as free text not resolvable
  to HPO terms (no HPO-to-CUI crosswalk is needed, or HPO matching is not possible).
- The goal is direct gene prioritization output (ranked gene list with GDA scores)
  rather than the intermediate ontology crosswalk — use the full VIBE run instead.
- Only HPO-to-ORDO or HPO-to-DisGeNET-PDA branches are needed without UMLS Metathesaurus
  CUI matching.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: VIBE
  role: Java 8 command-line tool that executes the optimized SPARQL query performing
    HPO-to-CUI matching as its first processing stage
  repo: https://github.com/molgenis/vibe
- name: Apache Jena 3.12.0
  role: Builds and serves the TDB triple store against which the HPO-to-CUI SPARQL
    query runs
  repo: https://github.com/molgenis/vibe
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:cbc019eff77ec213d60e6e72f3a4bd551296bb0c4e8fb350a616be976d974709
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
  - VIBE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# umls-metathesaurus-concept-lookup

## Summary

Maps Human Phenotype Ontology (HPO) search terms (e.g. HP:0002996, HP:0001377) to UMLS CUIs (concept unique identifiers) using an optimized SPARQL query executed against VIBE's TDB triple store, as the first step of phenotype-based disease gene prioritization.

## When to use

Use when patient symptoms are encoded as HPO codes and must be translated to UMLS CUIs to seed downstream gene-disease association retrieval, e.g. in the VIBE prioritization pipeline.

## When NOT to use

- The phenotypes are already provided as UMLS CUIs or as free text not resolvable to HPO terms (no HPO-to-CUI crosswalk is needed, or HPO matching is not possible).
- The goal is direct gene prioritization output (ranked gene list with GDA scores) rather than the intermediate ontology crosswalk — use the full VIBE run instead.
- Only HPO-to-ORDO or HPO-to-DisGeNET-PDA branches are needed without UMLS Metathesaurus CUI matching.

## Inputs

- HPO phenotype codes as search terms (e.g. HP:0002996, HP:0001377)
- Local HDT/TDB triple store built from DisGeNET r6.0 RDF, pda.ttl/phenotype.ttl/void.ttl (DisGeNET r5.0), SIO v1.43, Orphadata HOOM r1.3
- Human Phenotype Ontology OWL file (hp.owl), required when using -n/-m traversal options

## Outputs

- Per-HPO-code sets of UMLS CUIs written to a tab-delimited file (hpo_to_cui_map.tsv)
- Union of CUIs used downstream to retrieve matching GDAs from DisGeNET

## How to apply

Load the prebuilt TDB triple store (DisGeNET r6.0 RDF supplemented with pda.ttl, phenotype.ttl and void.ttl from DisGeNET r5.0, SIO v1.43, and Orphadata HOOM r1.3, built with Apache Jena v3.12.0) and read the input HPO codes as search terms. Execute the optimized SPARQL query that branches into three paths: (a) HPO-to-CUI matching via the UMLS Metathesaurus, (b) HPO-to-ORDO matching via HOOM, and (c) HPO-to-disease matching via DisGeNET PDAs. The rationale for the multi-branch design is that no single resource covers all phenotype-disease links, so the union of CUIs returned by the three branches maximizes recall before GDAs are retrieved. Optionally widen the phenotypic search space with ontology traversal: -n selects child or distance based traversal, -m sets the maximum distance, and -w supplies the HPO OWL file. Write the resulting per-HPO-code CUI sets to a tab-delimited mapping file (e.g. hpo_to_cui_map.tsv).

## Related tools

- **VIBE** (Java 8 command-line tool that executes the optimized SPARQL query performing HPO-to-CUI matching as its first processing stage) — https://github.com/molgenis/vibe
- **Apache Jena 3.12.0** (Builds and serves the TDB triple store against which the HPO-to-CUI SPARQL query runs) — https://github.com/molgenis/vibe

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -w hp.owl -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- Compare produced per-HPO-code CUI sets against UMLS crosswalk references.
- Confirm the TestNG v7.0.0 unit test suite passes (compiled with Apache Maven v3.3.9).
- Verify the union of CUIs covers matches from all three branches: UMLS Metathesaurus, HOOM/ORDO, and DisGeNET PDAs.
- Check the output is a valid tab-delimited file with one CUI set per input HPO code, and that known examples (e.g. HP:0002996) resolve to the expected concepts.

## Limitations

- The -n option (child or distance based ontology traversal with -m distance) changes the phenotypic search space and must be used carefully; the README flags a known caveat (issue #25) especially when using -n.
- The HPO OWL/OWL file version matters — benchmarking used HPO release v2018-03-08 — so crosswalk results depend on the ontology release supplied via -w.
- Coverage is bounded by what is loaded in the triple store: CUIs come only from the UMLS Metathesaurus content in the DisGeNET r6.0/r5.0 + SIO + HOOM store, so concepts absent from those resources are missed.

## Evidence

- [methods] VIBE first maps the HPO search terms to CUIs (concept unique identifiers) from UMLS (Unified Medical Language System) using a SPARQL query: [evidence span withheld — anchor status: not_found]
- [methods] (a) HPO to CUI matching according to UMLS Metathesaurus, (b) HPO to ORDO matching according to HOOM, (c) HPO to the gene‐diseases associations according to DisGeNET PDAs: "(a) HPO to CUI matching according to UMLS Metathesaurus, (b) HPO to ORDO matching according to HOOM, (c) HPO to the gene‐diseases associations according to DisGeNET PDAs"
- [methods] −n to select child or distance based ontology traversal, −w to supply an HPO OWL file when using the ‐m and ‐n options to increase phenotypic search space: "−n to select child or distance based ontology traversal, −w to supply an HPO OWL file when using the ‐m and ‐n options to increase phenotypic search space"
- [readme] java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377: "java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377"
