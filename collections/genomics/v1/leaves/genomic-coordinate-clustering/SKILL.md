---
name: genomic-coordinate-clustering
description: Use when you have an indexed BAM of long reads (ONT/PacBio) aligned with
  minimap2 or lra and need to call putative structural rearrangement breakpoints from
  split-read junction positions at a target locus.
when_to_use_negative:
- Input reads are short-read Illumina data — the article states large-scale structural
  rearrangements are difficult to detect with short-read data and AgileStructure is
  built for aligned long-read data
- The BAM was produced without secondary/supplementary alignment records (e.g. aligner
  settings that suppress split alignments), since no junction positions can be extracted
- The goal is de novo genome-wide SV discovery without a user-selected locus — AgileStructure
  is designed for user-guided exploration of breakpoint-spanning reads at loci of
  interest
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: AgileStructure
  role: Desktop application that extracts split-read junction positions from the indexed
    BAM, clusters them within 250 nucleotides, and writes called breakpoints to breakpoints.tsv
  repo: https://github.com/msjimc/AgileStructure
- name: minimap2
  role: Long-read-specific aligner used with -ax map-ont and --secondary=no to produce
    the primary and linked secondary split-read alignments in the input BAM
  repo: https://github.com/lh3/minimap2
- name: lra
  role: Alternative long-read aligner that can generate the aligned BAM input for
    breakpoint clustering
  repo: https://github.com/ChaissonLab/LRA
- name: MakeSVGenome
  role: Generates synthetic chromosomes with structural rearrangements and synthetic
    long-read data for testing breakpoint detection accuracy
  repo: msjimc/MakeSVGenome
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:8d23754cec54f8a985a0c36d20e5e911415cd04434d24c173e125d8e85b5b1cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msjimc/AgileStructure
  tools:
  - AgileStructure
  - MakeSVGenome
  - Samtools
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# genomic-coordinate-clustering

## Summary

Clusters split-read genomic junction positions from aligned long-read BAM data to identify putative structural variant breakpoints, using AgileStructure's criterion of 2 or more junction positions falling within 250 nucleotides of each other.

## When to use

Use when you have an indexed BAM of long reads (ONT/PacBio) aligned with minimap2 or lra and need to call putative structural rearrangement breakpoints from split-read junction positions at a target locus.

## When NOT to use

- Input reads are short-read Illumina data — the article states large-scale structural rearrangements are difficult to detect with short-read data and AgileStructure is built for aligned long-read data
- The BAM was produced without secondary/supplementary alignment records (e.g. aligner settings that suppress split alignments), since no junction positions can be extracted
- The goal is de novo genome-wide SV discovery without a user-selected locus — AgileStructure is designed for user-guided exploration of breakpoint-spanning reads at loci of interest

## Inputs

- Indexed BAM file of long reads aligned to a reference genome (e.g. minimap2 -ax map-ont --secondary=no output)
- Target genomic locus or chromosome region to scan for breakpoints

## Outputs

- breakpoints.tsv: TSV with breakpoint chromosome, position, and supporting read count per called breakpoint

## How to apply

Align ONT long reads to the reference genome with minimap2 using the -ax map-ont and --secondary=no parameters, then open the indexed BAM file in AgileStructure, which reads the header and alignment records to obtain primary alignments and their linked secondary alignments. For each split read, extract the genomic positions where the primary alignment starts or stops and where its linked secondary alignment starts or stops (retrieved from alignment records and tag data), since these mark the junctions between the primary and secondary alignments. Cluster the junction positions at the target locus and call a putative breakpoint when 2 or more junction positions fall within 250 nucleotides of each other. Write each called breakpoint's chromosome, position, and supporting read count to breakpoints.tsv for downstream annotation and user-guided inspection of breakpoint-spanning reads.

## Related tools

- **AgileStructure** (Desktop application that extracts split-read junction positions from the indexed BAM, clusters them within 250 nucleotides, and writes called breakpoints to breakpoints.tsv) — https://github.com/msjimc/AgileStructure
- **minimap2** (Long-read-specific aligner used with -ax map-ont and --secondary=no to produce the primary and linked secondary split-read alignments in the input BAM) — https://github.com/lh3/minimap2
- **lra** (Alternative long-read aligner that can generate the aligned BAM input for breakpoint clustering) — https://github.com/ChaissonLab/LRA
- **MakeSVGenome** (Generates synthetic chromosomes with structural rearrangements and synthetic long-read data for testing breakpoint detection accuracy) — msjimc/MakeSVGenome

## Examples

```
minimap2 -ax map-ont --secondary=no ref.fa ont_reads.fq | samtools sort -o aln.bam && samtools index aln.bam  # then open aln.bam in AgileStructure to cluster junctions and export breakpoints.tsv
```

## Evaluation signals

- breakpoints.tsv exists and is a well-formed TSV with coordinate and support-count fields
- Every reported breakpoint is supported by at least 2 junction positions falling within the 250-nucleotide window
- Each split read's junction positions match the start/stop coordinates of its primary alignment and its linked secondary alignment in the BAM records and tag data
- Called breakpoints are consistent with junction evidence visible when breakpoint-spanning reads are inspected in a genome browser such as IGV
- Using synthetic reads from MakeSVGenome with known rearrangements, called breakpoint coordinates match the ground-truth SV positions

## Limitations

- Requires at least 2 split-read junction positions within 250 nucleotides, so single-read (singleton) breakpoints are not called
- Designed for large-scale structural rearrangements detectable with long reads; approximately 50% of rare-disease patients' variants remain unresolved with short-read data and this skill targets precisely that long-read SV regime, not SNVs or small indels
- Runs on most POSIX-like systems with Intel or AMD CPUs via WINE, so native deployment on non-Windows platforms depends on the WINE compatibility layer

## Evidence

- [methods] AgileStructure retains the genomic positions of junctions between the primary and secondary alignments of split reads, and identifies a putative breakpoint when 2 or more of these positions occur within 250 nucleotides of each other.: [evidence span withheld — anchor status: not_found]
- [abstract] We developed AgileStructure, a desktop application for locating and annotating large-scale genomic rearrangements using aligned long-read data.: [evidence span withheld — anchor status: not_found]
- [intro] the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [abstract] Source code, binaries, user guide, and example aligned read data, are available on GitHub: https://github.com/msjimc/AgileStructure.: "Source code, binaries, user guide, and example aligned read data, are available on GitHub"
- [readme] MakeSVGenome is designed to create chromosomes that contain structural rearrangements and synthetic long read sequencing data that can then be used to test the accuracy of structural variant detection programs or pipe lines.: [evidence span withheld — anchor status: external_doc]
