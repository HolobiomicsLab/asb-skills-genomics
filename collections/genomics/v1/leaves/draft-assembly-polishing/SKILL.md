---
name: draft-assembly-polishing
description: Use when starting from Nanopore FASTQ reads (e.g. plasmid-containing
  or genomic long reads) and a de novo long-read assembly with error-corrected contigs
  is the target output, without command-line expertise.
when_to_use_negative:
- The input is short-read data (150-300 bp Illumina reads) — this skill targets long
  reads from single DNA molecules (Nanopore/PacBio); short-read tools would be required
  instead
- A finished, validated reference assembly already exists and only annotation or AMR
  analysis downstream is needed
- The platform/tool combination is untested — intensive testing in NanoGalaxy was
  performed for ONT data, so PacBio-only polishing expectations should be treated
  cautiously
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Galaxy
  role: User-friendly workflow platform hosting the tools, handling software dependencies;
    no programming experience required
- name: NanoGalaxy
  role: Galaxy-based toolkit integrating best-practice long-read assembly tools and
    workflows for Nanopore data
  repo: https://github.com/mmiladi/galaxy-nanopore
- name: minimap2
  role: Computes read-vs-read overlaps to feed the miniasm draft assembly of plasmid-containing
    reads
- name: miniasm
  role: Builds the draft de novo assembly from minimap2 overlaps
- name: Racon
  role: Polishes the miniasm draft assembly by correcting nanopore read errors using
    the original long reads
- name: PlasFlow
  role: Classifies assembled contigs as plasmid-derived
- name: Bandage
  role: Visual inspection/confirmation of the plasmid contigs in the assembly graph
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:0c5cbc17086caf5497f4c62e1d06834e94d48533c440e9bb02ae7ce621c77c68
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
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# draft-assembly-polishing

## Summary

Assemble Oxford Nanopore long reads into a draft genome/plasmid assembly with minimap2 + miniasm and correct raw nanopore sequencing errors by polishing with Racon, as implemented in the NanoGalaxy Galaxy toolkit.

## When to use

Use when starting from Nanopore FASTQ reads (e.g. plasmid-containing or genomic long reads) and a de novo long-read assembly with error-corrected contigs is the target output, without command-line expertise.

## When NOT to use

- The input is short-read data (150-300 bp Illumina reads) — this skill targets long reads from single DNA molecules (Nanopore/PacBio); short-read tools would be required instead
- A finished, validated reference assembly already exists and only annotation or AMR analysis downstream is needed
- The platform/tool combination is untested — intensive testing in NanoGalaxy was performed for ONT data, so PacBio-only polishing expectations should be treated cautiously

## Inputs

- Oxford Nanopore long-read sequencing data in FASTQ format (e.g. Li et al. plasmid-containing Nanopore reads)
- Optionally a reference plasmid/genome sequence set for benchmarking the polished assembly

## Outputs

- Racon-polished draft assembly contigs (FASTA)
- Per-contig plasmid/chromosome classification (e.g. PlasFlow predictions, inspected with Bandage)
- Benchmark metrics such as number of recovered plasmids (e.g. 19/21) and mean sequence identity (e.g. 97.76%) recorded in a results table

## How to apply

Upload the Nanopore long-read FASTQ data into the NanoGalaxy (Galaxy) environment. Compute read-vs-read overlaps and construct a draft de novo assembly from the reads using minimap2 for overlap detection and miniasm for assembly graph layout; miniasm draft contigs retain raw nanopore error rates, so polish the draft with Racon, which realigns the original long reads to the draft to correct sequencing errors. Classify resulting contigs (e.g. with PlasFlow for plasmid origin) and inspect the assembly graph with Bandage. Validate by comparing recovered sequences against a reference set — in the AMR use case the workflow recovered 19 of 21 plasmids with 97.76% mean sequence identity versus the Li et al. reference plasmids.

## Related tools

- **Galaxy** (User-friendly workflow platform hosting the tools, handling software dependencies; no programming experience required)
- **NanoGalaxy** (Galaxy-based toolkit integrating best-practice long-read assembly tools and workflows for Nanopore data) — https://github.com/mmiladi/galaxy-nanopore
- **minimap2** (Computes read-vs-read overlaps to feed the miniasm draft assembly of plasmid-containing reads)
- **miniasm** (Builds the draft de novo assembly from minimap2 overlaps)
- **Racon** (Polishes the miniasm draft assembly by correcting nanopore read errors using the original long reads)
- **PlasFlow** (Classifies assembled contigs as plasmid-derived)
- **Bandage** (Visual inspection/confirmation of the plasmid contigs in the assembly graph)

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- Number of reference plasmids recovered matches expectation: 19 of 21 in the Li et al. plasmid set
- Mean sequence identity of recovered plasmid contigs versus references reaches ~97.76%
- Every draft contig has passed through Racon polishing (raw miniasm contigs carry uncorrected nanopore errors and should not be reported as final)
- Plasmid-derived contigs are consistently labelled by PlasFlow and visually confirmed in Bandage
- Workflow runs reproducibly inside the NanoGalaxy Galaxy environment (e.g. the mmiladi/galaxy-nanopore Docker image serves Galaxy on http://localhost:8080)

## Limitations

- Integrated tools were intensively tested only for ONT data; tools supporting other technologies such as Pacific Biosciences 'should also work' but are untested
- Focus was limited to the most established long-read tools; newer/other long-read analysis tools were not integrated
- Tool version numbers are managed via the Conda framework on the Galaxy platform and are not stated in the paper
- Toolkit extension by the community is expected but not yet realized; no changelog is available

## Evidence

- [abstract] The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly: "The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly"
- [abstract] A range of best-practice tools and workflows for long-read sequence genome assembly has been integrated into a NanoGalaxy platform: "A range of best-practice tools and workflows for long-read sequence genome assembly has been integrated into a NanoGalaxy platform"
- [discussion] All tools and their dependencies are installed on the Galaxy platform and are managed by the Conda framework for dependency management: "All tools and their dependencies are installed on the Galaxy platform and are managed by the Conda framework for dependency management"
- [discussion] An online training manual for the AMR use case described in this publication: "An online training manual for the AMR use case described in this publication"
- [readme] docker run -d -p 8080:80 mmiladi/galaxy-nanopore: "``docker run -d -p 8080:80 mmiladi/galaxy-nanopore``"
