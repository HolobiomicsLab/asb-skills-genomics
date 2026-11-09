---
name: tabular-output-formatting-for-genome-diagnostics
description: 'Use when producing or validating a VIBE-style ranked gene output: a
  tab-delimited file ordered by descending highest GDA score carrying gene (NCBI),
  gene symbol (HGNC) and diseases (UMLS) columns.'
when_to_use_negative:
- The input is a ranked gene list from another prioritization tool (e.g., Phenomizer,
  hiPHIVE, GADO benchmark output) that must first be converted from HGNC symbols to
  NCBI gene IDs via the BenchmarkFileGeneSymbolToIdConverter workflow rather than
  re-formatted by VIBE.
- The consumer needs an interactive user interface rather than a file artifact; use
  the molgenis-app-vibe web application instead of the tabular command-line output.
edam_operation: http://edamontology.org/operation_0372
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_3517
tools:
- name: VIBE
  role: Java 8 command-line tool whose -o flag writes the ranked tab-delimited output
    (results.tsv) with the highest GDA score and diseases (UMLS) columns
  repo: https://github.com/molgenis/vibe
- name: Apache Jena
  role: Builds the TDB triple store queried with SPARQL to retrieve the GDAs that
    populate the ranked gene output
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:1f9dd29f8df2d6effd9c5629fc07c1c6c21358b5b6b2dd6c95e0c51111949130
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - Apache Jena
  - Java 8
  - VIBE
  edam_operation: http://edamontology.org/operation_0372
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# tabular-output-formatting-for-genome-diagnostics

## Summary

Emits VIBE's prioritized gene list as a tab-delimited results file (e.g., results.tsv) in which genes are ordered by descending highest GDA score, so downstream pipeline modules and human curators can consume the ranked candidates reproducibly.

## When to use

Use when producing or validating a VIBE-style ranked gene output: a tab-delimited file ordered by descending highest GDA score carrying gene (NCBI), gene symbol (HGNC) and diseases (UMLS) columns.

## When NOT to use

- The input is a ranked gene list from another prioritization tool (e.g., Phenomizer, hiPHIVE, GADO benchmark output) that must first be converted from HGNC symbols to NCBI gene IDs via the BenchmarkFileGeneSymbolToIdConverter workflow rather than re-formatted by VIBE.
- The consumer needs an interactive user interface rather than a file artifact; use the molgenis-app-vibe web application instead of the tabular command-line output.

## Inputs

- VIBE HDT database file (vibe-<db-version>.hdt) containing DisGeNET-derived GDAs
- HPO term IDs supplied via repeated -p flags (e.g., HP:0002996, HP:0001377)
- Human Phenotype Ontology OWL file (hp.owl) supplied via -w when using ontology traversal options
- Union of CUIs produced by the upstream HPO-to-CUI mapping step

## Outputs

- Tab-delimited results file (results.tsv) with columns: gene (NCBI), gene symbol (HGNC), highest GDA score, diseases (UMLS) with sources
- Genes ordered in descending priority by highest GDA score, with GDA scores and PubMed identifiers grouped per CUI matched to each gene

## How to apply

After SPARQL retrieval of GDAs against the triple store using the union of CUIs mapped from the HPO search terms, group the retrieved GDAs by unique NCBI gene identifier and assign each gene the highest GDA score within its group as its priority. Write the result as a tab-delimited file (VIBE's -o option, e.g., results.tsv) whose default header columns are 'gene (NCBI)', 'gene symbol (HGNC)', 'highest GDA score', and 'diseases (UMLS) with sources', listing genes in descending order of highest GDA score. Accompany each gene's highest score with all GDA scores and the PubMed identifiers of supporting literature grouped per CUI matched to that gene. VIBE currently supports 2 output format options, so choose the default tabular layout unless the pipeline explicitly expects the alternative. Run under Java 8 or higher with the pre-built VIBE HDT database supplied via -t (and hp.owl via -w when ontology traversal options such as -n/-m are used).

## Related tools

- **VIBE** (Java 8 command-line tool whose -o flag writes the ranked tab-delimited output (results.tsv) with the highest GDA score and diseases (UMLS) columns) — https://github.com/molgenis/vibe
- **Apache Jena** (Builds the TDB triple store queried with SPARQL to retrieve the GDAs that populate the ranked gene output)

## Examples

```
java -jar vibe-with-dependencies-5.0.0.jar -d -t vibe-5.0.0.hdt -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- The output file is tab-delimited and its header contains the columns 'gene (NCBI)', 'gene symbol (HGNC)', 'highest GDA score', and 'diseases (UMLS) with sources'.
- Genes are ordered by descending highest GDA score (descending priority).
- Each gene appears once per unique NCBI gene identifier, annotated with its highest GDA score within its group of GDAs.
- Diseases listed for a gene are UMLS CUIs matched to that gene, with supporting PubMed identifiers grouped per CUI.
- The file lands at the path given by the -o flag (e.g., results.tsv) when the run completes without error.

## Limitations

- Only 2 output format options are currently supported by VIBE, so pipelines expecting a different schema must post-process the default tab-delimited layout.
- Ontology traversal options (especially -n) carry documented caveats that the README flags as IMPORTANT to keep in mind (VIBE issue #25).
- VIBE shows a bias with over-represented genes (e.g., MECP2, KMT2D) at high positions, and the causal gene can be entirely absent (missed) from the output gene list.

## Evidence

- [methods] A union of resulting CUIs is then used to retrieve all matching GDAs (Gene‐Disease Associations). The GDAs are grouped by unique NCBI gene identifiers: [evidence span withheld — anchor status: not_found]
- [readme] There are currently 2 options for the output. By default the output will look something like:: "There are currently 2 options for the output. By default the output will look something like:"
- [readme] **IMPORTANT:** Do keep  [this](https://github.com/molgenis/vibe/issues/25) in mind. Especially when using `-n`.: "**IMPORTANT:** Do keep  [this](https://github.com/molgenis/vibe/issues/25) in mind. Especially when using `-n`."
- [results] Figure 1 shows the number of returned genes vs the rank of the causal gene if found within the output gene list.: "Figure 1 shows the number of returned genes vs the rank of the causal gene if found within the output gene list."
