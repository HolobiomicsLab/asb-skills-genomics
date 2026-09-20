---
name: structural-variant-coordinate-reporting
description: Use when long-read (PacBio/ONT) sequencing data is aligned to a reference
  genome and a large-scale structural rearrangement such as a tandem duplication or
  deletion needs breakpoint-resolved coordinate annotation.
when_to_use_negative:
- The pathogenic variant is already resolved by short-read sequencing (small SNV/indel
  variants are not the target of this long-read breakpoint annotation approach)
- Only short-read alignment data is available; AgileStructure relies on long-read
  split-read and soft-clip signatures from long-read-specific alignments
- A fully automated SV calling pipeline is needed — AgileStructure is a desktop application
  requiring user-guided exploration of breakpoint-spanning reads
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: AgileStructure
  role: Desktop application used to visually identify breakpoints from split reads
    spanning them, and to produce the structural variant annotation from soft-clip
    data
  repo: https://github.com/msjimc/AgileStructure
- name: minimap2
  role: Long-read-specific aligner used with '-ax map-ont' and '--secondary=no' to
    align ONT reads to mm10, producing the SAM/BAM input for AgileStructure
  repo: https://github.com/lh3/minimap2
- name: Samtools
  role: Used to convert SAM to BAM, sort by genomic position, and index the alignment
    for viewing in AgileStructure
- name: MakeSVGenome
  role: Creates chromosomes containing structural rearrangements and synthetic long-read
    sequencing data for testing the accuracy of SV detection programs
  repo: msjimc/MakeSVGenome
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:b5931268cf68b7637ed3293e4fd91d2158efb8f2479f639f5b75f03a9078e7ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msjimc/AgileStructure
  tools:
  - AgileStructure
  - MakeSVGenome
  - Samtools
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# structural-variant-coordinate-reporting

## Summary

A method for annotating large-scale genomic rearrangements from aligned long-read data using AgileStructure, producing precise structural variant coordinates (e.g. tandem duplications) that can be compared to published variant calls.

## When to use

Use when long-read (PacBio/ONT) sequencing data is aligned to a reference genome and a large-scale structural rearrangement such as a tandem duplication or deletion needs breakpoint-resolved coordinate annotation.

## When NOT to use

- The pathogenic variant is already resolved by short-read sequencing (small SNV/indel variants are not the target of this long-read breakpoint annotation approach)
- Only short-read alignment data is available; AgileStructure relies on long-read split-read and soft-clip signatures from long-read-specific alignments
- A fully automated SV calling pipeline is needed — AgileStructure is a desktop application requiring user-guided exploration of breakpoint-spanning reads

## Inputs

- Long-read FASTQ data (ONT or PacBio, e.g. NCBI SRA sample SRR12783028)
- Reference genome sequence (e.g. mouse mm10 from the UCSC Genome Browser)
- Sorted, indexed BAM file produced by minimap2 and Samtools

## Outputs

- Structural variant annotation with breakpoint coordinates (e.g. chr12:37,206,133_37,300,424dup)
- Per-read annotation median-based variant coordinates
- Saved AgileStructure variant annotation record

## How to apply

Download the long-read dataset and reference sequence (e.g. mm10 from the UCSC Genome Browser), then align reads with minimap2 using the '-ax map-ont' preset and '--secondary=no' so each read has a single primary alignment; convert SAM to BAM, sort by genomic position and index with Samtools. Load the sorted, indexed BAM in AgileStructure (under WINE on POSIX systems) and screen the chromosome region around the locus of interest in blocks of up to 10 Mb, identifying a breakpoint where two or more split reads have primary and secondary alignment junction positions falling within 250 nucleotides of each other. Select the split reads linked to the breakpoint in the upper and lower panels and annotate the variant via Variant determination > Use soft clip data, letting AgileStructure merge per-read annotations into a single generic annotation and record a per-read-annotation-median-based coordinate. Judge success by comparing the reported coordinates to any published variant calls; e.g. the Agmo knockout analysis reported chr12:37,206,133_37,300,424dup (mm10) versus the published chr12:37,206,133-37,300,425dup, matching on the 5' side and differing by -1 bp on the 3' side.

## Related tools

- **AgileStructure** (Desktop application used to visually identify breakpoints from split reads spanning them, and to produce the structural variant annotation from soft-clip data) — https://github.com/msjimc/AgileStructure
- **minimap2** (Long-read-specific aligner used with '-ax map-ont' and '--secondary=no' to align ONT reads to mm10, producing the SAM/BAM input for AgileStructure) — https://github.com/lh3/minimap2
- **Samtools** (Used to convert SAM to BAM, sort by genomic position, and index the alignment for viewing in AgileStructure)
- **MakeSVGenome** (Creates chromosomes containing structural rearrangements and synthetic long-read sequencing data for testing the accuracy of SV detection programs) — msjimc/MakeSVGenome

## Examples

```
minimap2 -ax map-ont --secondary=no mm10.fa SRR12783028.fq.gz | samtools sort -o sorted.bam - && samtools index sorted.bam
```

## Evaluation signals

- Two or more split reads at the candidate breakpoint have primary and secondary alignment junction positions within 250 nucleotides of each other
- The saved AgileStructure annotation reports coordinates consistent with published variant calls (Agmo example: 5' coordinate exact at chr12:37,206,133 and 3' coordinate within 1 bp, -1 bp, of the published chr12:37,300,425)
- A per-read-annotation-median-based coordinate is recorded alongside the generic merged annotation
- Alignment was produced with '--secondary=no' so only a single primary alignment exists per read, making split-read junctions unambiguous
- The synthetic read data from MakeSVGenome (fixed length/spacing, alternating orientation, no extra mutations) can be used to benchmark that the pipeline recovers known SV coordinates

## Limitations

- AgileStructure requires user-guided exploration; breakpoint identification is a manual screening process over chromosome blocks of up to 10 Mb rather than a fully automated call
- On non-Windows systems, AgileStructure must be run through WINE, which adds a compatibility layer dependency
- The generic annotation produced by AgileStructure may differ by ~1 bp from published coordinates (e.g. -1 bp on the 3' side), so exact end-coordinate agreement should not be expected

## Evidence

- [abstract] AgileStructure is a desktop application for locating and annotating large-scale genomic rearrangements using aligned long-read data: "We developed AgileStructure, a desktop application for locating and annotating large‑scale genomic rearrangements using aligned long‑read data."
- [abstract] The software enables user-guided exploration of breakpoint-spanning reads: "The software enables user‑guided exploration of breakpoint‑spanning reads, supporting accurate interpretation of complex events"
- [intro] Alignment with a long-read-specific aligner such as minimap2: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [intro] WINE support on POSIX systems: "AgileStructure can be run on most POSIX like systems using Intel or AMD CPUs via WINE"
- [readme] minimap2 ONT preset usage: [evidence span withheld — anchor status: external_doc]
