---
name: median-coordinate-variant-annotation
description: Use when long reads (PacBio/ONT) aligned with minimap2 or lra contain
  CIGAR-encoded indels rather than split-read breakpoints, and a consensus indel coordinate/length
  must be reported across supporting reads.
when_to_use_negative:
- The structural variant is reported as split reads (separate supplementary alignments)
  rather than CIGAR string gaps — use split-read breakpoint handling instead
- Indels are small (≤10 bp), below the threshold AgileStructure displays for indels
  within a read
- Input is unaligned FASTQ/FASTA reads with no CIGAR information available
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3168
tools:
- name: AgileStructure
  role: Desktop application that identifies reported start/end points of gapped-read
    indels in each selected read and annotates the variant with the median of these
    per-read values
  repo: https://github.com/msjimc/AgileStructure
- name: minimap2
  role: Long-read-specific aligner producing the SAM/BAM alignments whose CIGAR strings
    encode the indel gaps
  repo: https://github.com/lh3/minimap2
- name: lra
  role: Alternative long-read aligner whose gapped alignments can supply CIGAR-encoded
    indels for annotation
  repo: https://github.com/ChaissonLab/LRA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:0d588435db646ecf8d7f2a1d742f02809a776bc6376362fade3d484cb5619068
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msjimc/AgileStructure
  tools:
  - AgileStructure
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# median-coordinate-variant-annotation

## Summary

Annotates structural-variant indels that aligners encode as CIGAR string gaps (I/D operations) in long reads by collecting each supporting read's reported start, end, and length and reporting the median of the per-read values, as AgileStructure does for gapped-read indels.

## When to use

Use when long reads (PacBio/ONT) aligned with minimap2 or lra contain CIGAR-encoded indels rather than split-read breakpoints, and a consensus indel coordinate/length must be reported across supporting reads.

## When NOT to use

- The structural variant is reported as split reads (separate supplementary alignments) rather than CIGAR string gaps — use split-read breakpoint handling instead
- Indels are small (≤10 bp), below the threshold AgileStructure displays for indels within a read
- Input is unaligned FASTQ/FASTA reads with no CIGAR information available

## Inputs

- SAM/BAM alignments of long reads (PacBio or ONT) against a reference genome, with CIGAR strings containing I/D gap operations
- Reference genome FASTA for coordinate extraction

## Outputs

- Indel annotations with variant type and median per-read start coordinate, end coordinate, and length (e.g. a cigar_indel_annotations.tsv table)
- AgileStructure display of individual CIGAR-gapped indels >10 bp via 'Look for indels within a read'

## How to apply

Align ONT or PacBio long reads to the reference with a long-read-specific aligner (e.g. minimap2 with the map-ont preset and --secondary=no, or lra) and load the SAM/BAM in AgileStructure. Parse each CIGAR string as a series of consecutive local alignments ordered as in the read sequence so that only CIGAR-encoded I and D operations are candidate indels, not split-read breakpoints. Filter candidates to indels larger than 10 bp, the threshold AgileStructure uses for its 'Look for indels within a read' display option. For each candidate, collect the reported start position, end position, and length from every supporting read, then annotate the variant using the median of the per-read start and end points and lengths rather than a single read's values.

## Related tools

- **AgileStructure** (Desktop application that identifies reported start/end points of gapped-read indels in each selected read and annotates the variant with the median of these per-read values) — https://github.com/msjimc/AgileStructure
- **minimap2** (Long-read-specific aligner producing the SAM/BAM alignments whose CIGAR strings encode the indel gaps) — https://github.com/lh3/minimap2
- **lra** (Alternative long-read aligner whose gapped alignments can supply CIGAR-encoded indels for annotation) — https://github.com/ChaissonLab/LRA

## Examples

```
minimap2 -ax map-ont --secondary=no ref.fa ont_reads.fq > aln.sam
```

## Evaluation signals

- Each reported start and end coordinate equals the median of the per-read reported start and end points across all supporting reads
- Annotated indel length is the median of per-read lengths and agrees within a few base pairs across reads (e.g. the paper's ~38,440 bp insertion varied by only 8 bp)
- Only indels larger than 10 bp are annotated, matching the 'Look for indels within a read' display threshold
- All candidate events derive from CIGAR I/D operations, not from split (supplementary) alignments
- Alignments were generated with long-read-appropriate settings (e.g. minimap2 -ax map-ont --secondary=no)

## Limitations

- Indels ≤10 bp are not displayed or annotated via the 'Look for indels within a read' option
- Median coordinates depend on the aligner's per-read reporting; different aligners (minimap2 vs lra) may yield different per-read start/end points
- The approach applies only to gapped alignments, not split-read representations of structural variants

## Evidence

- [abstract] AgileStructure is a desktop application for locating and annotating large-scale genomic rearrangements using aligned long-read data: "We developed AgileStructure, a desktop application for locating and annotating large‑scale genomic rearrangements using aligned long‑read data."
- [readme] Data is aligned with a long-read-specific aligner such as minimap2: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [readme] Long reads from PacBio or ONT sequencers are used to find large-scale rearrangements unresolved by short reads: "long-read sequencing data generated by PacBio or ONT sequencers has been increasingly used in the hope of identifying these large-scale rearrangements"
