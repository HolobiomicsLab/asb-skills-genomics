---
name: structural-variant-length-quantification
description: Reference for deriving indel coordinates and lengths from gapped long-read
  alignments (SAM/BAM CIGAR strings from minimap2 or lra) when the variant is not
  represented as a split read and a consensus per-read median annotation is required.
when_to_use_negative:
- The variant is represented as split reads with separate alignment blocks rather
  than as gaps encoded within a single read's CIGAR string
- The alignment was produced without a CIGAR (e.g. minimap2 default PAF output without
  '-a' or '-c'), since no CIGAR-encoded indels can be parsed
- Small variants (<=10 bp indels or SNVs) are of interest, as the 10 bp threshold
  excludes them by design
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: AgileStructure
  role: Reference implementation whose median-based gapped-read indel reporting rule
    ('Look for indels within a read') this skill reconstructs
  repo: https://github.com/msjimc/AgileStructure
- name: minimap2
  role: Long-read-specific aligner producing the SAM/BAM alignments whose CIGAR strings
    are parsed for gapped indels
  repo: https://github.com/lh3/minimap2
- name: lra
  role: Alternative long-read aligner whose gapped alignments can substitute for minimap2
    output
  repo: https://github.com/ChaissonLab/LRA
- name: MakeSVGenome
  role: Generator of synthetic chromosomes with structural rearrangements and simulated
    long reads for testing indel-detection accuracy
  repo: https://github.com/msjimc/MakeSVGenome
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:d47d25d4ce137ad0c464d659e8a45195a901c60505a6f5627ea0ca75ec0a1de9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msjimc/AgileStructure
  tools:
  - AgileStructure
  - MakeSVGenome
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# structural-variant-length-quantification

## Summary

Reconstructs AgileStructure's handling of indels reported as CIGAR gaps in long-read alignments rather than split reads, parsing per-read CIGAR-encoded insertion/deletion start, end, and length values and reporting each variant with the median across supporting reads.

## When to use

Reference for deriving indel coordinates and lengths from gapped long-read alignments (SAM/BAM CIGAR strings from minimap2 or lra) when the variant is not represented as a split read and a consensus per-read median annotation is required.

## When NOT to use

- The variant is represented as split reads with separate alignment blocks rather than as gaps encoded within a single read's CIGAR string
- The alignment was produced without a CIGAR (e.g. minimap2 default PAF output without '-a' or '-c'), since no CIGAR-encoded indels can be parsed
- Small variants (<=10 bp indels or SNVs) are of interest, as the 10 bp threshold excludes them by design

## Inputs

- SAM/BAM alignments of long reads (PacBio or ONT) produced by minimap2 (-ax map-ont --secondary=no) or lra
- CIGAR strings of reads spanning the target locus
- Reference genome sequence used for alignment

## Outputs

- cigar_indel_annotations.tsv with columns for variant type, median start, median end, and median length
- Per-read lists of reported indel start, end, and length values used for median computation

## How to apply

Align long-read data (PacBio or ONT) to the reference using a long-read-specific aligner such as minimap2 with '-ax map-ont --secondary=no', then extract the CIGAR strings of reads spanning the target locus. Parse each CIGAR to describe the global alignment as a series of consecutive local alignments ordered as they appear in the read, so only CIGAR-encoded insertions and deletions become candidates. Filter candidates to indels larger than 10 bp, matching AgileStructure's 'Look for indels within a read' display threshold. For each candidate, collect the reported start, end, and length from every supporting read (e.g. the ~38,440 bp insertion whose length varied by only 8 bp across reads) and annotate the variant using the median of the per-read values, mirroring AgileStructure's median-based reporting rule. Write results as a TSV with variant type, median start, median end, and median length.

## Related tools

- **AgileStructure** (Reference implementation whose median-based gapped-read indel reporting rule ('Look for indels within a read') this skill reconstructs) — https://github.com/msjimc/AgileStructure
- **minimap2** (Long-read-specific aligner producing the SAM/BAM alignments whose CIGAR strings are parsed for gapped indels) — https://github.com/lh3/minimap2
- **lra** (Alternative long-read aligner whose gapped alignments can substitute for minimap2 output) — https://github.com/ChaissonLab/LRA
- **MakeSVGenome** (Generator of synthetic chromosomes with structural rearrangements and simulated long reads for testing indel-detection accuracy) — https://github.com/msjimc/MakeSVGenome

## Examples

```
minimap2 -ax map-ont --secondary=no ref.fa ont_reads.fq > aln.sam
```

## Evaluation signals

- Reported insertion length should be close to the reference value (~38,440 bp, which varied by only 8 bp across reads)
- Each annotated start and end coordinate must equal the median of the per-read reported values, satisfying an output-matches-reference check
- No annotated indel is smaller than the 10 bp display threshold
- Parsed events derive exclusively from CIGAR I/D operations; no split-read junctions are counted as gapped indels
- Output TSV contains exactly the columns variant type, median start, median end, and median length

## Limitations

- Only CIGAR-encoded insertions and deletions are candidates; variants aligners report as split reads follow a different pathway
- Indels of 10 bp or smaller are excluded, matching AgileStructure's display threshold rather than exhaustive small-variant calling
- Median-based annotation depends on a sufficient number of supporting reads; sparse read coverage may yield unstable coordinates

## Evidence

- [abstract] We developed AgileStructure, a desktop application for locating and annotating large-scale genomic rearrangements using aligned long-read data.: [evidence span withheld — anchor status: not_found]
- [intro] the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [readme] long-read sequencing data generated by PacBio or ONT sequencers has been increasingly used: "long-read sequencing data generated by PacBio or ONT sequencers has been increasingly used"
- [full_text] identifies the reported start and end points in each selected read and uses the median of these per-read values in the variant annotation: [evidence span withheld — anchor status: not_found]
