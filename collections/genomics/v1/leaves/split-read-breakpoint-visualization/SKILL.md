---
name: split-read-breakpoint-visualization
description: Use when long-read sequencing data (ONT or PacBio) is suspected to harbor
  a large structural rearrangement (duplication, deletion, translocation) that short-read
  data failed to resolve and a breakpoint must be pinpointed and annotated.
when_to_use_negative:
- The input is short-read sequencing data — the article motivates long-read data because
  approximately 50% of rare-disease variants are not resolved with short reads and
  large rearrangements are harder to detect with them
- The goal is automated, hands-off SV calling across many samples rather than user-guided
  interpretation of a single complex event
- Reads are not aligned (raw FASTQ only) and no reference genome is available to align
  them against
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Long-read-specific aligner used with '-ax map-ont' preset and '--secondary=no'
    to generate the split-read alignments (SAM/BAM) that expose primary/secondary
    junction positions
  repo: https://github.com/lh3/minimap2
- name: Samtools
  role: Converts minimap2 SAM output to BAM, sorts by genomic position, and indexes
    the BAM so it can be loaded in AgileStructure
- name: AgileStructure
  role: Desktop application used to screen the aligned BAM, visualize breakpoint-spanning
    split reads in upper/lower panels, and annotate the rearrangement via Variant
    determination > Use soft clip data
  repo: https://github.com/msjimc/AgileStructure
- name: lra
  role: Alternative long-read aligner that can substitute for minimap2 in producing
    the alignments inspected in AgileStructure
  repo: https://github.com/ChaissonLab/LRA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:48c2160c0feec3d5d72803f24c6b8ff771a38f6223ea210e39fb9bdb79647938
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - AgileStructure
  - Samtools
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# split-read-breakpoint-visualization

## Summary

A user-guided workflow for locating and annotating large-scale genomic rearrangements from aligned long-read data (PacBio/ONT) by visually inspecting breakpoint-spanning split reads in AgileStructure, then recording per-read and consensus variant coordinates.

## When to use

Use when long-read sequencing data (ONT or PacBio) is suspected to harbor a large structural rearrangement (duplication, deletion, translocation) that short-read data failed to resolve and a breakpoint must be pinpointed and annotated.

## When NOT to use

- The input is short-read sequencing data — the article motivates long-read data because approximately 50% of rare-disease variants are not resolved with short reads and large rearrangements are harder to detect with them
- The goal is automated, hands-off SV calling across many samples rather than user-guided interpretation of a single complex event
- Reads are not aligned (raw FASTQ only) and no reference genome is available to align them against

## Inputs

- Long-read sequencing data (FASTQ/FASTA from ONT or PacBio, e.g. SRA sample SRR12783028)
- Reference genome FASTA (e.g. mouse mm10 from the UCSC Genome Browser)
- Sorted, indexed BAM file of long reads aligned with minimap2

## Outputs

- Structural variant annotation in HGVS-like form (e.g. chr12:37,206,133_37,300,424dup)
- Per-read breakpoint annotations and a per-read-annotation-median-based coordinate
- Visually identified breakpoint position(s) based on concordant split-read junctions

## How to apply

Align the long reads to the relevant reference (e.g. mouse mm10 from UCSC) with minimap2 using the long-read preset '-ax map-ont' and '--secondary=no', then convert the SAM to BAM, sort by genomic position and index it with Samtools so AgileStructure can load it. Screen the chromosome region around the candidate locus (e.g. the Agmo locus on chr12) in blocks of up to 10 Mb and locate the breakpoint by finding two or more split reads whose primary and secondary alignment junction positions fall within 250 nucleotides of each other. Select the split reads linked to the breakpoint in the upper and lower panels, then use the Variant determination > Use soft clip data menu so AgileStructure derives per-read annotations from soft-clipped sequence and consolidates them into a single generic annotation. The program supports user-guided exploration of breakpoint-spanning reads precisely because existing alignment/visualisation tools (e.g. IGV alone) are limited for rapid SV detection and annotation.

## Related tools

- **minimap2** (Long-read-specific aligner used with '-ax map-ont' preset and '--secondary=no' to generate the split-read alignments (SAM/BAM) that expose primary/secondary junction positions) — https://github.com/lh3/minimap2
- **Samtools** (Converts minimap2 SAM output to BAM, sorts by genomic position, and indexes the BAM so it can be loaded in AgileStructure)
- **AgileStructure** (Desktop application used to screen the aligned BAM, visualize breakpoint-spanning split reads in upper/lower panels, and annotate the rearrangement via Variant determination > Use soft clip data) — https://github.com/msjimc/AgileStructure
- **lra** (Alternative long-read aligner that can substitute for minimap2 in producing the alignments inspected in AgileStructure) — https://github.com/ChaissonLab/LRA

## Examples

```
minimap2 -ax map-ont --secondary=no mm10.fa SRR12783028.fastq > aln.sam; samtools sort -o aln.sorted.bam aln.sam && samtools index aln.sorted.bam
```

## Evaluation signals

- The saved annotation reports the duplication as chr12:37,206,133_37,300,424dup (mm10), matching the published chr12:37,206,133–37,300,425dup on the 5' side and differing by only -1 bp on the 3' side
- A per-read-annotation-median-based coordinate is recorded alongside the generic annotation
- Two or more split reads have primary and secondary alignment junction positions falling within 250 nucleotides of each other at the called breakpoint
- The BAM is sorted and indexed and successfully loads in AgileStructure; screening is done in blocks of up to 10 Mb around the locus
- A soft-clip-based per-read annotation set is generated and consolidated into a single generic annotation via the Variant determination menu

## Limitations

- AgileStructure is designed for user-guided exploration, so breakpoint identification depends on the analyst visually screening regions in blocks of up to 10 Mb rather than on fully automated calling
- Breakpoint confidence rests on the heuristic of two or more split reads with junction positions within 250 nucleotides; events with fewer supporting split reads may be missed
- Coordinates can differ by small offsets (e.g. -1 bp on the 3' side vs published coordinates), so results should be cross-checked against published variant positions

## Evidence

- [abstract] We developed AgileStructure, a desktop application for locating and annotating large-scale genomic rearrangements using aligned long-read data.: "We developed AgileStructure, a desktop application for locating and annotating large‑scale genomic rearrangements using aligned long‑read data."
- [abstract] The software enables user-guided exploration of breakpoint-spanning reads, supporting accurate interpretation of complex events.: "The software enables user‑guided exploration of breakpoint‑spanning reads, supporting accurate interpretation of complex events"
- [intro] Existing programs are limited for rapid detection and annotation of structural rearrangements, motivating AgileStructure.: "these programs are limited in their ability to allow the rapid detection and annotation of structural rearrangements; consequently, we have developed AgileStructure specifically for this purpose."
- [readme] Long-read data is aligned with a long-read-specific aligner such as minimap2 before visual inspection.: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
