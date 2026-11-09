---
name: nanopore-long-read-assembly
description: Use when the input is Oxford Nanopore long-read FASTQ data (genomic,
  metagenomic, or plasmid) and a quick de novo assembly with error-corrected consensus
  is needed without command-line skills.
when_to_use_negative:
- Input consists of short reads (150–300 bp), which the article notes complicate assembly
  of repetitive regions — use a short-read workflow instead
- Input is already an assembled set of contigs requiring only classification (e.g.
  only PlasFlow/Krona steps are needed)
- The platform is Pacific Biosciences data and validated performance is required —
  the article states PacBio tools 'should also work' but were not intensively tested
  for ONT alternatives
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Galaxy
  role: User-friendly platform hosting the workflow; handles software dependencies
    and provides refined workflows
- name: NanoGalaxy
  role: Galaxy-based toolkit providing the integrated long-read assembly tools and
    workflows
  repo: mmiladi/galaxy-nanopore
- name: minimap2
  role: Computes read-to-read overlaps in long-read overlap mode and maps reads back
    onto miniasm unitigs for correction
- name: miniasm
  role: Builds the string graph and generates the initial draft assembly unitigs from
    overlap data
- name: Racon
  role: Polishes the miniasm assembly using read-to-assembly alignments to derive
    an error-corrected consensus
- name: PlasFlow
  role: Classifies assembled contigs as plasmid-derived in the AMR plasmid-recovery
    use case
- name: Bandage
  role: Visual inspection/confirmation of plasmid contigs in the assembly graph
- name: Conda
  role: Framework managing installation of all tools and their dependencies on the
    Galaxy platform
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:c451e3cc4dad55ebecd72f1fa15089ac56e1f33db59ac996f828bc82c1c1c3dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mmiladi/galaxy-nanopore
  tools:
  - Bandage
  - Conda
  - Galaxy
  - NanoGalaxy
  - PlasFlow
  - Racon
  - miniasm
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# nanopore-long-read-assembly

## Summary

Rapid de novo assembly of Oxford Nanopore long-read sequencing data within the Galaxy (NanoGalaxy) environment, chaining minimap2 overlap detection, miniasm string-graph assembly, and Racon polishing. It packages best-practice long-read tools behind Galaxy's GUI so non-bioinformaticians can assemble genomic, metagenomic, or plasmid reads.

## When to use

Use when the input is Oxford Nanopore long-read FASTQ data (genomic, metagenomic, or plasmid) and a quick de novo assembly with error-corrected consensus is needed without command-line skills.

## When NOT to use

- Input consists of short reads (150–300 bp), which the article notes complicate assembly of repetitive regions — use a short-read workflow instead
- Input is already an assembled set of contigs requiring only classification (e.g. only PlasFlow/Krona steps are needed)
- The platform is Pacific Biosciences data and validated performance is required — the article states PacBio tools 'should also work' but were not intensively tested for ONT alternatives

## Inputs

- Oxford Nanopore long-read sequencing data in FASTQ format
- Nanopore metagenomic or plasmid-containing FASTQ reads (e.g. deposited via Zenodo/GigaDB)
- Optional: reference plasmid sequence set for identity comparison

## Outputs

- polished_assembly.fasta (Racon-corrected consensus assembly exported from Galaxy)
- miniasm draft unitigs and minimap2 overlap/alignment files (PAF/BAM)
- PlasFlow plasmid-contig classifications and Bandage visualizations
- plasmid_recovery_results.csv (recovery counts and mean sequence identity vs. reference set)

## How to apply

Import the deposited nanopore long-read dataset (FASTQ, e.g. from Zenodo/GigaDB) into a Galaxy history on NanoGalaxy (or launch the mmiladi/galaxy-nanopore Docker image). Compute read-vs-read overlaps with minimap2 in its long-read overlap mode to feed the overlap-layout-consensus stage, then build the string graph and initial assembly unitigs with miniasm. Map the original nanopore reads back onto the miniasm unitigs with minimap2 and run Racon to derive an error-corrected consensus. For plasmid-focused analyses, classify contigs as plasmid-derived with PlasFlow and visually confirm them with Bandage, then record recovery counts and mean sequence identity against a reference set (e.g. 19/21 plasmids at 97.76% mean identity in the AMR use case). Rationale: all tools and dependencies are installed and managed via the Conda framework on Galaxy, and wrappers are openly tested on GitHub with Travis CI.

## Related tools

- **Galaxy** (User-friendly platform hosting the workflow; handles software dependencies and provides refined workflows)
- **NanoGalaxy** (Galaxy-based toolkit providing the integrated long-read assembly tools and workflows) — mmiladi/galaxy-nanopore
- **minimap2** (Computes read-to-read overlaps in long-read overlap mode and maps reads back onto miniasm unitigs for correction)
- **miniasm** (Builds the string graph and generates the initial draft assembly unitigs from overlap data)
- **Racon** (Polishes the miniasm assembly using read-to-assembly alignments to derive an error-corrected consensus)
- **PlasFlow** (Classifies assembled contigs as plasmid-derived in the AMR plasmid-recovery use case)
- **Bandage** (Visual inspection/confirmation of plasmid contigs in the assembly graph)
- **Conda** (Framework managing installation of all tools and their dependencies on the Galaxy platform)

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- The Racon-polished consensus is exported as polished_assembly.fasta from the Galaxy history
- For the AMR plasmid benchmark, the pipeline recovers 19 of 21 plasmids from the Li et al. reference set with mean sequence identity of 97.76%
- minimap2 overlap output feeds a non-empty miniasm string graph with unitig output before Racon polishing
- PlasFlow labels a subset of contigs as plasmid-derived and Bandage rendering confirms those contigs visually
- All integrated tools resolve dependencies cleanly under Conda and pass Galaxy wrapper tests (Travis CI)

## Limitations

- Other long-read sequencing analysis tools exist or are under development but were not integrated; the toolkit focuses on the most established and widely used tools
- Support for Pacific Biosciences tools inside Galaxy is untested ('should also work'); only ONT tools were intensively tested
- Toolkit extension by the community is only expected, not yet realized
- As more tools become available, these will need to be assembled into existing or new toolkits; no changelog or software version numbers are stated for the integrated tools

## Evidence

- [abstract] The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly: "The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly"
- [abstract] The Galaxy platform provides a user-friendly interface to computational command line–based tools, handles the software dependencies, and provides refined workflows.: "The Galaxy platform provides a user-friendly interface to computational command line–based tools, handles the software dependencies, and provides refined workflows."
- [discussion] All tools and their dependencies are installed on the Galaxy platform and are managed by the Conda framework for dependency management: "All tools and their dependencies are installed on the Galaxy platform and are managed by the Conda framework for dependency management"
- [discussion] we expect that the toolkit will be further extended by the community because NanoGalaxy is part of the open Galaxy platform: "we expect that the toolkit will be further extended by the community because NanoGalaxy is part of the open Galaxy platform"
- [readme] Inside the container a Apache Webserver is running on port 80 and that port can be bound to a local port on your host computer.: "Inside the container a Apache Webserver is running on port 80 and that port can be bound to a local port on your host computer."
