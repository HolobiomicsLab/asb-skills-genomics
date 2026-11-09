---
name: gene-disease-association-scoring
description: Use when patient symptoms encoded as HPO codes (e.g. HP:0002996) must
  be converted into a ranked candidate gene list backed by literature/curated gene-disease
  associations, e.g. as a module in a genome diagnostics pipeline.
when_to_use_negative:
- The input lacks HPO-encoded patient phenotypes (the tool requires HPO codes as -p
  arguments)
- The task is variant-level filtering or molecular diagnosis beyond phenotype-based
  gene prioritization (VIBE only provides prioritized candidate genes, not variant
  selection)
- The analysis requires a web-service-based prioritizer or a tool whose strengths
  complement VIBE's ranking (e.g. AMELIE), since results show high complementarity
  among tools and external services may violate privacy/availability requirements
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: VIBE
  role: Stand-alone Java 8 command-line tool that performs the HPO-to-GDA matching
    and gene ranking by highest GDA score
  repo: https://github.com/molgenis/vibe
- name: DisGeNET-RDF
  role: Core data source providing gene-disease associations (GDAs) queried via SPARQL
- name: Apache Jena 3.12.0
  role: Builds the TDB triple store from which the minimized HDT dataset used by VIBE
    is derived
- name: vibe-suppl
  role: Supplementary Python/R scripts to merge per-case VIBE outputs and compute
    per-case causal-gene rankings for benchmarking
  repo: https://github.com/molgenis/vibe-suppl
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:d48c37a09407b2475b84b728dd39cd7944741be561c961bdae1624f89bfdf583
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Apache Jena 3.12.0
  - DisGeNET-RDF
  - HPO v2018-03-08
  - Java 8
  - VIBE
  - vibe-suppl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-disease-association-scoring

## Summary

Rank candidate disease genes for a patient case by matching Human Phenotype Ontology (HPO) codes to gene-disease associations (GDAs) from DisGeNET, scoring each NCBI gene identifier by its highest GDA score. This is the core prioritization step performed by VIBE, using literature-, animal model- and curated-source evidence.

## When to use

Use when patient symptoms encoded as HPO codes (e.g. HP:0002996) must be converted into a ranked candidate gene list backed by literature/curated gene-disease associations, e.g. as a module in a genome diagnostics pipeline.

## When NOT to use

- The input lacks HPO-encoded patient phenotypes (the tool requires HPO codes as -p arguments)
- The task is variant-level filtering or molecular diagnosis beyond phenotype-based gene prioritization (VIBE only provides prioritized candidate genes, not variant selection)
- The analysis requires a web-service-based prioritizer or a tool whose strengths complement VIBE's ranking (e.g. AMELIE), since results show high complementarity among tools and external services may violate privacy/availability requirements

## Inputs

- Patient case HPO phenotype codes (e.g. HP:0002996, HP:0001377) from HPO release v2018-03-08
- Local VIBE HDT knowledge-base archive (DisGeNET r6.0 RDF + r5.0 supplements + HOOM r1.3 + SIO v1.43)
- HPO.owl file (only required when ontology traversal via -w/-m/-n is used)
- CGD (Clinical Genomic Database) gene list when simulating a clinical exome interpretation

## Outputs

- Tab-delimited ranked gene list per case: NCBI gene identifier, HGNC gene symbol, highest GDA score, and pipe-separated UMLS disease associations with GDA scores and PubMed evidence IDs
- Per-case prioritized gene rankings against known causal genes (via vibe-suppl Python/R processing scripts)

## How to apply

Map the input HPO codes to UMLS CUIs via SPARQL against the unified knowledge base, and additionally via HPO-to-ORDO matching using Orphadata HOOM r1.3; take the union of resulting CUIs to retrieve all matching GDAs from the DisGeNET r6.0 RDF core (supplemented with DisGeNET r5.0 pda.ttl/phenotype.ttl/void.ttl and SIO v1.43). Group the GDAs by unique NCBI gene identifiers and rank genes in descending order of the highest GDA score. Run VIBE 2.0 (Java 8) as a stand-alone command-line tool supplying the HDT triple store with -t, the output file with -o, and one or more -p HP:XXXXXXX codes; at default settings no ontology traversal is used (no -w/-m/-n options), while -n children/distance with -m and an HPO OWL file (-w) can be supplied to increase the phenotypic search space. The default output is a tab-delimited file with NCBI gene ID, HGNC gene symbol, highest GDA score, and pipe-separated UMLS disease associations with GDA scores and PubMed evidence IDs.

## Related tools

- **VIBE** (Stand-alone Java 8 command-line tool that performs the HPO-to-GDA matching and gene ranking by highest GDA score) — https://github.com/molgenis/vibe
- **DisGeNET-RDF** (Core data source providing gene-disease associations (GDAs) queried via SPARQL)
- **Apache Jena 3.12.0** (Builds the TDB triple store from which the minimized HDT dataset used by VIBE is derived)
- **vibe-suppl** (Supplementary Python/R scripts to merge per-case VIBE outputs and compute per-case causal-gene rankings for benchmarking) — https://github.com/molgenis/vibe-suppl

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996
```

## Evaluation signals

- Each per-case output is a tab-delimited gene list whose columns are NCBI gene identifier, HGNC symbol, highest GDA score, and diseases (UMLS) with sources
- Genes are ranked in descending order of the highest GDA score and the known causal gene can be located within each ranking
- In the benchmark, total recall of all 308 causal genes is reached by rank 20 of the tool output under the clinical exome simulation (20 candidate genes, median of 25 permutations)
- All supplied -p HPO codes exist in HPO release v2018-03-08
- Re-running the 305 patient cases with default settings (no -w/-m/-n) reproduces the VIBE benchmark column in the vibe-suppl results

## Limitations

- VIBE lacks exceptionally well prioritized genes near the bottom of the rank distribution and shows a bias with over-represented genes (MECP2, KMT2D) at high positions
- The benchmark may be representative of clinical practice only to a degree and does not demonstrate individual strengths of the tested tools
- Using the -n ontology-traversal option carries known caveats documented in the molgenis/vibe issue #25 referenced by the README

## Evidence

- [methods] VIBE first maps the HPO search terms to CUIs: [evidence span withheld — anchor status: not_found]
- [methods] GDAs grouped by NCBI gene identifiers: [evidence span withheld — anchor status: not_found]
- [methods] core data source DisGeNET r6.0: [evidence span withheld — anchor status: not_found]
- [results] total recall by rank 20: "At rank 20, all tools arrive at their total recall of 308 causal genes."
- [readme] README quickstart invocation: "java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377"
