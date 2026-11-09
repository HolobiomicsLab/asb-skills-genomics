---
name: reference-plasmid-identity-comparison
description: Use when validating that a long-read (Oxford Nanopore) plasmid assembly
  pipeline recovers known plasmids from a reference set, or when benchmarking an assembly
  against published plasmid recovery results.
when_to_use_negative:
- The input is short-read (150-300 bp) sequencing data rather than Nanopore long reads
  — the skill and its underlying tools were intensively tested only for ONT data
- The goal is de novo plasmid discovery without any reference plasmid set to compare
  against
- No polished assembly is available yet — run minimap2/miniasm and Racon first before
  the identity comparison
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Galaxy
  role: User-friendly platform hosting the workflow and handling software dependencies
- name: NanoGalaxy
  role: Galaxy-based toolkit executing the long-read plasmid assembly and analysis
    workflow
  repo: mmiladi/galaxy-nanopore
- name: minimap2
  role: Computes read-vs-read overlaps to build the draft plasmid assembly
- name: miniasm
  role: Builds the draft de novo assembly from plasmid-containing reads
- name: Racon
  role: Polishes the draft assembly to correct nanopore read errors before comparison
- name: PlasFlow
  role: Classifies assembled contigs as plasmid-derived
- name: Bandage
  role: Visual inspection/confirmation of the plasmid contigs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:70c845b2d4d2c0535ed71885f71e517dfaaef15be5a1fe4ed61451336f5e403b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mmiladi/galaxy-nanopore
  tools:
  - Bandage
  - Galaxy
  - NanoGalaxy
  - PlasFlow
  - Racon
  - miniasm
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# Reference Plasmid Identity Comparison

## Summary

Compares de novo-assembled Nanopore plasmid contigs against a published reference plasmid set, recording the number of plasmids recovered and their mean sequence identity (e.g. the Li et al. benchmark: 19 of 21 plasmids at 97.76% mean identity) to validate a long-read assembly workflow.

## When to use

Use when validating that a long-read (Oxford Nanopore) plasmid assembly pipeline recovers known plasmids from a reference set, or when benchmarking an assembly against published plasmid recovery results.

## When NOT to use

- The input is short-read (150-300 bp) sequencing data rather than Nanopore long reads — the skill and its underlying tools were intensively tested only for ONT data
- The goal is de novo plasmid discovery without any reference plasmid set to compare against
- No polished assembly is available yet — run minimap2/miniasm and Racon first before the identity comparison

## Inputs

- Nanopore long-read sequencing FASTQ files containing plasmid reads
- Reference plasmid sequence set (e.g. Li et al. plasmid references)
- Polished assembly contigs (FASTA) from minimap2/miniasm + Racon

## Outputs

- plasmid_recovery_results.csv recording number of plasmids recovered and mean sequence identity
- PlasFlow-classified plasmid contigs
- Bandage visualizations confirming plasmid contigs

## How to apply

Run the NanoGalaxy long-read workflow: upload the Nanopore long-read FASTQ data, compute read-vs-read overlaps and build a draft assembly with minimap2 and miniasm, then polish the draft with Racon to correct nanopore read errors. Classify assembled contigs as plasmid-derived using PlasFlow and visually inspect/confirm the plasmid contigs with Bandage. Align the recovered plasmid sequences against the published reference plasmid set (e.g. the Li et al. set), then record two metrics into plasmid_recovery_results.csv: the count of plasmids recovered (target: 19 of 21) and the mean sequence identity (target: 97.76%). Rationale: matching the published recovery count and identity threshold demonstrates the Galaxy-based workflow reproduces the reference results without command-line expertise.

## Related tools

- **Galaxy** (User-friendly platform hosting the workflow and handling software dependencies)
- **NanoGalaxy** (Galaxy-based toolkit executing the long-read plasmid assembly and analysis workflow) — mmiladi/galaxy-nanopore
- **minimap2** (Computes read-vs-read overlaps to build the draft plasmid assembly)
- **miniasm** (Builds the draft de novo assembly from plasmid-containing reads)
- **Racon** (Polishes the draft assembly to correct nanopore read errors before comparison)
- **PlasFlow** (Classifies assembled contigs as plasmid-derived)
- **Bandage** (Visual inspection/confirmation of the plasmid contigs)

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- Number of reference plasmids recovered equals 19 of the 21 plasmids in the Li et al. set
- Mean sequence identity of recovered plasmids against the reference set is 97.76%
- plasmid_recovery_results.csv exists and contains both the recovery count and mean identity fields
- All reported plasmid contigs are classified as plasmid by PlasFlow and visually confirmed in Bandage
- The draft assembly was polished with Racon before identity comparison (unpolished assemblies should show lower identity)

## Limitations

- Support for other technologies such as Pacific Biosciences is untested inside Galaxy ('should also work'); tools were intensively tested only for ONT data
- No software version numbers are stated for the integrated tools in the discussion section
- Toolkit extension by the community is only expected, not yet realized

## Evidence

- [abstract] The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly: "The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly"
- [discussion] An online training manual for the AMR use case described in this publication: "An online training manual for the AMR use case described in this publication"
- [abstract] The Galaxy platform provides a user-friendly interface to computational command line–based tools, handles the software dependencies, and provides refined workflows.: "The Galaxy platform provides a user-friendly interface to computational command line–based tools, handles the software dependencies, and provides refined workflows."
- [readme] docker run -d -p 8080:80 mmiladi/galaxy-nanopore: "``docker run -d -p 8080:80 mmiladi/galaxy-nanopore``"
