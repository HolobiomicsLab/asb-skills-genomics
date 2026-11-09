---
name: consensus-error-polishing
description: Use when a de novo assembly has been generated from Oxford Nanopore long
  reads (e.g., miniasm unitigs) and an error-corrected FASTA consensus is needed within
  a Galaxy-based workflow.
when_to_use_negative:
- The assembly was produced from short reads (150–300 bp) or by a polished/assembler
  that already emits a corrected consensus — the miniasm-to-Racon chain is specific
  to noisy long-read drafts.
- The analysis target is not genome assembly (e.g., NanoGalaxy's metagenomic analysis
  or AMR use case workflows).
- Pacific Biosciences data — the article states PB tool support inside Galaxy is untested
  ('should also work'), since only ONT data were intensively tested.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: minimap2
  role: Maps original nanopore reads back onto the miniasm unitigs to produce read-to-assembly
    alignments used by Racon for correction
- name: miniasm
  role: Generates the initial draft assembly unitigs from minimap2 read-to-read overlaps
    that serve as the polishing input
- name: Racon
  role: Polishes the miniasm assembly using the read-to-assembly alignments to derive
    the error-corrected consensus
- name: Galaxy
  role: Platform hosting the wrapped polishing tools and workflow; handles dependencies
    and exports polished_assembly.fasta
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:8c12019038885bed53601b75c213998a7da4ce0667c6c102982a73a1a5109652
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Racon
  - miniasm
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# consensus-error-polishing

## Summary

Polishes a raw long-read (nanopore) assembly by mapping the original nanopore reads back onto miniasm unitigs with minimap2 and running Racon to derive an error-corrected consensus, as implemented in the NanoGalaxy Galaxy toolkit.

## When to use

Use when a de novo assembly has been generated from Oxford Nanopore long reads (e.g., miniasm unitigs) and an error-corrected FASTA consensus is needed within a Galaxy-based workflow.

## When NOT to use

- The assembly was produced from short reads (150–300 bp) or by a polished/assembler that already emits a corrected consensus — the miniasm-to-Racon chain is specific to noisy long-read drafts.
- The analysis target is not genome assembly (e.g., NanoGalaxy's metagenomic analysis or AMR use case workflows).
- Pacific Biosciences data — the article states PB tool support inside Galaxy is untested ('should also work'), since only ONT data were intensively tested.

## Inputs

- miniasm draft assembly unitigs (FASTA)
- Oxford Nanopore long-read FASTQ reads
- minimap2 read-to-assembly alignment files (SAM/BAM)

## Outputs

- polished_assembly.fasta — error-corrected consensus assembly exported from the Galaxy history

## How to apply

Start from the miniasm unitigs produced by the NanoGalaxy rapid assembly workflow (minimap2 read-to-read overlaps -> miniasm string graph). Map the original nanopore FASTQ reads back onto the miniasm unitigs using minimap2 to produce read-to-assembly alignment files. Feed those alignments, together with the reads and the draft assembly, into Racon to compute an error-corrected consensus, then export the polished consensus assembly from the Galaxy history as polished_assembly.fasta. Rationale: miniasm output from noisy nanopore reads is unpolished, so the Racon consensus step is required to obtain a usable error-corrected assembly; NanoGalaxy wraps each step as a Galaxy tool with dependencies managed via Conda so no command-line skills are required.

## Related tools

- **minimap2** (Maps original nanopore reads back onto the miniasm unitigs to produce read-to-assembly alignments used by Racon for correction)
- **miniasm** (Generates the initial draft assembly unitigs from minimap2 read-to-read overlaps that serve as the polishing input)
- **Racon** (Polishes the miniasm assembly using the read-to-assembly alignments to derive the error-corrected consensus)
- **Galaxy** (Platform hosting the wrapped polishing tools and workflow; handles dependencies and exports polished_assembly.fasta)

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- The Galaxy history contains a minimap2 read-to-assembly alignment step whose inputs are the original nanopore FASTQ and the miniasm unitigs.
- Racon consumes the alignment files plus reads and draft assembly and outputs a consensus that differs from the raw miniasm unitigs (corrected bases).
- The final dataset is exported as polished_assembly.fasta from the Galaxy history.
- The polished consensus is a valid FASTA assembly derived from the deposited Zenodo/GigaDB nanopore dataset used in the NanoGalaxy workflow.

## Limitations

- Only ONT data were intensively tested; integrated tools supporting Pacific Biosciences 'should also work' inside Galaxy but are unverified for polishing.
- No software version numbers are stated for the integrated tools in the article; dependencies are managed by Conda on the Galaxy platform.
- Racon polishing only corrects errors captured by the read-to-assembly alignments from the same nanopore read set; the workflow covers rapid assembly, not consensus-grade finishing.

## Evidence

- [abstract] The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly: "The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly"
- [discussion] here we have performed intensive testing of the integrated tools for ONT data: "However, here we have performed intensive testing of the integrated tools for ONT data."
- [discussion] All tools and their dependencies are installed on the Galaxy platform and are managed by the Conda framework for dependency management: "All tools and their dependencies are installed on the Galaxy platform and are managed by the Conda framework for dependency management"
- [readme] docker run -d -p 8080:80 mmiladi/galaxy-nanopore: "NanoGalaxy: Galaxy toolkit and workflows for long-read Oxford Nanopore data"
