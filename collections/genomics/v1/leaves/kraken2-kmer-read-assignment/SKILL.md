---
name: kraken2-kmer-read-assignment
description: Use when the input is Nanopore long-read metagenomic FASTQ data and the
  goal is per-read taxonomic classification with a hierarchical, interactive visualization
  of community composition.
when_to_use_negative:
- The data are short reads or amplicon data rather than Nanopore long-read metagenomic
  samples — NanoGalaxy was built and intensively tested for ONT long-read data
- A taxonomic abundance table already exists and only visualization is needed (skip
  Kraken2, feed the table straight to Krona)
- A novel reference database must be built or a specific database version/version
  number must be certified — the paper does not state tool version numbers
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0091
tools:
- name: Kraken2
  role: Classifies each Nanopore long read via exact k-mer matches against a pre-indexed
    database, producing per-read assignments and a clade-ranked report
- name: Krona
  role: Consumes the Kraken2 report to build a hierarchical taxonomy table and render
    an interactive HTML pie chart of the metagenome composition
- name: Galaxy
  role: Provides the user-friendly interface, workflow chaining (Kraken2 → Krona),
    and dependency handling so the pipeline runs without coding
  repo: https://nanopore.usegalaxy.eu
- name: NanoGalaxy
  role: Galaxy toolkit distributing the metagenomics (Kraken2/Krona) workflow for
    Oxford Nanopore long-read data
  repo: mmiladi/galaxy-nanopore
- name: Conda
  role: Manages installation of Kraken2/Krona tool dependencies inside the Galaxy
    platform
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:12d4296ef251f746bf8a562c3a226dffecaa8f630cb2086fa70c55d7f3fc074e
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
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# kraken2-kmer-read-assignment

## Summary

Classify Oxford Nanopore long metagenomic reads taxonomically in NanoGalaxy using Kraken2 exact k-mer matching against a pre-indexed database, then render the clade-ranked report as an interactive Krona HTML pie chart.

## When to use

Use when the input is Nanopore long-read metagenomic FASTQ data and the goal is per-read taxonomic classification with a hierarchical, interactive visualization of community composition.

## When NOT to use

- The data are short reads or amplicon data rather than Nanopore long-read metagenomic samples — NanoGalaxy was built and intensively tested for ONT long-read data
- A taxonomic abundance table already exists and only visualization is needed (skip Kraken2, feed the table straight to Krona)
- A novel reference database must be built or a specific database version/version number must be certified — the paper does not state tool version numbers

## Inputs

- Nanopore long-read metagenomic FASTQ reads (e.g. from GigaDB)
- Pre-indexed Kraken2 k-mer database

## Outputs

- Kraken2 per-read taxonomic classification output
- Kraken2 clade-ranked taxonomic report
- Krona interactive HTML pie chart of taxonomic composition
- Exported Galaxy workflow definition (kraken2_krona_workflow.ga)

## How to apply

Upload the Nanopore metagenomic FASTQ reads (e.g. those deposited via GigaDB for this paper) into a Galaxy (NanoGalaxy) history, and attach a pre-built Kraken2 database as the k-mer reference. Run Kraken2 in Galaxy so each long read is classified by exact k-mer matches against the database, yielding both a per-read classification output and a clade-ranked report. Feed the Kraken2 report into the Krona tool, which builds the hierarchical taxonomy input table and renders an interactive HTML pie chart of the taxonomic composition. Save the connected Kraken2 → Krona steps as an exported Galaxy workflow definition (kraken2_krona_workflow.ga) with the FASTQ and database as workflow inputs and the Krona HTML as the output; all dependencies are handled by Conda-managed Galaxy wrappers. Rationale: NanoGalaxy packages these tools so users without programming experience can run command-line-grade metagenomics on long reads via the Galaxy platform.

## Related tools

- **Kraken2** (Classifies each Nanopore long read via exact k-mer matches against a pre-indexed database, producing per-read assignments and a clade-ranked report)
- **Krona** (Consumes the Kraken2 report to build a hierarchical taxonomy table and render an interactive HTML pie chart of the metagenome composition)
- **Galaxy** (Provides the user-friendly interface, workflow chaining (Kraken2 → Krona), and dependency handling so the pipeline runs without coding) — https://nanopore.usegalaxy.eu
- **NanoGalaxy** (Galaxy toolkit distributing the metagenomics (Kraken2/Krona) workflow for Oxford Nanopore long-read data) — mmiladi/galaxy-nanopore
- **Conda** (Manages installation of Kraken2/Krona tool dependencies inside the Galaxy platform)

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- The Kraken2 output contains a per-read classification line for each input FASTQ read (read count preserved between input and classification output)
- The Kraken2 clade-ranked report contains rank-coded clade rows whose read counts are consistent with the per-read output
- The Krona HTML renders an interactive pie chart whose hierarchical taxonomy matches the clade-ranked report
- The exported workflow (kraken2_krona_workflow.ga) exposes exactly two inputs (FASTQ and Kraken2 database) and one output (Krona HTML) and replays Kraken2 → Krona in order
- Tools resolved cleanly through the Conda-managed Galaxy installation (no missing-dependency errors in the job logs)

## Limitations

- No software version numbers are stated for the integrated tools in the publication
- Classification quality depends entirely on the pre-indexed Kraken2 database; only exact k-mer matches assign reads
- Support outside ONT data (e.g. Pacific Biosciences) is untested — the paper says such tools 'should also work' but intensive testing was done only for ONT data
- Toolkit extension by the community is expected but not yet realized, and future tools still need to be assembled into existing or new toolkits

## Evidence

- [abstract] The Galaxy platform provides a user-friendly interface to computational command line–based tools, handles the software dependencies, and provides refined workflows.: "The Galaxy platform provides a user-friendly interface to computational command line–based tools, handles the software dependencies, and provides refined workflows."
- [discussion] All tools and their dependencies are installed on the Galaxy platform and are managed by the Conda framework for dependency management.: "All tools and their dependencies are installed on the Galaxy platform and are managed by the Conda framework for dependency management"
- [readme] NanoGalaxy: Galaxy toolkit and workflows for long-read Oxford Nanopore data.: "NanoGalaxy: Galaxy toolkit and workflows for long-read Oxford Nanopore data"
- [intro] Here we introduce the NanoGalaxy toolkit for analysing Nanopore long-read data.: "Here we introduce the NanoGalaxy toolkit for analysing Nanopore long-read data."
