---
name: split-read-translocation-breakpoint-identification
description: Use when ONT or PacBio long reads (BAM from minimap2 alignment to hg19)
  must be inspected to call and annotate a suspected inter-chromosomal translocation
  breakpoint, e.g. reproducing a published t(chr6;chr8) coordinate set.
when_to_use_negative:
- 'Input is short-read sequencing data: the article states approximately 50% of pathogenic
  variants are not resolved with short-read data and AgileStructure is built for aligned
  long-read data.'
- 'The goal is automated high-throughput SV discovery without manual review: AgileStructure
  is designed for user-guided exploration and annotation of rearrangements rather
  than fully automated calling.'
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: AgileStructure
  role: Desktop application for user-guided split-read exploration and Variant determination
    annotation of translocation breakpoints from indexed BAM files
  repo: https://github.com/msjimc/AgileStructure
- name: minimap2
  role: Long-read-specific aligner producing the SAM/BAM input; run with -ax map-ont
    and --secondary=no
  repo: https://github.com/lh3/minimap2
- name: Samtools
  role: SAM-to-BAM conversion, sorting, and indexing of the alignment before loading
    into AgileStructure
- name: MakeSVGenome
  role: Generates synthetic chromosomes with structural rearrangements and simulated
    long reads for testing SV detection accuracy
  repo: https://github.com/msjimc/MakeSVGenome
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:848e3b6d56a783380a5598d6ffa35721532700c19a41513e191ed6962f840cee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msjimc/AgileStructure
  tools:
  - AgileStructure
  - MakeSVGenome
  - Samtools
  - minimap2
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# split-read-translocation-breakpoint-identification

## Summary

Identify and annotate inter-chromosomal translocation breakpoints from aligned long-read sequencing data using AgileStructure's split-read exploration and Variant determination menu, validated against published breakpoint coordinates.

## When to use

Use when ONT or PacBio long reads (BAM from minimap2 alignment to hg19) must be inspected to call and annotate a suspected inter-chromosomal translocation breakpoint, e.g. reproducing a published t(chr6;chr8) coordinate set.

## When NOT to use

- Input is short-read sequencing data: the article states approximately 50% of pathogenic variants are not resolved with short-read data and AgileStructure is built for aligned long-read data.
- The goal is automated high-throughput SV discovery without manual review: AgileStructure is designed for user-guided exploration and annotation of rearrangements rather than fully automated calling.

## Inputs

- Long-read sequencing dataset in SRA (e.g. SRR9982132, ONT long reads)
- Human hg19 reference genome sequence (FASTA from UCSC Genome Browser)
- Indexed, sorted BAM of long reads aligned to hg19 with minimap2 (-ax map-ont --secondary=no)
- Putative breakpoint coordinates (e.g. chr6:167,281,717 and chr8:113,696,089)

## Outputs

- Annotated t(chr6;chr8) translocation with breakpoint coordinates from per-read consensus
- translocation_annotation.txt containing breakpoint coordinates and supporting read information

## How to apply

Download the long-read dataset (e.g. SRR9982132 from NCBI SRA) and the hg19 reference from UCSC, then align with minimap2 using -ax map-ont and --secondary=no. Convert SAM to BAM, sort and index with Samtools, and load the indexed BAM into AgileStructure (via WINE on POSIX systems). Examine split-read alignments at the candidate loci (e.g. chr6:167,281,717 and chr8:113,696,089), calling a putative breakpoint when 2 or more split reads have junction positions within 250 nucleotides of each other. Annotate the rearrangement with the Variant determination menu using the consensus of per-read annotations, and save breakpoints and supporting read information. For SRR9982132, AgileStructure recovered t(chr6;chr8) at g.167,281,716/g.113,696,100 (primary alignments on chr6) and g.167,281,719/g.113,696,098 (on chr8), i.e. within -1/+2 bp (chr6) and +9/+11 bp (chr8) of the published positions.

## Related tools

- **AgileStructure** (Desktop application for user-guided split-read exploration and Variant determination annotation of translocation breakpoints from indexed BAM files) — https://github.com/msjimc/AgileStructure
- **minimap2** (Long-read-specific aligner producing the SAM/BAM input; run with -ax map-ont and --secondary=no) — https://github.com/lh3/minimap2
- **Samtools** (SAM-to-BAM conversion, sorting, and indexing of the alignment before loading into AgileStructure)
- **MakeSVGenome** (Generates synthetic chromosomes with structural rearrangements and simulated long reads for testing SV detection accuracy) — https://github.com/msjimc/MakeSVGenome

## Examples

```
minimap2 -ax map-ont --secondary=no hg19.fa SRR9982132.fastq > aln.sam && samtools sort -o aln.sorted.bam aln.sam && samtools index aln.sorted.bam
```

## Evaluation signals

- Called chr6 breakpoint falls within -1/+2 bp of g.167,281,717
- Called chr8 breakpoint falls within +9/+11 bp of g.113,696,089
- Breakpoint supported by 2 or more split reads with junction positions within 250 nucleotides of each other
- Annotation uses consensus of per-read Variant determination calls rather than a single read
- Saved translocation_annotation.txt records both breakpoint coordinates and supporting read information

## Limitations

- AgileStructure requires manual, user-guided exploration; breakpoints are validated by inspecting split reads rather than called fully automatically.
- Accuracy is measured relative to published positions with small offsets (up to ~11 bp here), so exact base-level identity is not guaranteed.
- On POSIX systems the program runs via WINE, adding a platform compatibility layer.

## Evidence

- [abstract] AgileStructure, a desktop application for locating and annotating large-scale genomic rearrangements using aligned long-read data: "We developed AgileStructure, a desktop application for locating and annotating large‑scale genomic rearrangements using aligned long‑read data."
- [abstract] user-guided exploration of breakpoint-spanning reads: "The software enables user‑guided exploration of breakpoint‑spanning reads, supporting accurate interpretation of complex events"
- [intro] long-read-specific aligner such as minimap2: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [results] AgileStructure t(chr6;chr8) breakpoint coordinates within reported offsets: [evidence span withheld — anchor status: not_found]
- [other] putative breakpoint called when 2 or more split reads have junction positions within 250 nucleotides: [evidence span withheld — anchor status: not_found]
- [readme] minimap2 map-ont preset for Oxford Nanopore genomic reads: [evidence span withheld — anchor status: external_doc]
