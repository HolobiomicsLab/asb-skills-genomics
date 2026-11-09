---
name: long-read-genomic-rearrangement-analysis
description: Use when an unresolved rare-disease case is suspected to harbor a large-scale
  structural rearrangement and you have long-read sequencing data (ONT or PacBio)
  already aligned to a reference genome in BAM format.
when_to_use_negative:
- The pathogenic variant is already resolved by short-read data or is an SNV/small
  indel, which short-read workflows detect well
- Input is raw long-read signal or unaligned reads — the skill requires an indexed
  BAM of alignments produced by a long-read-specific aligner
- A fully automated, no-user-verification SV caller output is required, since AgileStructure
  emphasizes user-guided exploration of breakpoint-spanning reads
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: AgileStructure
  role: Desktop application that locates and annotates large-scale genomic rearrangements
    from aligned long-read data by clustering split-read junction positions into putative
    breakpoints
  repo: https://github.com/msjimc/AgileStructure
- name: minimap2
  role: Long-read-specific aligner used to produce the BAM alignments (e.g. -ax map-ont
    preset for ONT reads) that AgileStructure consumes
  repo: https://github.com/lh3/minimap2
- name: lra
  role: Alternative long-read aligner that can produce the alignments used for rearrangement
    detection
  repo: https://github.com/ChaissonLab/LRA
- name: MakeSVGenome
  role: Generates chromosomes with structural rearrangements and synthetic long reads
    to test the accuracy of the detection workflow
  repo: msjimc/MakeSVGenome
- name: WINE
  role: Compatibility layer allowing AgileStructure to run on most POSIX-like systems
    using Intel or AMD CPUs
  repo: msjimc/RunningWindowsProgramsOnLinux
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:cae4d01f04d6c1e7f4b8bba088a4b36071058e63278903935733c31b82bf968a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msjimc/AgileStructure
  tools:
  - AgileStructure
  - MakeSVGenome
  - Samtools
  - WINE
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# long-read-genomic-rearrangement-analysis

## Summary

Detect and annotate large-scale genomic rearrangements from aligned long-read (PacBio or ONT) data using AgileStructure, a desktop application that identifies breakpoint-spanning split reads and supports user-guided interpretation of complex structural variant events.

## When to use

Use when an unresolved rare-disease case is suspected to harbor a large-scale structural rearrangement and you have long-read sequencing data (ONT or PacBio) already aligned to a reference genome in BAM format.

## When NOT to use

- The pathogenic variant is already resolved by short-read data or is an SNV/small indel, which short-read workflows detect well
- Input is raw long-read signal or unaligned reads — the skill requires an indexed BAM of alignments produced by a long-read-specific aligner
- A fully automated, no-user-verification SV caller output is required, since AgileStructure emphasizes user-guided exploration of breakpoint-spanning reads

## Inputs

- Indexed BAM file of long reads aligned to a reference genome (e.g. ONT reads aligned with minimap2 -ax map-ont)
- Reference genome assembly matching the alignment
- Optional: synthetic rearranged chromosomes and simulated long reads from MakeSVGenome for benchmarking

## Outputs

- Called breakpoint coordinates (chromosome, position, supporting read count), e.g. written to breakpoints.tsv
- Annotated structural rearrangement based on user-guided exploration of breakpoint-spanning reads

## How to apply

Align the generated long reads to the appropriate reference genome with a long-read-specific aligner such as minimap2 (e.g. the -ax map-ont preset for ONT data, with secondary alignments retained) or lra, producing an indexed BAM file. Load the indexed BAM into AgileStructure, which reads the primary alignments and their linked secondary alignments and retains the genomic positions of junctions between primary and secondary alignments of split reads. AgileStructure clusters these junction positions at the locus of interest and calls a putative breakpoint when 2 or more junction positions occur within 250 nucleotides of each other. The user then visually explores the breakpoint-spanning reads within the application to confirm and annotate the rearrangement, a workflow motivated by the finding that existing align/visualise pipelines (e.g. IGV alone) are limited for rapid structural variant detection. For validation of the detection pipeline itself, synthetic chromosomes and synthetic long-read data with known rearrangements can be generated with MakeSVGenome.

## Related tools

- **AgileStructure** (Desktop application that locates and annotates large-scale genomic rearrangements from aligned long-read data by clustering split-read junction positions into putative breakpoints) — https://github.com/msjimc/AgileStructure
- **minimap2** (Long-read-specific aligner used to produce the BAM alignments (e.g. -ax map-ont preset for ONT reads) that AgileStructure consumes) — https://github.com/lh3/minimap2
- **lra** (Alternative long-read aligner that can produce the alignments used for rearrangement detection) — https://github.com/ChaissonLab/LRA
- **MakeSVGenome** (Generates chromosomes with structural rearrangements and synthetic long reads to test the accuracy of the detection workflow) — msjimc/MakeSVGenome
- **WINE** (Compatibility layer allowing AgileStructure to run on most POSIX-like systems using Intel or AMD CPUs) — msjimc/RunningWindowsProgramsOnLinux

## Examples

```
minimap2 -ax map-ont ref.fa ont.fq.gz > aln.sam
```

## Evaluation signals

- Each reported breakpoint is supported by at least 2 split-read junction positions falling within a 250-nucleotide window
- Output breakpoint file is a well-formed TSV with chromosome, position, and support-count fields
- Called breakpoints are consistent between minimap2- and lra-aligned versions of the same read set
- On MakeSVGenome synthetic chromosomes, detected breakpoints match the known, manually created rearrangement coordinates
- Breakpoint-spanning reads visualized in AgileStructure show the expected primary/secondary alignment junctions at the called coordinates

## Limitations

- Approximately 50% of rare-disease patients' pathogenic variants are not resolved with short-read data; large rearrangements remain harder to detect and this skill targets exactly those unresolved cases
- AgileStructure is a Windows desktop application requiring WINE on POSIX-like systems (Intel or AMD CPUs), which may complicate automated pipelines
- The workflow involves user-guided interpretation of breakpoint-spanning reads, so results depend on the analyst's review of complex events

## Evidence

- [abstract] AgileStructure as desktop SV caller from aligned long reads: "We developed AgileStructure, a desktop application for locating and annotating large‑scale genomic rearrangements using aligned long‑read data."
- [intro] Alignment prerequisite with long-read aligner: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [intro] Motivation: unresolved cases due to large rearrangements: "It has been suggested that many of these cases are due to large-scale structural rearrangements, which are more difficult to detect compared to SNVs or small indels when using short-read data."
- [abstract] User-guided exploration supports interpretation: "The software enables user‑guided exploration of breakpoint‑spanning reads, supporting accurate interpretation of complex events"
- [intro] AgileStructure runs on POSIX systems via WINE: "AgileStructure can be run on most POSIX like systems using Intel or AMD CPUs via WINE"
- [readme] MakeSVGenome for benchmarking detection accuracy: [evidence span withheld — anchor status: not_found]
