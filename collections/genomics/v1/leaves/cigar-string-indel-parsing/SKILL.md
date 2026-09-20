---
name: cigar-string-indel-parsing
description: Use when SAM/BAM alignments from a long-read aligner (minimap2 -ax map-ont,
  lra) contain CIGAR-encoded insertions/deletions at a target locus and you need per-read
  coordinates aggregated into a median-based indel annotation.
when_to_use_negative:
- Structural variants are represented as split reads or separate supplementary alignments
  rather than gapped CIGAR strings — this skill only handles CIGAR-encoded I/D events
- Indels are 10 bp or smaller — below the 'Look for indels within a read' display
  threshold used by AgileStructure
- Reads were aligned with an aligner/preset whose output lacks explicit CIGAR strings
  (e.g. minimap2 without -a/-c), so per-read I/D coordinates cannot be extracted
edam_operation: http://edamontology.org/operation_0256
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: AgileStructure
  role: Reference implementation whose median-of-per-read reporting rule and >10 bp
    'Look for indels within a read' threshold this parsing skill reproduces
  repo: https://github.com/msjimc/AgileStructure
- name: minimap2
  role: Long-read-specific aligner producing the SAM/BAM alignments and CIGAR strings
    parsed for indel candidates (map-ont preset, --secondary=no)
  repo: https://github.com/lh3/minimap2
- name: lra
  role: Alternative long-read aligner whose gapped alignments can substitute minimap2
    output as the CIGAR source
  repo: https://github.com/ChaissonLab/LRA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:79e0c57de98c989951bfc09a7d21b9247c28cb87fdaa49adcc126963b7232b1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msjimc/AgileStructure
  tools:
  - AgileStructure
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_0256
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cigar-string-indel-parsing

## Summary

Reconstructs AgileStructure's method for identifying indels that long-read aligners (minimap2, lra) report as CIGAR I/D operations within a single gapped read rather than as split reads, and annotates each variant using the median of per-read reported start, end, and length values.

## When to use

Use when SAM/BAM alignments from a long-read aligner (minimap2 -ax map-ont, lra) contain CIGAR-encoded insertions/deletions at a target locus and you need per-read coordinates aggregated into a median-based indel annotation.

## When NOT to use

- Structural variants are represented as split reads or separate supplementary alignments rather than gapped CIGAR strings — this skill only handles CIGAR-encoded I/D events
- Indels are 10 bp or smaller — below the 'Look for indels within a read' display threshold used by AgileStructure
- Reads were aligned with an aligner/preset whose output lacks explicit CIGAR strings (e.g. minimap2 without -a/-c), so per-read I/D coordinates cannot be extracted

## Inputs

- SAM/BAM file of long reads aligned to a reference genome (e.g. minimap2 -ax map-ont --secondary=no output)
- CIGAR strings of reads overlapping the target structural-variant locus
- Per-read reported start position, end position, and indel length values

## Outputs

- cigar_indel_annotations.tsv with columns: variant type, median start, median end, median length
- Median-coordinate indel annotations (start, end, length) per CIGAR-gapped variant

## How to apply

Align the long-read data with minimap2 using the -ax map-ont and --secondary=no parameters (as in the AgileStructure example data) and extract the CIGAR strings of reads spanning the target locus. Parse each CIGAR string as a series of consecutive local alignments whose order reflects their order in the read sequence, so that only CIGAR-encoded insertions (I) and deletions (D) become candidate events — split-read variants are explicitly excluded. Filter candidates to indels larger than 10 bp, matching AgileStructure's 'Look for indels within a read' display threshold. For each candidate, collect the reported start position, end position, and length from every supporting read and annotate the variant with the median of these per-read values, mirroring AgileStructure's median-based reporting rule; write results to cigar_indel_annotations.tsv with variant type, median start, median end, and median length columns.

## Related tools

- **AgileStructure** (Reference implementation whose median-of-per-read reporting rule and >10 bp 'Look for indels within a read' threshold this parsing skill reproduces) — https://github.com/msjimc/AgileStructure
- **minimap2** (Long-read-specific aligner producing the SAM/BAM alignments and CIGAR strings parsed for indel candidates (map-ont preset, --secondary=no)) — https://github.com/lh3/minimap2
- **lra** (Alternative long-read aligner whose gapped alignments can substitute minimap2 output as the CIGAR source) — https://github.com/ChaissonLab/LRA

## Examples

```
minimap2 -ax map-ont --secondary=no ref.fa ont_reads.fq > aln.sam; python parse_cigar_indels.py --bam aln.sam --min-length 10 --out cigar_indel_annotations.tsv
```

## Evaluation signals

- Annotated insertion length matches the reference ~38,440 bp value (which varied by only 8 bp across supporting reads)
- Every annotated start and end coordinate equals the median of the per-read reported values (output-matches-reference check)
- Candidate set contains only events from CIGAR I/D operations — no split-read or supplementary-alignment events leak in
- All retained events exceed the 10 bp threshold
- cigar_indel_annotations.tsv has exactly the four expected columns: variant type, median start, median end, median length

## Limitations

- Only indels larger than 10 bp can be surfaced/displayed, per AgileStructure's 'Look for indels within a read' option
- Handles only gapped-read CIGAR indels; structural variants aligners report as split reads require a different approach
- Secondary alignments must be excluded (--secondary=no) so the same read does not contribute duplicate or conflicting per-read coordinates

## Evidence

- [abstract] CIGAR-based indel identification mirrors AgileStructure: [evidence span withheld — anchor status: not_found]
- [intro] Aligner choice produces the CIGAR input: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [readme] README documents the map-ont preset used for the aligned read data: [evidence span withheld — anchor status: external_doc]
