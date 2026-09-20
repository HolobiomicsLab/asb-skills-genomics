---
name: long-anchor-placement-on-reference-genome
description: Use when you have orphan insertion contigs assembled from unaligned/poorly-aligned
  linked reads and need to localize the insertion on a reference genome by reassembling
  barcode-linked reads into anchored contigs.
when_to_use_negative:
- Insertions are shorter than 300 bp — small insertions usually recruit too few distinct
  barcodes for sufficient anchor assembly coverage
- You need de novo whole-genome assembly rather than targeted local placement of non-reference
  insertions
- Data lacks barcode/long-fragment information (e.g. standard short-read WGS without
  linked-read structure)
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Novel-X
  role: Pipeline orchestrating barcode list extraction, reassembly, and anchor placement
    on the reference
  repo: github.com/1dayac/Novel-X
- name: SPAdes
  role: Reassembles reads carrying candidate barcodes (k=77, cov-cutoff=3) to build
    anchor--insertion contigs
  repo: github.com/1dayac/Novel-X
- name: Minimap2
  role: Aligns reassembled contigs to the reference genome; best match must correspond
    to the orphan contig
- name: Velvet
  role: Initial assembly of unaligned read set into orphan insertion contigs (k=63
    for 10x data)
  repo: https://github.com/dzerbino/velvet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:1618c150542432bd8d76ccf83da62de8bd7c25663f597306609fb330036363e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LongRanger
  - Minimap2
  - Novel-X
  - SPAdes
  - Velvet
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Long-anchor placement on reference genome

## Summary

Places reassembled left-anchor--insertion--right-anchor contigs onto the GRCh38 reference using Minimap2 alignment plus a QUAST-like best-alignment-subset filter, enabling precise positioning of non-reference sequence (NRS) insertions detected from synthetic long-read (10x Genomics) data without whole-genome de novo assembly.

## When to use

Use when you have orphan insertion contigs assembled from unaligned/poorly-aligned linked reads and need to localize the insertion on a reference genome by reassembling barcode-linked reads into anchored contigs.

## When NOT to use

- Insertions are shorter than 300 bp — small insertions usually recruit too few distinct barcodes for sufficient anchor assembly coverage
- You need de novo whole-genome assembly rather than targeted local placement of non-reference insertions
- Data lacks barcode/long-fragment information (e.g. standard short-read WGS without linked-read structure)

## Inputs

- LongRanger-aligned BAM files with barcode-annotated 10x Genomics paired-end reads
- Orphan insertion contigs from the initial Velvet assembly (k=63)
- Barcode lists extracted from filtered read alignments (per orphan contig)
- Reference genome FASTA (e.g. GRCh38)

## Outputs

- Reassembled left-anchor--insertion--right-anchor contigs (SPAdes, k=77)
- Per-sample VCFs of placed NRS insertions (>300 bp, anchor length sum >300 bp)
- Cohort-level merged VCF callset with aggregate statistics (insertion count, total length, longest insertion)

## How to apply

For each orphan contig assembled with Velvet (k=63, MAXKMERLENGTH=63 for 10x data), extract barcode lists from filtered read alignments (discarding alignments with >=20% soft/hard-clipped bases) and discard contigs recruiting fewer than t=5 barcodes. Reassemble all original reads carrying those barcodes per contig with SPAdes (k=77, cov-cutoff=3) to build left-anchor--insertion--right-anchor contigs, retaining only reassembled contigs whose best Minimap2 match corresponds to the orphan contig. Align reassembled contigs to the reference with Minimap2 and apply a QUAST-like filtering procedure that chooses best alignment subsets maximizing the number of continuously covered base pairs. Output only insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp, writing per-sample VCFs that can be merged into a cohort-level callset.

## Related tools

- **Novel-X** (Pipeline orchestrating barcode list extraction, reassembly, and anchor placement on the reference) — github.com/1dayac/Novel-X
- **SPAdes** (Reassembles reads carrying candidate barcodes (k=77, cov-cutoff=3) to build anchor--insertion contigs) — github.com/1dayac/Novel-X
- **Minimap2** (Aligns reassembled contigs to the reference genome; best match must correspond to the orphan contig)
- **Velvet** (Initial assembly of unaligned read set into orphan insertion contigs (k=63 for 10x data)) — https://github.com/dzerbino/velvet

## Examples

```
git clone --recursive git@github.com:1dayac/Novel-X.git
```

## Evaluation signals

- Merged 68-sample Polaris callset contains ~16,691 NRS insertions >300 bp with total length 18.2 Mb and longest insertion 85,908 bp
- Every retained insertion is longer than 300 bp and the sum of its two anchor lengths exceeds 300 bp
- Each reassembled contig's best Minimap2 match corresponds to the original orphan contig
- Reassembled contig alignments pass QUAST-like best-alignment-subset filtering that maximizes continuously covered base pairs
- Per-sample VCFs merge consistently into a single cohort-level VCF without duplicate/overlapping call collisions

## Limitations

- Novel-X needs a substantial number of distinct barcodes; small insertions often have too few associated barcodes and insufficient coverage for anchor assembly
- SuperNova assembler (used elsewhere in the pipeline) offers no user control over parameters, with defaults tuned for whole human genome diploid assembly
- Known pipeline issue on the filter_target_contig stage noted in the methods

## Evidence

- [methods] Insertions reassembly: [evidence span withheld — anchor status: not_found]
- [methods] Location of insertions: [evidence span withheld — anchor status: not_found]
- [methods] Barcode threshold: [evidence span withheld — anchor status: not_found]
- [methods] Output filter: [evidence span withheld — anchor status: not_found]
- [methods] QUAST-like filtering: "a filtering procedure that resembles the QUAST (37) procedure, i.e. choosing best alignment subsets from a given alignment set that maximizes the number of continuously covered base pairs"
- [abstract] Headline cohort result: "Novel-X discovered 16 691 NRS insertions of size > 300 bp (total length 18.2 Mb)."
