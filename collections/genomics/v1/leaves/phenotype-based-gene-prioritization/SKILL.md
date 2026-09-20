---
name: phenotype-based-gene-prioritization
description: Use when a patient case or benchmark case is described as HPO-coded phenotypes
  (e.g. HP:0002996) and a ranked candidate gene list is needed for genome diagnostics
  or pipeline integration.
when_to_use_negative:
- The molecular question involves structural variation, CGH, Sanger sequencing, cytogenetics,
  RNA-sequencing or metabolomics rather than phenotype-to-gene matching, which VIBE
  does not address
- No HPO-coded phenotype description of the case exists (only raw variant calls),
  since VIBE requires HPO search terms as input
- Privacy or availability rules forbid a local tool's dependencies being fetched at
  run time — VIBE is designed for local installation precisely to avoid external services
  such as AMELIE
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: VIBE
  role: Stand-alone command-line tool that performs HPO-to-CUI mapping, GDA retrieval
    and gene ranking by highest GDA score
  repo: https://github.com/molgenis/vibe
- name: Java 8
  role: Runtime used to invoke the vibe-with-dependencies JAR
- name: Apache Jena 3.12.0
  role: Builds the TDB triple store from combined DisGeNET-RDF, HOOM, SIO and HPO
    sources underlying VIBE
- name: HPO v2018-03-08
  role: Ontology file (supplied via -w) required when using -m/-n ontology traversal
    to increase the phenotypic search space
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:cab1701785d359e2391f83f9cfaecd8b88131843c6f1cdd4fd19a6d4748c3156
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Apache Jena 3.12.0
  - HPO v2018-03-08
  - Java 8
  - Python
  - R
  - VIBE
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# phenotype-based-gene-prioritization

## Summary

Prioritize candidate disease genes by matching patient symptoms encoded as Human Phenotype Ontology (HPO) codes to gene-disease associations from literature, animal models and curated sources, using the stand-alone VIBE command-line tool built on DisGeNET-RDF.

## When to use

Use when a patient case or benchmark case is described as HPO-coded phenotypes (e.g. HP:0002996) and a ranked candidate gene list is needed for genome diagnostics or pipeline integration.

## When NOT to use

- The molecular question involves structural variation, CGH, Sanger sequencing, cytogenetics, RNA-sequencing or metabolomics rather than phenotype-to-gene matching, which VIBE does not address
- No HPO-coded phenotype description of the case exists (only raw variant calls), since VIBE requires HPO search terms as input
- Privacy or availability rules forbid a local tool's dependencies being fetched at run time — VIBE is designed for local installation precisely to avoid external services such as AMELIE

## Inputs

- HPO-coded patient phenotypes (e.g. HP:0002996, HP:0001377)
- Local HDT knowledge-base archive from molgenis/vibe releases
- HPO.owl ontology file (release v2018-03-08) when ontology traversal options are used
- vibe-with-dependencies JAR executable

## Outputs

- Tab-delimited prioritized gene list per case (NCBI gene ID, HGNC symbol, highest GDA score, UMLS disease associations with GDA scores and PubMed IDs)
- Per-case ranked gene lists for benchmark evaluation against known causal genes

## How to apply

Run VIBE (v2.0, Java 8) against a local HDT triple store compiled from DisGeNET r6.0/RDF r5.0, HOOM r1.3, SIO v1.43 and HPO v2018-03-08 via Apache Jena 3.12.0, supplying each case's HPO codes with -p (repeatable). VIBE first maps HPO search terms to UMLS CUIs with a SPARQL query, matches HPO to ORDO via HOOM and to DisGeNET PDAs, then retrieves the union of matching GDAs grouped by unique NCBI gene identifiers and ranked by highest GDA score. At default settings no ontology traversal is used (omit -m/-n; if expanding the phenotypic search space, supply an HPO OWL file via -w). Output is a tab-delimited file with NCBI gene identifier, HGNC gene symbol, highest GDA score, and pipe-separated UMLS disease associations with GDA scores and PubMed evidence IDs, sorted by descending GDA score.

## Related tools

- **VIBE** (Stand-alone command-line tool that performs HPO-to-CUI mapping, GDA retrieval and gene ranking by highest GDA score) — https://github.com/molgenis/vibe
- **Java 8** (Runtime used to invoke the vibe-with-dependencies JAR)
- **Apache Jena 3.12.0** (Builds the TDB triple store from combined DisGeNET-RDF, HOOM, SIO and HPO sources underlying VIBE)
- **HPO v2018-03-08** (Ontology file (supplied via -w) required when using -m/-n ontology traversal to increase the phenotypic search space)

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- Each per-case output is a tab-delimited gene list ranked in descending order of highest GDA score keyed by NCBI gene identifier
- In the 305-patient-case benchmark, total recall of all 308 causal genes is reached by rank 20 of the output gene list
- Disease associations are emitted as pipe-separated UMLS CUIs with GDA scores and PubMed evidence IDs
- Top-10 occurrence counting across cases reproduces the reported over-representation of MECP2 (~226/305) and KMT2D (~219/305)
- No ontology traversal options (-w/-m/-n) are passed when running at default settings

## Limitations

- VIBE shows a bias with over-represented genes (MECP2, KMT2D) at high positions of the ranked list, which the authors aim to resolve in upcoming versions
- VIBE lacks exceptionally well prioritized genes near the bottom of the rank distribution
- Using -n ontology traversal has known caveats (molgenis/vibe issue #25) and requires an HPO OWL file via -w
- The 305-case benchmark may be representative of clinical practice only to a degree and does not demonstrate individual strengths of the tested tools; results show high complementarity with Phenomizer, PhenoTips, hiPHIVE, PhenIX, PubCaseFinder, AMELIE and GADO

## Evidence

- [abstract] We present Variant Interpretation using Biomedical literature Evidence (VIBE), a tool to prioritize disease genes based on Human Phenotype Ontology codes.: "We present Variant Interpretation using Biomedical literature Evidence (VIBE), a tool to prioritize disease genes based on Human Phenotype Ontology codes."
- [methods] HPO to CUI mapping via SPARQL: [evidence span withheld — anchor status: not_found]
- [methods] GDA retrieval and gene ranking: [evidence span withheld — anchor status: not_found]
- [methods] Benchmark design and clinical exome simulation: "For each of the 308 causal genes from the patient cases, we selected 19 other pseudorandom genes from CGD and spiked in the causal gene"
- [results] Recall at rank 20: "At rank 20, all tools arrive at their total recall of 308 causal genes."
- [readme] Quickstart invocation from README: "java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377"
