---
name: gencode-gene-overlap-intersection
description: Use when a VCF of non-reference insertion calls needs functional/genomic
  context annotation — e.g. counting how many insertion breakpoints overlap GENCODE-annotated
  genes vs exons of protein-coding genes.
when_to_use_negative:
- Input is raw reads (FASTQ/BAM) or a Novel-X VCF has not yet been generated — the
  intersection operates on placed breakpoint positions, not alignments
- Goal is to annotate the inserted (non-reference) sequence content itself (e.g. repeat
  screening via RepeatMasker or mappability via Umap) — those use the 300 bp flanking
  sequences, not gene annotation
- Calls are shorter than 300 bp or lack placed anchors, since Novel-X only reports
  insertions >300 bp with anchor-length sum exceeding 300 bp
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Novel-X
  role: Produces the VCF of non-reference insertions whose breakpoint positions are
    intersected with GENCODE annotation
  repo: github.com/1dayac/Novel-X
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:97dff144a33aa0b8572eb81d14d62e6b4e02d4c4218105ca0e248bc9212d7a1f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LongRanger
  - Novel-X
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gencode-gene-overlap-intersection

## Summary

Characterizes called non-reference sequence insertions by intersecting their reference-genome breakpoint positions with GENCODE gene annotation, quantifying how many calls fall inside known genes and, specifically, exons of protein-coding genes.

## When to use

Use when a VCF of non-reference insertion calls needs functional/genomic context annotation — e.g. counting how many insertion breakpoints overlap GENCODE-annotated genes vs exons of protein-coding genes.

## When NOT to use

- Input is raw reads (FASTQ/BAM) or a Novel-X VCF has not yet been generated — the intersection operates on placed breakpoint positions, not alignments
- Goal is to annotate the inserted (non-reference) sequence content itself (e.g. repeat screening via RepeatMasker or mappability via Umap) — those use the 300 bp flanking sequences, not gene annotation
- Calls are shorter than 300 bp or lack placed anchors, since Novel-X only reports insertions >300 bp with anchor-length sum exceeding 300 bp

## Inputs

- VCF of non-reference sequence insertion calls from Novel-X (records >300 bp with anchors placed on GRCh38)
- GENCODE gene annotation GTF (e.g. release v.24 for GRCh38)

## Outputs

- Per-insertion genic/exonic overlap classification
- Summary counts: number of insertions overlapping known genes and number overlapping exons of protein-coding genes (e.g. 111 and 2 of 226 for CHM1)

## How to apply

Take the Novel-X output VCF (insertions longer than 300 bp with anchor-length sums exceeding 300 bp, placed on the GRCh38 reference via Minimap2 anchors) and extract each record's breakpoint position on the reference. Intersect these breakpoint coordinates against GENCODE annotation (v.24 in the CHM1 benchmark) restricted to the reference-side flanks, not the inserted sequence, since the inserted non-reference DNA has no gene annotation by definition. Classify hits in two tiers: insertions falling inside any known gene sequence (genic) and the stricter subset inside exonic regions of known protein-coding genes. Report both counts against the total number of VCF records (in the paper's CHM1 10x run, 111 of 226 insertions overlapped genes, with only 2 in exons of protein-coding genes), which distinguishes insertions mostly landing in intronic/intergenic space from those likely to disrupt coding sequence.

## Related tools

- **Novel-X** (Produces the VCF of non-reference insertions whose breakpoint positions are intersected with GENCODE annotation) — github.com/1dayac/Novel-X

## Evaluation signals

- VCF record count matches expectation (226 insertion records for the CHM1 10x run at 0.8 downsampled coverage)
- Gene-overlap count reproduces the published value: 111 of 226 insertions inside known gene sequences
- Exonic protein-coding count reproduces the published value: only 2 insertions in exons of known protein-coding genes
- Every intersected breakpoint coordinate lies within GRCh38 reference coordinates (consistent with Minimap2 anchor placement)
- Intersections are computed on reference-side breakpoints/flanks only — no overlap calls should be attributed to the non-reference inserted sequence

## Limitations

- Analysis restricted to insertions longer than 300 bp; small insertions are excluded, so genic/exonic fractions describe only large NRS insertions
- Breakpoint accuracy depends on Minimap2 anchor placement; imprecise anchors can misassign genic/exonic status
- Overlap with an annotated gene does not imply the inserted sequence is gene-homologous — it only indicates the insertion lands within gene coordinates on the reference

## Evidence

- [results] Intersect the reported breakpoint positions with GENCODE annotation v.24 and verify that 111 of 226 insertions fall inside known gene sequences, with only two in exonic regions of known protein-coding genes.: [evidence span withheld — anchor status: not_found]
- [methods] We only output insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp to prevent false calls: [evidence span withheld — anchor status: not_found]
- [readme] Results folder contains all vcf files that we used for benchmarks that generated Figures 4-5 and Tables S1-S3 from the article.: [evidence span withheld — anchor status: external_doc]
