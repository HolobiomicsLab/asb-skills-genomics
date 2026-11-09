---
name: linked-read-barcode-extraction
description: Use when working with linked-read (10x Chromium, TELL-Seq, stLFR) BAM
  data where orphan contigs from unaligned reads must be linked to molecule barcodes
  before targeted reassembly of non-reference insertions.
when_to_use_negative:
- Input data lacks synthetic long-read barcode information (plain short-read or long-read
  data without linked-read barcodes) — barcode recruitment is impossible
- The goal is whole-genome de novo assembly; Novel-X's barcode-guided local reassembly
  is explicitly designed to avoid that
- Detecting insertions small enough that few distinct barcodes associate with them
  (the authors note small insertions usually have insufficient barcode coverage for
  anchor assembly)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Novel-X
  role: 'Snakemake pipeline implementing the barcode list extraction step: aligns
    reads in U to each orphan contig, filters alignments, collects barcodes B(c),
    and enforces the t=5 barcode threshold'
  repo: https://github.com/1dayac/Novel-X
- name: Velvet
  role: Assembles the unaligned read set U into orphan contigs (k-mer length 63, MAXKMERLENGTH=63
    for 10x data) that barcode extraction operates on
  repo: https://github.com/dzerbino/velvet
- name: SPAdes
  role: 'Downstream consumer of the extracted barcode lists: reassembles reads with
    barcodes in B(c) per barcode set (k=77, cov-cutoff=3) to obtain anchors'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:7ddc9d68ad4bd9131124023ba7d0705fc2d94ec4e8b9a81476c82cbe77b3ef83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Novel-X
  tools:
  - BLASTN
  - Novel-X
  - SPAdes
  - Samtools
  - Velvet
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# linked-read-barcode-extraction

## Summary

Extracts the set of 10x/stLFR/TELL-Seq synthetic long-read barcodes associated with candidate non-reference insertion contigs, by aligning unaligned-end reads to orphan contigs and collecting the barcodes of supporting reads. This barcode recruitment step is what lets Novel-X reassemble insertion anchors locally without whole-genome de novo assembly.

## When to use

Use when working with linked-read (10x Chromium, TELL-Seq, stLFR) BAM data where orphan contigs from unaligned reads must be linked to molecule barcodes before targeted reassembly of non-reference insertions.

## When NOT to use

- Input data lacks synthetic long-read barcode information (plain short-read or long-read data without linked-read barcodes) — barcode recruitment is impossible
- The goal is whole-genome de novo assembly; Novel-X's barcode-guided local reassembly is explicitly designed to avoid that
- Detecting insertions small enough that few distinct barcodes associate with them (the authors note small insertions usually have insufficient barcode coverage for anchor assembly)

## Inputs

- LongRanger-aligned 10x/TELL-Seq/stLFR BAM file with per-read barcode (BX) tags
- Orphan contigs (FASTA) produced by Velvet assembly of the unaligned read set U
- Unaligned read set U (paired-end reads with unaligned/low-MQ/soft-clipped ends and average base quality > 20)

## Outputs

- Per-contig barcode sets B(c) (list of barcodes with at least one read aligned to each insertion candidate contig)
- Set of retained orphan contigs recruiting >= t = 5 barcodes (low-barcode contigs discarded)

## How to apply

First, build the unaligned read set U by selecting paired-end reads from the LongRanger-aligned BAM with at least one unaligned end, mapping quality below 10, or >20% soft-clipped bases, keeping only reads with average base quality > 20, and assemble U into orphan contigs with Velvet (k-mer length 63, compiled with MAXKMERLENGTH=63 for 10x data). Then, for each orphan contig, align the reads in U back to the contig and filter out alignments with low quality scores or a large fraction (>= 20%) of soft- or hard-clipped sequence. Collect the barcodes B(c) of all reads whose alignments pass the filter, and discard any contig that recruits fewer than t barcodes (default t = 5). The resulting per-contig barcode lists B(c) feed the next Novel-X stage, in which both aligned and unaligned original reads carrying barcodes in B(c) are pulled from the BAM and reassembled per barcode set with SPAdes (k=77, cov-cutoff=3) to obtain anchors for placement on the reference.

## Related tools

- **Novel-X** (Snakemake pipeline implementing the barcode list extraction step: aligns reads in U to each orphan contig, filters alignments, collects barcodes B(c), and enforces the t=5 barcode threshold) — https://github.com/1dayac/Novel-X
- **Velvet** (Assembles the unaligned read set U into orphan contigs (k-mer length 63, MAXKMERLENGTH=63 for 10x data) that barcode extraction operates on) — https://github.com/dzerbino/velvet
- **SPAdes** (Downstream consumer of the extracted barcode lists: reassembles reads with barcodes in B(c) per barcode set (k=77, cov-cutoff=3) to obtain anchors)

## Examples

```
git clone --recursive git@github.com:1dayac/Novel-X.git && snakemake  # barcode extraction runs as step (ii) of the Novel-X Snakemake pipeline on the LongRanger-aligned input BAM
```

## Evaluation signals

- Every retained orphan contig is associated with at least t = 5 distinct barcodes; contigs recruiting fewer barcodes are absent from the output
- No read contributing to B(c) has an alignment with >= 20% soft- or hard-clipped sequence or a low-quality score
- All reads contributing barcodes belong to the unaligned set U (average base quality > 20, at least one unaligned end, MAPQ < 10, or > 20% soft-clipping)
- For the demo run, the downstream VCF is parseable and its records pass the > 300 bp insertion and anchor-sum thresholds, confirming barcode recruitment fed a valid reassembly

## Limitations

- Novel-X needs a substantial number of distinct barcodes per insertion; small insertions usually have few associated barcodes and insufficient coverage for anchor assembly
- A known problem exists on the filter_target_contig stage, per the article's methods note
- The t=5 barcode threshold and the >= 20% clipping filter trade sensitivity for precision — contigs near the threshold may be spuriously retained or discarded

## Evidence

- [methods] Barcode list extraction stage definition: [evidence span withheld — anchor status: not_found]
- [methods] Alignment filtering for barcode collection: [evidence span withheld — anchor status: not_found]
- [methods] Barcode count threshold: [evidence span withheld — anchor status: not_found]
- [discussion] Small-insertion barcode coverage limitation: "Novel-X needs a substantial number of distinct barcodes to be associated with it. For small insertions, the number of associated barcodes is usually small and the coverage is not sufficient"
- [readme] Barcode-based detection rationale from the repository: "Novel-X detects and genotypes novel sequence insertions in 10X sequencing dataset using non-trivial read alignment signatures and barcode information."
