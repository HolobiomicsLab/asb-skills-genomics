---
name: phenotype-disease-annotation-integration
description: Use when patient symptoms are encoded as HPO codes (e.g. HP:0002996,
  HP:0001377) and need to be converted into UMLS CUIs or ORDO disease terms to retrieve
  candidate disease genes for variant interpretation.
when_to_use_negative:
- Input phenotypes are not expressed as HPO codes (e.g. free text or phenotype names
  not first matched to HPO release v2018-03-08)
- No local TDB/HDT dataset is available and reliance on external web services is required,
  which VIBE explicitly avoids for privacy and availability reasons
- The goal is direct variant prioritization rather than phenotype-to-disease-concept
  annotation integration
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0625
tools:
- name: VIBE
  role: Executes the three-branch SPARQL HPO-to-CUI/ORDO mapping and computes the
    union of CUIs
  repo: https://github.com/molgenis/vibe
- name: Apache Jena
  role: Builds and queries the TDB triple store that integrates DisGeNET RDF, HOOM
    and SIO sources
  repo: https://github.com/molgenis/vibe
- name: TestNG
  role: Unit test suite used to validate the mapping module (compiled with Apache
    Maven)
  repo: https://github.com/molgenis/vibe
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:ff8d03a0cae275c2872035fde3fb47b0c28fb3c0e326ecc97a38dbc014bd3d7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Apache Jena
  - Apache Maven
  - Java 8
  - TestNG
  - VIBE
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# phenotype-disease-annotation-integration

## Summary

Maps Human Phenotype Ontology (HPO) patient-symptom codes onto disease concept identifiers (UMLS CUIs) by integrating phenotype-disease annotations from UMLS Metathesaurus, Orphadata HOOM (HPO-ORDO), and DisGeNET phenotype-disease associations, producing the per-HPO CUI sets needed to retrieve and rank gene-disease associations.

## When to use

Use when patient symptoms are encoded as HPO codes (e.g. HP:0002996, HP:0001377) and need to be converted into UMLS CUIs or ORDO disease terms to retrieve candidate disease genes for variant interpretation.

## When NOT to use

- Input phenotypes are not expressed as HPO codes (e.g. free text or phenotype names not first matched to HPO release v2018-03-08)
- No local TDB/HDT dataset is available and reliance on external web services is required, which VIBE explicitly avoids for privacy and availability reasons
- The goal is direct variant prioritization rather than phenotype-to-disease-concept annotation integration

## Inputs

- Patient symptoms encoded as HPO codes (e.g. HP:0002996, HP:0001377)
- TDB triple store of DisGeNET r6.0 RDF plus pda.ttl, phenotype.ttl, void.ttl (DisGeNET r5.0), SIO v1.43 and Orphadata HOOM r1.3
- Human Phenotype Ontology OWL file (hp.owl) when ontology traversal is enabled

## Outputs

- Per-HPO-code CUI sets written to a tab-delimited mapping file (hpo_to_cui_map.tsv)
- Union of CUIs used downstream to retrieve GDAs and produce a ranked gene list (results.tsv)

## How to apply

Load the prebuilt TDB triple store — DisGeNET r6.0 RDF supplemented with pda.ttl, phenotype.ttl and void.ttl from DisGeNET r5.0, SIO v1.43, and Orphadata HOOM r1.3, built with Apache Jena v3.12.0 — then read the sample HPO codes as search terms. Execute an optimized SPARQL query that branches into three paths: (a) HPO-to-CUI matching according to the UMLS Metathesaurus, (b) HPO-to-ORDO matching according to HOOM, and (c) HPO-to-disease matching according to DisGeNET PDAs. Compute the union of the CUIs returned by the three branches, since each source contributes phenotype-disease associations the others lack. Optionally expand the phenotypic search space using child or distance-based ontology traversal (-n with -m distance and an HPO OWL file supplied via -w). Write the per-HPO-code CUI sets to a tab-delimited mapping file; the union then drives retrieval of matching GDAs grouped by NCBI gene identifiers and ranked by highest GDA score.

## Related tools

- **VIBE** (Executes the three-branch SPARQL HPO-to-CUI/ORDO mapping and computes the union of CUIs) — https://github.com/molgenis/vibe
- **Apache Jena** (Builds and queries the TDB triple store that integrates DisGeNET RDF, HOOM and SIO sources) — https://github.com/molgenis/vibe
- **TestNG** (Unit test suite used to validate the mapping module (compiled with Apache Maven)) — https://github.com/molgenis/vibe

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -w hp.owl -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- Compare produced CUI sets against UMLS crosswalk references for the sampled HPO codes
- Confirm the TestNG v7.0.0 unit test suite passes (built with Apache Maven v3.3.9)
- Check that the union covers all three branches: UMLS Metathesaurus, HOOM ORDO matches, and DisGeNET PDA matches
- Verify output is a well-formed tab-delimited file with a CUI set for every supplied HPO code (e.g. HP:0002996, HP:0001377)
- Confirm the HPO codes used were validated against HPO release v2018-03-08

## Limitations

- Ontology traversal via -n/-m requires the HPO OWL file (-w) and has a documented caveat for the -n option (GitHub issue #25)
- VIBE shows a bias with over-represented genes (MECP2, KMT2D) at high ranking positions, to be resolved in upcoming versions
- Phenotype-gene matching in this pipeline only applies to SNVs and indels, excluding structural variation, CGH, Sanger sequencing, cytogenetics, RNA-sequencing and metabolomics

## Evidence

- [methods] VIBE first maps the HPO search terms to CUIs (concept unique identifiers) from UMLS (Unified Medical Language System) using a SPARQL query: [evidence span withheld — anchor status: not_found]
- [methods] three-branch mapping: "(a) HPO to CUI matching according to UMLS Metathesaurus, (b) HPO to ORDO matching according to HOOM, (c) HPO to the gene‐diseases associations according to DisGeNET PDAs"
- [methods] triple store construction: "All these sources are combined into a TDB triple store, which is built using Apache Jena (v3.12.0)"
- [methods] HOOM integration: [evidence span withheld — anchor status: not_found]
- [readme] CLI invocation: "java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377"
