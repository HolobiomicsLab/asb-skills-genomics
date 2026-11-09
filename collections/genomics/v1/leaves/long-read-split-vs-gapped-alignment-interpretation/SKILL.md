---
name: long-read-split-vs-gapped-alignment-interpretation
description: Use when long-read SAM/BAM alignments contain gapped CIGAR strings (I/D
  operations >10 bp) at a locus of interest and these within-read indels must be extracted,
  coordinated, and annotated rather than relying on split-read (N/soft-clip) breakpoints.
when_to_use_negative:
- The variant is already reported as split reads or separate supplementary alignments
  — handle those with breakpoint-based methods, not CIGAR-gap parsing
- The gapped CIGAR indels are ≤10 bp — AgileStructure only displays within-read indels
  larger than 10 bp via the 'Look for indels within a read' option
- The input is unaligned FASTQ/BAM without CIGAR strings — alignment with minimap2
  or lra must happen first
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: AgileStructure
  role: Reference implementation whose 'Look for indels within a read' option and
    median-coordinate reporting rule the CIGAR-gap parsing replicates
  repo: https://github.com/msjimc/AgileStructure
- name: minimap2
  role: Long-read-specific aligner producing the CIGAR strings parsed for within-read
    insertions and deletions
  repo: https://github.com/lh3/minimap2
- name: lra
  role: Alternative long-read aligner whose gap-aware alignments increase sensitivity
    for SVs above 1 kb, particularly from ONT reads
  repo: https://github.com/ChaissonLab/LRA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:2cf67813a9526c838c4e8d240e1bef641d4c66627cda288ec62128d7b20ed99e
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Long-read split vs gapped alignment interpretation

## Summary

Distinguishes structural variants reported by long-read aligners as split reads from those encoded as large CIGAR gaps (I/D operations) within a single read alignment, and derives per-read indel coordinates for annotation. This mirrors how AgileStructure recovers within-read indels (e.g. a ~38,440 bp insertion varying by only 8 bp across reads) that split-read workflows would miss.

## When to use

Use when long-read SAM/BAM alignments contain gapped CIGAR strings (I/D operations >10 bp) at a locus of interest and these within-read indels must be extracted, coordinated, and annotated rather than relying on split-read (N/soft-clip) breakpoints.

## When NOT to use

- The variant is already reported as split reads or separate supplementary alignments — handle those with breakpoint-based methods, not CIGAR-gap parsing
- The gapped CIGAR indels are ≤10 bp — AgileStructure only displays within-read indels larger than 10 bp via the 'Look for indels within a read' option
- The input is unaligned FASTQ/BAM without CIGAR strings — alignment with minimap2 or lra must happen first

## Inputs

- SAM/BAM file of long reads aligned with minimap2 (-ax map-ont --secondary=no) or lra
- Per-read CIGAR strings spanning the target locus
- Reference genome build used for the alignment

## Outputs

- cigar_indel_annotations.tsv with columns: variant type, median start, median end, median length
- Per-read list of reported indel start/end positions and lengths supporting each event

## How to apply

Load the minimap2-aligned long-read data (aligned with `-ax map-ont --secondary=no`) at the target locus and extract the CIGAR string of each spanning read. Parse each CIGAR as a series of consecutive local alignments ordered along the read sequence, so that only CIGAR-encoded insertions and deletions (not split-read junctions) become candidate events. Filter candidates to indels larger than 10 bp, the threshold AgileStructure applies for its 'Look for indels within a read' display option. For each candidate, collect the reported start position, end position, and length from every supporting read, then report the MEDIAN of these per-read values in the final annotation, mirroring AgileStructure's median-based reporting rule. Write the annotations to a TSV with columns for variant type, median start, median end, and median length; validate the length against a reference value (e.g. the ~38,440 bp insertion that varied by only 8 bp across reads).

## Related tools

- **AgileStructure** (Reference implementation whose 'Look for indels within a read' option and median-coordinate reporting rule the CIGAR-gap parsing replicates) — https://github.com/msjimc/AgileStructure
- **minimap2** (Long-read-specific aligner producing the CIGAR strings parsed for within-read insertions and deletions) — https://github.com/lh3/minimap2
- **lra** (Alternative long-read aligner whose gap-aware alignments increase sensitivity for SVs above 1 kb, particularly from ONT reads) — https://github.com/ChaissonLab/LRA

## Examples

```
minimap2 -ax map-ont --secondary=no ref.fa ont_reads.fq.gz > aln.sam
```

## Evaluation signals

- Each reported indel coordinate equals the median of per-read reported start and end values (median-based reporting rule)
- The insertion length matches the reference value within read-to-read variation (e.g. ~38,440 bp varying by only 8 bp)
- All extracted events are CIGAR-encoded I/D operations, not split-read junctions
- Every annotated indel exceeds the 10 bp threshold used by AgileStructure's 'Look for indels within a read' display
- Output TSV contains the four required columns (variant type, median start, median end, median length)

## Limitations

- Only CIGAR-encoded insertions and deletions within a single read alignment are candidates; split-read breakpoints require a different interpretation path
- AgileStructure displays within-read indels only above the 10 bp threshold, so smaller gapped events are not annotated
- Median reporting smooths per-read coordinate differences; per-read start/end offsets must be retained for validation (e.g. the 8 bp spread around ~38,440 bp)

## Evidence

- [other] For gapped-read indels, AgileStructure identifies the reported start and end points in each selected read and uses the median of these per-read values in the variant annotation: [evidence span withheld — anchor status: not_found]
- [readme] the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [other] Parse each CIGAR string to describe the global alignment as a series of consecutive local alignments: [evidence span withheld — anchor status: not_found]
- [readme] lra increases sensitivity and specificity for SV discovery, particularly for variants above 1kb: [evidence span withheld — anchor status: external_doc]
