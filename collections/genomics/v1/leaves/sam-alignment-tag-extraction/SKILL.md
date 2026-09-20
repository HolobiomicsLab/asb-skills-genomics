---
name: sam-alignment-tag-extraction
description: Use when an indexed BAM of ONT or PacBio long reads aligned with minimap2
  (or lra) contains split reads with SA tags, and per-read junction coordinates must
  be extracted for breakpoint calling or downstream structural variant detection.
when_to_use_negative:
- Input is unaligned long-read data (FASTQ/POD5) rather than an indexed BAM — alignment
  with a long-read aligner must happen first
- The goal is short-read SNV or small indel calling, which the article notes does
  not resolve ~50% of rare disease pathogenic variants but is a different analysis
  task
- Reads were aligned with settings that suppress secondary/supplementary alignments
  needed for split-read junction extraction
edam_operation: http://edamontology.org/operation_2520
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: AgileStructure
  role: Desktop application that extracts split-read junction positions from aligned
    BAM data and clusters them to call breakpoints
  repo: https://github.com/msjimc/AgileStructure
- name: minimap2
  role: Long-read-specific aligner producing the BAM with primary and secondary split-read
    alignments whose tags are extracted
  repo: https://github.com/lh3/minimap2
- name: lra
  role: Alternative long-read aligner whose SAM output can substitute minimap2 for
    producing split-read alignments
  repo: https://github.com/ChaissonLab/LRA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:2c2f1505d0c7327a504771ccf0de11654fa65e0ecbec77563fe6e3e34052eeb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msjimc/AgileStructure
  tools:
  - AgileStructure
  - Samtools
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_2520
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# SAM alignment tag extraction

## Summary

Extracts the genomic junction positions between primary and secondary (SA-tagged) alignments of split reads from an indexed BAM file, as the raw signal AgileStructure uses to detect structural rearrangement breakpoints in long-read data.

## When to use

Use when an indexed BAM of ONT or PacBio long reads aligned with minimap2 (or lra) contains split reads with SA tags, and per-read junction coordinates must be extracted for breakpoint calling or downstream structural variant detection.

## When NOT to use

- Input is unaligned long-read data (FASTQ/POD5) rather than an indexed BAM — alignment with a long-read aligner must happen first
- The goal is short-read SNV or small indel calling, which the article notes does not resolve ~50% of rare disease pathogenic variants but is a different analysis task
- Reads were aligned with settings that suppress secondary/supplementary alignments needed for split-read junction extraction

## Inputs

- Indexed BAM file of long reads aligned with minimap2 (-ax map-ont, --secondary=no)
- SAM/BAM alignment records including primary alignments and secondary/supplementary alignments with linkage tags

## Outputs

- breakpoints.tsv with chromosome, position, and supporting read count for each called breakpoint

## How to apply

Open the indexed BAM file produced by aligning ONT long reads with minimap2 using the -ax map-ont and --secondary=no parameters, and read both the header and alignment sections to obtain each primary alignment and its linked secondary alignments. For every split read, extract the genomic positions where the primary alignment starts or stops and where its linked secondary alignments start or stop, retrieving these junction coordinates from the alignment records and their tag data (e.g. the SA tag linking supplementary alignments). Cluster the extracted junction positions at the locus of interest; a putative breakpoint is called when 2 or more junction positions fall within 250 nucleotides of each other. Write the chromosome, position, and supporting read count for each called breakpoint to a TSV file. The rationale is that genuine rearrangements produce reproducible split-read junctions at the same locus, whereas sequencing noise produces isolated, unclustered junction positions.

## Related tools

- **AgileStructure** (Desktop application that extracts split-read junction positions from aligned BAM data and clusters them to call breakpoints) — https://github.com/msjimc/AgileStructure
- **minimap2** (Long-read-specific aligner producing the BAM with primary and secondary split-read alignments whose tags are extracted) — https://github.com/lh3/minimap2
- **lra** (Alternative long-read aligner whose SAM output can substitute minimap2 for producing split-read alignments) — https://github.com/ChaissonLab/LRA

## Examples

```
minimap2 -ax map-ont --secondary=no ref.fa ont.fq.gz | samtools view -h -o aln.bam; samtools index aln.bam
```

## Evaluation signals

- breakpoints.tsv exists and is a well-formed TSV with chromosome/position coordinates and support-count fields
- Every reported breakpoint is supported by at least 2 junction positions within a 250-nucleotide window
- Extracted junction positions correspond to primary-to-secondary alignment boundaries recorded in the alignment records and tag data
- Called breakpoints are consistent with the expected rearrangement coordinates in synthetic test data generated by MakeSVGenome

## Limitations

- The BAM must retain secondary/supplementary alignment information (align with --secondary=no suppresses secondary records but the SA-linked alignments must still be parseable from records and tags)
- Detection depends on the 2-junctions-within-250-nt clustering rule; single isolated junction positions are not called as breakpoints
- AgileStructure is a Windows application, so on POSIX systems it must be run via WINE

## Evidence

- [other] AgileStructure retains the genomic positions of junctions between the primary and secondary alignments of split reads: [evidence span withheld — anchor status: not_found]
- [abstract] We developed AgileStructure: [evidence span withheld — anchor status: not_found]
- [intro] the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
