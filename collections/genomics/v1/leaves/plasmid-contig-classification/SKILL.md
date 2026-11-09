---
name: plasmid-contig-classification
description: Use when you have draft or polished contigs from an ONT long-read assembly
  (e.g. minimap2/miniasm/Racon output) and need to identify which contigs are plasmid-derived
  versus chromosomal, as in the Li et al. plasmid recovery benchmark.
when_to_use_negative:
- Input is raw Nanopore FASTQ reads that have not yet been assembled into contigs
  — run the minimap2/miniasm/Racon assembly steps first
- The sequencing data are short reads (150–300 bp); NanoGalaxy targets long-read Nanopore
  data, where reads exceed ~10 kb
- You need a metagenomic taxonomic profile rather than plasmid-vs-chromosome contig
  classification
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PlasFlow
  role: Classifies assembled contigs as plasmid-derived based on sequence features
  repo: https://toolshed.g2.bx.psu.edu/
- name: Bandage
  role: Graphical inspection and confirmation of the PlasFlow-designated plasmid contigs
    within the assembly graph
- name: minimap2
  role: Computes read-vs-read overlaps on plasmid-containing Nanopore reads prior
    to contig generation
  repo: https://nanopore.usegalaxy.eu
- name: miniasm
  role: Builds the draft de novo assembly whose contigs are subsequently classified
- name: Racon
  role: Polishes the draft assembly to correct nanopore read errors before classification
    and comparison
- name: NanoGalaxy
  role: Galaxy-based toolkit providing the integrated environment in which the whole
    plasmid classification workflow runs
  repo: https://nanopore.usegalaxy.eu
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:c06004fa3a0dea0f963a0913f13b556a95aece033df69098ec5afe1a6faedab1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bandage
  - NanoGalaxy
  - PlasFlow
  - Racon
  - miniasm
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# plasmid-contig-classification

## Summary

Classify contigs from Nanopore long-read de novo assemblies as plasmid-derived using PlasFlow inside the NanoGalaxy Galaxy environment, then visually confirm candidate plasmid contigs with Bandage before comparing them to a reference plasmid set.

## When to use

Use when you have draft or polished contigs from an ONT long-read assembly (e.g. minimap2/miniasm/Racon output) and need to identify which contigs are plasmid-derived versus chromosomal, as in the Li et al. plasmid recovery benchmark.

## When NOT to use

- Input is raw Nanopore FASTQ reads that have not yet been assembled into contigs — run the minimap2/miniasm/Racon assembly steps first
- The sequencing data are short reads (150–300 bp); NanoGalaxy targets long-read Nanopore data, where reads exceed ~10 kb
- You need a metagenomic taxonomic profile rather than plasmid-vs-chromosome contig classification

## Inputs

- Nanopore long-read sequencing FASTQ data (e.g. Li et al. plasmid-containing reads)
- Draft or Racon-polished FASTA contigs from minimap2/miniasm assembly

## Outputs

- Contig classification labels (plasmid-derived vs chromosomal) from PlasFlow
- Bandage assembly graph visualizations confirming plasmid contigs
- plasmid_recovery_results.csv recording plasmids recovered and mean sequence identity

## How to apply

First produce contigs by de novo assembly of the plasmid-containing Nanopore FASTQ reads with minimap2 and miniasm, then polish the draft with Racon to correct nanopore read errors. Classify the polished contigs as plasmid-derived versus chromosomal using PlasFlow, a tool integrated into the NanoGalaxy toolkit on the European Galaxy server. Inspect and confirm the PlasFlow-designated plasmid contigs graphically with Bandage to check assembly structure and circularity. Finally, compare the confirmed plasmid sequences against the reference plasmid set (e.g. the Li et al. set), recording the number recovered (target: 19 of 21) and the mean sequence identity (target: 97.76%) in plasmid_recovery_results.csv. The rationale is that NanoGalaxy wraps all of these tools so users without programming experience can run the full long-read pipeline reproducibly in Galaxy.

## Related tools

- **PlasFlow** (Classifies assembled contigs as plasmid-derived based on sequence features) — https://toolshed.g2.bx.psu.edu/
- **Bandage** (Graphical inspection and confirmation of the PlasFlow-designated plasmid contigs within the assembly graph)
- **minimap2** (Computes read-vs-read overlaps on plasmid-containing Nanopore reads prior to contig generation) — https://nanopore.usegalaxy.eu
- **miniasm** (Builds the draft de novo assembly whose contigs are subsequently classified)
- **Racon** (Polishes the draft assembly to correct nanopore read errors before classification and comparison)
- **NanoGalaxy** (Galaxy-based toolkit providing the integrated environment in which the whole plasmid classification workflow runs) — https://nanopore.usegalaxy.eu

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- The workflow recovers 19 of the 21 plasmids in the Li et al. reference plasmid set
- Mean sequence identity between recovered plasmid sequences and the reference set is 97.76%
- Every contig labeled plasmid-derived by PlasFlow is visually consistent with the assembly structure when inspected in Bandage
- Results are recorded in plasmid_recovery_results.csv with both recovery count and mean identity fields populated
- Classified contigs originate from a Racon-polished assembly, not the raw miniasm draft

## Limitations

- NanoGalaxy focused on the most established and widely used long-read tools; other plasmid-classification or analysis tools may exist but were not integrated
- Only ONT data were intensively tested; results for other platforms (e.g. Pacific Biosciences) are expected but unverified
- Users do not need programming experience, but PlasFlow classification quality still depends on the accuracy of the upstream long-read assembly and Racon polishing

## Evidence

- [abstract] including de novo genome assembly from genomic, metagenomic, and plasmid sequence reads: "The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly from genomic,"
- [discussion] enable non-bioinformatics-trained researchers to perform genomics analysis: "enable non-bioinformatics-trained researchers to perform extensive genomics analysis using long-read sequence data, without the need for any coding skills"
- [abstract] NanoGalaxy tools and workflows freely available at European Galaxy server: "A range of best-practice tools and workflows for long-read sequence genome assembly has been integrated into a NanoGalaxy platform to facilitate easy access and use of bioinformatics tools for"
- [readme] docker run command to launch the NanoGalaxy container: "docker run -d -p 8080:80 mmiladi/galaxy-nanopore"
