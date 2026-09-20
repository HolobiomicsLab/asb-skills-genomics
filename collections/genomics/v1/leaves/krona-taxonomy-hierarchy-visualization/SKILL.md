---
name: krona-taxonomy-hierarchy-visualization
description: Use when nanopore long-read metagenomic FASTQ reads have been classified
  with Kraken2 against a pre-indexed database and an interactive, hierarchical visualization
  of the taxonomic composition is required.
when_to_use_negative:
- Input is already a Krona/Kraken taxonomy table or an interactive HTML visualization
  rather than raw classified reads.
- Data are short-read (150-300 bp) metagenomic reads rather than nanopore long reads
  — NanoGalaxy was designed and intensively tested for ONT long-read data.
- The analysis goal is de novo genome assembly or AMR analysis rather than taxonomic
  classification and composition visualization.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0092
tools:
- name: Galaxy
  role: Platform hosting the Kraken2 and Krona tools with a user-friendly interface
    that handles software dependencies and workflows
  repo: https://nanopore.usegalaxy.eu
- name: NanoGalaxy
  role: Galaxy-based toolkit providing the metagenomics workflow for nanopore long-read
    data including Kraken2/Krona
  repo: https://github.com/mmiladi/galaxy-nanopore
- name: Kraken2
  role: Classifies each long read by exact k-mer matches against a pre-indexed database,
    producing the per-read classification and clade-ranked report consumed by Krona
- name: Krona
  role: Builds the hierarchical taxonomy table from the Kraken2 report and renders
    the interactive HTML pie chart of taxonomic composition
- name: Conda
  role: Dependency management framework under which all tools and their dependencies
    are installed on the Galaxy platform
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:456b3edbc2e6710ac46e824b9326f92c477942aaa0ba2407aef2f6ea319ba838
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Conda
  - Galaxy
  - Kraken2
  - Krona
  - NanoGalaxy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# krona-taxonomy-hierarchy-visualization

## Summary

Render an interactive HTML pie-chart visualization of Kraken2 taxonomic classification results for nanopore long-read metagenomics data using the Krona tool inside Galaxy (NanoGalaxy), so users can explore the hierarchical composition without coding skills.

## When to use

Use when nanopore long-read metagenomic FASTQ reads have been classified with Kraken2 against a pre-indexed database and an interactive, hierarchical visualization of the taxonomic composition is required.

## When NOT to use

- Input is already a Krona/Kraken taxonomy table or an interactive HTML visualization rather than raw classified reads.
- Data are short-read (150-300 bp) metagenomic reads rather than nanopore long reads — NanoGalaxy was designed and intensively tested for ONT long-read data.
- The analysis goal is de novo genome assembly or AMR analysis rather than taxonomic classification and composition visualization.

## Inputs

- Nanopore long-read metagenomic FASTQ reads (deposited via GigaDB)
- Pre-indexed Kraken2 reference database
- Kraken2 clade-ranked taxonomic report

## Outputs

- Interactive HTML pie chart of taxonomic composition (Krona output)
- Per-read Kraken2 classification output
- Clade-ranked Kraken2 report
- Exported Galaxy workflow definition (kraken2_krona_workflow.ga)

## How to apply

In NanoGalaxy, first run Kraken2 on the uploaded Nanopore metagenomic FASTQ reads using a pre-indexed Kraken2 database, which classifies each long read by exact k-mer matches and produces both a per-read classification output and a clade-ranked report. Feed the Kraken2 report into the Krona Galaxy tool to build the hierarchical taxonomy input table. Run Krona to render an interactive HTML pie chart of the taxonomic composition. Connect the Kraken2 and Krona steps into an exported Galaxy workflow (kraken2_krona_workflow.ga) with the FASTQ file and Kraken2 database as workflow inputs and the Krona HTML as the workflow output. The rationale is that NanoGalaxy wraps all tools and dependencies via Conda, so no command-line or programming experience is needed by the analyst.

## Related tools

- **Galaxy** (Platform hosting the Kraken2 and Krona tools with a user-friendly interface that handles software dependencies and workflows) — https://nanopore.usegalaxy.eu
- **NanoGalaxy** (Galaxy-based toolkit providing the metagenomics workflow for nanopore long-read data including Kraken2/Krona) — https://github.com/mmiladi/galaxy-nanopore
- **Kraken2** (Classifies each long read by exact k-mer matches against a pre-indexed database, producing the per-read classification and clade-ranked report consumed by Krona)
- **Krona** (Builds the hierarchical taxonomy table from the Kraken2 report and renders the interactive HTML pie chart of taxonomic composition)
- **Conda** (Dependency management framework under which all tools and their dependencies are installed on the Galaxy platform)

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- Kraken2 produces two artifacts: a per-read classification output and a clade-ranked report; both must exist before Krona runs.
- The Krona output must be a self-contained interactive HTML file rendering a pie chart of the taxonomic composition.
- The exported kraken2_krona_workflow.ga must list the FASTQ file and Kraken2 database as workflow inputs and the Krona HTML as the workflow output.
- The dominant taxa shown in the Krona chart should be consistent with the top-ranked clades in the Kraken2 report.
- Galaxy history should show the connected Kraken2 -> Krona step chain with all tool dependencies resolved via Conda.

## Limitations

- Kraken2 classification depends entirely on the completeness and relevance of the pre-indexed database; reads absent from the database remain unclassified.
- The workflow was designed and tested for ONT nanopore long reads; Pacific Biosciences tools in Galaxy are untested ('should also work').
- The article does not state software version numbers for the integrated tools in this section; tools are managed via Conda.

## Evidence

- [other] NanoGalaxy provides a metagenomics workflow in which Kraken2 classifies long reads against a pre-indexed database, followed by post-processing and interactive pie-chart visualization of the results using the Krona tool.: [evidence span withheld — anchor status: not_found]
- [abstract] Galaxy platform handles dependencies: "The Galaxy platform provides a user-friendly interface to computational command line–based tools, handles the software dependencies, and provides refined workflows."
- [discussion] Tools managed by Conda: "All tools and their dependencies are installed on the Galaxy platform and are managed by the Conda framework for dependency management"
- [abstract] No programming experience needed: "The users do not have to possess programming experience or extended computer skills."
- [readme] Docker launch of NanoGalaxy: "docker run -d -p 8080:80 mmiladi/galaxy-nanopore"
