---
name: command-line-tool-execution
description: Use when a task requires running a local stand-alone CLI tool (e.g. VIBE)
  on HPO term inputs to generate ranked gene prioritizations, or when benchmarking
  such a tool across many patient cases in parallel.
when_to_use_negative:
- When privacy or availability rules forbid running an external web service — this
  skill is specifically for local stand-alone execution, but conversely if a graphical/web
  interface is needed, use the molgenis-app-vibe web app rather than the CLI
- When the goal requires ontology traversal (child or distance-based phenotype expansion)
  — that changes the invocation with -w/-m/-n options and falls outside the default
  benchmark configuration
- When input phenotypes are not HPO terms (e.g. free-text symptoms); benchmark_data.tsv
  required phenotype names that exist within the Human Phenotype Ontology (2018-03-08
  release)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: VIBE
  role: The stand-alone Java 8 command-line tool executed per patient case to produce
    ranked gene prioritizations from HPO codes against an HDT knowledge base
  repo: https://github.com/molgenis/vibe
- name: molgenis/vibe-suppl
  role: Supplemental scripts (VibeBenchmarkParallelBashScriptsGenerator.py, VibeBenchmarkFileGenerator.py)
    used to generate parallel benchmark runs and merge per-case VIBE outputs
  repo: https://github.com/molgenis/vibe-suppl
- name: molgenis/molgenis-app-vibe
  role: Web application interface for VIBE, an alternative to the command-line execution
    path for interactive use
  repo: https://github.com/molgenis/molgenis-app-vibe
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ggn2.10023
    title: A pipeline‐friendly software tool for genome diagnostics to prioritize
      genes by matching patient symptoms to literature
  source_content_hash: sha256:ab96963b6ed78586c64d112620c295a7e78d00a7c799619ade1a5c3b2b1efe9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/molgenis/vibe
  tools:
  - HPO v2018-03-08
  - Java 8
  - VIBE
  - molgenis/molgenis-app-vibe
  - molgenis/vibe-suppl
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# command-line-tool-execution

## Summary

Execute a stand-alone Java command-line prioritization tool (VIBE v2.0) locally against an HDT knowledge base to produce ranked gene lists per patient case, enabling pipeline-friendly, offline phenotype-based gene prioritization without external web services.

## When to use

Use when a task requires running a local stand-alone CLI tool (e.g. VIBE) on HPO term inputs to generate ranked gene prioritizations, or when benchmarking such a tool across many patient cases in parallel.

## When NOT to use

- When privacy or availability rules forbid running an external web service — this skill is specifically for local stand-alone execution, but conversely if a graphical/web interface is needed, use the molgenis-app-vibe web app rather than the CLI
- When the goal requires ontology traversal (child or distance-based phenotype expansion) — that changes the invocation with -w/-m/-n options and falls outside the default benchmark configuration
- When input phenotypes are not HPO terms (e.g. free-text symptoms); benchmark_data.tsv required phenotype names that exist within the Human Phenotype Ontology (2018-03-08 release)

## Inputs

- HPO term codes (e.g. HP:0002996, HP:0001377) per patient case
- vibe-with-dependencies-<version>.jar executable
- Local HDT dataset archive (e.g. vibe-<db-version>.hdt, with sha256 checksum)
- HPO.owl ontology file (release v2018-03-08 for the benchmark)
- benchmark_data.tsv with case IDs and comma-separated phenotype names

## Outputs

- Tab-delimited ranked gene list per case: NCBI gene ID, HGNC gene symbol, highest GDA score, and pipe-separated disease (UMLS) associations with GDA scores and PubMed evidence IDs
- Merged vibe_results.tsv across all cases via VibeBenchmarkFileGenerator.py
- Per-case priority rankings used to locate the known causal gene (308 causal genes total)

## How to apply

Download the vibe-with-dependencies jar, the HDT database archive (with sha256 checksum), and HPO.owl from the molgenis/vibe GitHub releases, and confirm Java 8 or higher is installed. For each patient case, invoke VIBE via `java -jar vibe-with-dependencies-<version>.jar` passing `-t` for the HDT triple store, `-o` for the output TSV, and one or more `-p HP:XXXXXXX` HPO codes; the benchmark runs at default settings without ontology traversal (no -w/-m/-n options). Parse the tab-delimited output containing the NCBI gene identifier, HGNC gene symbol, highest GDA score, and pipe-separated UMLS disease associations with GDA scores and PubMed evidence IDs. For large-scale benchmarking (305 patient cases, 308 causal genes), generate parallel bash scripts with VibeBenchmarkParallelBashScriptsGenerator.py, duplicate the TDB store per script (TDB0/, TDB1/, ...), then merge results with VibeBenchmarkFileGenerator.py. Validate by confirming each output is a gene list ranked in descending order of highest GDA score and that the known causal gene is locatable within the ranking.

## Related tools

- **VIBE** (The stand-alone Java 8 command-line tool executed per patient case to produce ranked gene prioritizations from HPO codes against an HDT knowledge base) — https://github.com/molgenis/vibe
- **molgenis/vibe-suppl** (Supplemental scripts (VibeBenchmarkParallelBashScriptsGenerator.py, VibeBenchmarkFileGenerator.py) used to generate parallel benchmark runs and merge per-case VIBE outputs) — https://github.com/molgenis/vibe-suppl
- **molgenis/molgenis-app-vibe** (Web application interface for VIBE, an alternative to the command-line execution path for interactive use) — https://github.com/molgenis/molgenis-app-vibe

## Examples

```
java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377
```

## Evaluation signals

- Each per-case output is a tab-delimited gene list ranked in descending order of highest GDA score
- The known causal gene can be located within each per-case ranking (the paper expects total recall of all 308 causal genes by rank 20 of the tool output)
- HDT archive integrity is confirmed via the published sha256 checksum (database/checksums/database.sha256)
- HPO codes passed via -p are valid codes resolvable in the HPO release used (v2018-03-08 for the benchmark)
- Merged benchmark output converts successfully to NCBI gene IDs via BenchmarkFileGeneSymbolToIdConverter.py with no gene-symbol conversion errors

## Limitations

- VIBE lacks exceptionally well prioritized genes near the bottom of the rank distribution compared with other tools
- VIBE shows a bias with over-represented genes (e.g. MECP2, KMT2D) at high positions, which the authors aim to resolve in upcoming versions
- Ontology traversal options (-n, -m, -w) carry documented caveats: users are warned to keep a specific GitHub issue (issue #25) in mind, especially when using -n
- The benchmark may be representative of clinical practice only to a degree and does not demonstrate individual strengths of the tested tools

## Evidence

- [methods] VIBE (version 2.0) was programmed in Java 8, using DisGeNET as its main data source: [evidence span withheld — anchor status: not_found]
- [readme] Open a terminal and run VIBE.: "java -jar vibe-with-dependencies-<version>.jar -d -t vibe-<db-version>.hdt -o results.tsv -p HP:0002996 -p HP:0001377"
- [methods] Ranking by highest GDA score: [evidence span withheld — anchor status: not_found]
- [intro] Standalone local execution rationale: "this software is available as a stand‐alone command‐line executable which trivializes local integration into bioinformatic pipelines"
- [readme] VIBE benchmark workflow in supplemental repo: [evidence span withheld — anchor status: not_found]
