---
name: metagenomic-taxonomic-classification
description: Use when nanopore long-read metagenomic FASTQ reads need per-read taxonomic
  classification and an interactive visual summary of community composition in a Galaxy-based
  (NanoGalaxy) workflow.
when_to_use_negative:
- Input is genomic or plasmid sequence data intended for de novo genome assembly rather
  than metagenomic taxonomic profiling.
- Short-read (150-300 bp) sequencing data where a dedicated short-read classification
  pipeline is preferred — this skill is grounded in ONT long-read data that the integrated
  tools were intensively tested for.
- The user already has a Kraken2 report or feature/taxonomy table — in that case go
  straight to visualization, not re-classification.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: Kraken2
  role: Classifies each nanopore long read by exact k-mer matches against a pre-indexed
    database, producing per-read classifications and a clade-ranked report
- name: Krona
  role: Converts the Kraken2 clade-ranked report into a hierarchical taxonomy input
    table and renders an interactive HTML pie chart of taxonomic composition
- name: Galaxy
  role: User-friendly interface that wraps Kraken2 and Krona, handles software dependencies,
    and supports saving/exporting the connected Kraken2 -> Krona steps as a reusable
    workflow
  repo: https://toolshed.g2.bx.psu.edu/
- name: NanoGalaxy
  role: Galaxy-based toolkit providing the metagenomics workflow environment and the
    pre-configured Kraken2/Krona tools for long-read data
  repo: mmiladi/galaxy-nanopore
- name: Conda
  role: Dependency framework managing installation of the Kraken2 and Krona tools
    and their dependencies on the Galaxy platform
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:2fea6a20cc9d6de0be474c14b3490b5ae0ce34ec6fa53b85167581bb94602d54
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Conda
  - Galaxy
  - Kraken2
  - Krona
  - NanoGalaxy
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# metagenomic-taxonomic-classification

## Summary

Classifies nanopore long-read metagenomic FASTQ reads taxonomically using Kraken2 (exact k-mer matches against a pre-indexed database) within NanoGalaxy/Galaxy, then renders the clade-ranked report as an interactive Krona HTML pie chart so users can inspect sample composition without coding.

## When to use

Use when nanopore long-read metagenomic FASTQ reads need per-read taxonomic classification and an interactive visual summary of community composition in a Galaxy-based (NanoGalaxy) workflow.

## When NOT to use

- Input is genomic or plasmid sequence data intended for de novo genome assembly rather than metagenomic taxonomic profiling.
- Short-read (150-300 bp) sequencing data where a dedicated short-read classification pipeline is preferred — this skill is grounded in ONT long-read data that the integrated tools were intensively tested for.
- The user already has a Kraken2 report or feature/taxonomy table — in that case go straight to visualization, not re-classification.

## Inputs

- Nanopore long-read metagenomic FASTQ reads (e.g. from GigaDB)
- Pre-indexed Kraken2 database (k-mer reference for taxonomic classification)

## Outputs

- Kraken2 per-read classification output
- Kraken2 clade-ranked report
- Interactive Krona HTML pie chart of taxonomic composition
- Exported Galaxy workflow definition (kraken2_krona_workflow.ga)

## How to apply

Upload the Nanopore metagenomic FASTQ reads (deposited via GigaDB) into a Galaxy (NanoGalaxy) history and attach a pre-indexed Kraken2 database as the reference. Run Kraken2 so each long read is classified by exact k-mer matches against the database, producing a per-read classification output and a clade-ranked report. Feed the Kraken2 report into the Krona tool to build the hierarchical taxonomy input table, then render an interactive HTML pie chart of the taxonomic composition. Connect Kraken2 -> Krona in an exported Galaxy workflow definition (kraken2_krona_workflow.ga) with the FASTQ and database as workflow inputs and the Krona HTML as the workflow output, so the classification is reproducible without command-line skills. Rationale: NanoGalaxy installs tools and manages dependencies via the Conda framework, allowing non-bioinformatics-trained researchers to run the analysis entirely through the Galaxy interface.

## Related tools

- **Kraken2** (Classifies each nanopore long read by exact k-mer matches against a pre-indexed database, producing per-read classifications and a clade-ranked report)
- **Krona** (Converts the Kraken2 clade-ranked report into a hierarchical taxonomy input table and renders an interactive HTML pie chart of taxonomic composition)
- **Galaxy** (User-friendly interface that wraps Kraken2 and Krona, handles software dependencies, and supports saving/exporting the connected Kraken2 -> Krona steps as a reusable workflow) — https://toolshed.g2.bx.psu.edu/
- **NanoGalaxy** (Galaxy-based toolkit providing the metagenomics workflow environment and the pre-configured Kraken2/Krona tools for long-read data) — mmiladi/galaxy-nanopore
- **Conda** (Dependency framework managing installation of the Kraken2 and Krona tools and their dependencies on the Galaxy platform)

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- Kraken2 produces both a per-read classification output and a clade-ranked report — absence of either indicates a misconfigured run.
- The Krona tool successfully builds a hierarchical taxonomy input table from the Kraken2 report and renders an interactive HTML pie chart that opens correctly in a browser.
- The exported workflow (kraken2_krona_workflow.ga) contains the connected Kraken2 -> Krona steps with the FASTQ and database as workflow inputs and the Krona HTML as the single workflow output.
- Tool dependencies resolve through the Conda framework on the Galaxy platform, and wrappers install from the Galaxy Tool Shed (https://toolshed.g2.bx.psu.edu/).
- Re-running the workflow on the same GigaDB-deposited FASTQ and database reproduces the same taxonomic composition in the Krona chart.

## Limitations

- NanoGalaxy focused on the most established and widely used tools; other long-read metagenomic analysis tools that exist or are under development were not integrated.
- Support for Pacific Biosciences tools inside Galaxy is untested ('should also work'); the integrated tools were intensively tested for ONT data only.
- No software version numbers were stated for the integrated tools in the paper's discussion, and no changelog was found.
- Toolkit extension by the community is expected but not yet realized, so the workflow surface may change.

## Evidence

- [other] Kraken2 classifies long reads against a pre-indexed database, followed by post-processing and interactive pie-chart visualization of the results using the Krona tool.: [evidence span withheld — anchor status: not_found]
- [other] Run Kraken2 in Galaxy to classify each long read by exact k-mer matches against the database: [evidence span withheld — anchor status: not_found]
- [abstract] The Galaxy platform provides a user-friendly interface to computational command line–based tools, handles the software dependencies, and provides refined workflows.: "The Galaxy platform provides a user-friendly interface to computational command line–based tools, handles the software dependencies, and provides refined workflows."
- [discussion] presented solutions for metagenomic analysis and genome assembly: "presented solutions for metagenomic analysis and genome assembly"
- [discussion] All tools and their dependencies are installed on the Galaxy platform and are managed by the Conda framework for dependency management: "All tools and their dependencies are installed on the Galaxy platform and are managed by the Conda framework for dependency management"
- [readme] docker run -d -p 8080:80 mmiladi/galaxy-nanopore: "``docker run -d -p 8080:80 mmiladi/galaxy-nanopore``"
