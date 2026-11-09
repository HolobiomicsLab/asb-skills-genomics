---
name: structural-variant-annotation
description: Use when ONT or PacBio long reads are aligned to a reference genome and
  a suspected large-scale rearrangement (translocation, deletion, inversion) needs
  breakpoint-level annotation, e.g. reproducing a published t(chr6;chr8) breakpoint
  from SRA reads.
when_to_use_negative:
- The underlying variant is expected to be a SNV or small indel resolvable by short-read
  sequencing; AgileStructure targets large-scale structural rearrangements that short-read
  data misses.
- Only short-read Illumina data is available — breakpoint-spanning split reads from
  long reads are required to call junctions.
- You need fully automated, unsupervised SV discovery rather than user-guided interpretation
  of specific candidate breakpoints.
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: AgileStructure
  role: Desktop application to visualise split-read alignments, call breakpoints from
    clustered junctions, and annotate the structural variant via the Variant determination
    menu
  repo: https://github.com/msjimc/AgileStructure
- name: minimap2
  role: Long-read-specific aligner used with '-ax map-ont --secondary=no' to produce
    the split-read alignments AgileStructure inspects
  repo: https://github.com/lh3/minimap2
- name: lra
  role: Alternative long-read aligner mentioned for aligning long reads before SV
    inspection
  repo: https://github.com/ChaissonLab/LRA
- name: WINE
  role: Compatibility layer to run the Windows AgileStructure binary on POSIX systems
  repo: https://github.com/msjimc/RunningWindowsProgramsOnLinux
- name: MakeSVGenome
  role: Generates synthetic rearranged chromosomes and simulated long-read data to
    benchmark SV detection accuracy
  repo: https://github.com/msjimc/MakeSVGenome
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:4d3fd2d69cf47903df98017f81f20d085ca5febb1e2a64e1ee96c583e88b36e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msjimc/AgileStructure
  tools:
  - AgileStructure
  - MakeSVGenome
  - WINE
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# structural-variant-annotation

## Summary

Locate and annotate large-scale genomic structural rearrangements (e.g. translocations) from aligned long-read sequencing data using AgileStructure, a desktop application that supports user-guided exploration of breakpoint-spanning split reads and consensus per-read variant annotation.

## When to use

Use when ONT or PacBio long reads are aligned to a reference genome and a suspected large-scale rearrangement (translocation, deletion, inversion) needs breakpoint-level annotation, e.g. reproducing a published t(chr6;chr8) breakpoint from SRA reads.

## When NOT to use

- The underlying variant is expected to be a SNV or small indel resolvable by short-read sequencing; AgileStructure targets large-scale structural rearrangements that short-read data misses.
- Only short-read Illumina data is available — breakpoint-spanning split reads from long reads are required to call junctions.
- You need fully automated, unsupervised SV discovery rather than user-guided interpretation of specific candidate breakpoints.

## Inputs

- Indexed BAM/SAM of long reads aligned to a reference genome (e.g. hg19) with secondary alignments disabled
- Long-read sequencing dataset (ONT or PacBio), e.g. SRR9982132 from NCBI SRA
- Reference genome FASTA (e.g. hg19 from UCSC)

## Outputs

- Annotated structural variant with breakpoint coordinates (e.g. t(chr6;chr8) (g.167281716; g.113696100))
- translocation_annotation.txt containing breakpoint coordinates and supporting read information

## How to apply

Align ONT long reads to the hg19 reference with minimap2 using '-ax map-ont' and '--secondary=no', then convert, sort and index the BAM with samtools. Load the indexed BAM into AgileStructure and examine split-read alignments at candidate breakpoints (e.g. chr6:167,281,717 and chr8:113,696,089); a putative breakpoint is called when 2 or more split reads have junction positions within 250 nucleotides of each other. Annotate the variant via the Variant determination menu, using the consensus of per-read annotations, and record the coordinates (e.g. t(chr6;chr8) (g.167281716; g.113696100)). On Linux/macOS, AgileStructure runs under WINE; save the annotation with supporting read information to a text file such as translocation_annotation.txt.

## Related tools

- **AgileStructure** (Desktop application to visualise split-read alignments, call breakpoints from clustered junctions, and annotate the structural variant via the Variant determination menu) — https://github.com/msjimc/AgileStructure
- **minimap2** (Long-read-specific aligner used with '-ax map-ont --secondary=no' to produce the split-read alignments AgileStructure inspects) — https://github.com/lh3/minimap2
- **lra** (Alternative long-read aligner mentioned for aligning long reads before SV inspection) — https://github.com/ChaissonLab/LRA
- **WINE** (Compatibility layer to run the Windows AgileStructure binary on POSIX systems) — https://github.com/msjimc/RunningWindowsProgramsOnLinux
- **MakeSVGenome** (Generates synthetic rearranged chromosomes and simulated long-read data to benchmark SV detection accuracy) — https://github.com/msjimc/MakeSVGenome

## Examples

```
minimap2 -ax map-ont --secondary=no hg19.fa reads.fq.gz | samtools sort -o sorted.bam - && samtools index sorted.bam
```

## Evaluation signals

- Called chr6 breakpoint falls within -1/+2 bp of the published g.167,281,717 position
- Called chr8 breakpoint falls within +9/+11 bp of the published g.113,696,089 position
- Each called breakpoint is supported by 2 or more split reads with junction positions within 250 nucleotides of each other
- Per-read annotations reach a consensus (chr6-primary and chr8-primary calls agree within a few bp)
- BAM was produced with --secondary=no and successfully indexed, so only primary alignments contribute to calls

## Limitations

- Requires user-guided exploration of breakpoint-spanning reads; existing programs are noted as limited for rapid detection and annotation, so results depend on the analyst examining the right loci.
- Aligned with --secondary=no, so calls rely on primary alignments; chr6-primary and chr8-primary annotation sets can differ by a few base pairs (e.g. g.167281716 vs g.167281719), requiring consensus judgement.
- Breakpoint calling depends on a 250 nucleotide junction-clustering window and a minimum of 2 supporting split reads, which may miss events with very sparse read support.

## Evidence

- [abstract] We developed AgileStructure, a desktop application for locating and annotating large‑scale genomic rearrangements using aligned long‑read data.: "We developed AgileStructure, a desktop application for locating and annotating large‑scale genomic rearrangements using aligned long‑read data."
- [abstract] The software enables user‑guided exploration of breakpoint‑spanning reads, supporting accurate interpretation of complex events: "The software enables user‑guided exploration of breakpoint‑spanning reads, supporting accurate interpretation of complex events"
- [intro] the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [readme] ./minimap2 -ax map-ont ref.fa ont.fq.gz > aln.sam         # Oxford Nanopore genomic reads: [evidence span withheld — anchor status: external_doc]
- [intro] AgileStructure can be run on most POSIX like systems using Intel or AMD CPUs via WINE: "AgileStructure can be run on most POSIX like systems using Intel or AMD CPUs via WINE"
