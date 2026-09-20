---
name: barcode-aware-local-assembly
description: Use when detecting and placing novel sequence insertions (>300 bp) in
  linked-read data (10X Genomics barcoded BAMs), when short-read SV callers fail to
  resolve large insertions.
when_to_use_negative:
- 'Insertions of interest are shorter than ~300 bp: Novel-X needs a substantial number
  of distinct barcodes per insertion, and small insertions usually have too few associated
  barcodes and insufficient coverage for anchor assembly'
- The input is a whole-genome de novo assembly task or a general SV discovery problem
  beyond NRS insertions (e.g. deletions, inversions, duplications)
- Data lacks barcoded synthetic long-read structure (e.g. standard short-read or long-read-only
  data without barcodes) — the method depends on barcode and long-fragment information
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Novel-X
  role: Main Snakemake pipeline that integrates read-alignment signature extraction,
    barcode list extraction, insertion reassembly, and reference placement of NRS
    insertions
  repo: github.com/1dayac/Novel-X
- name: Velvet
  role: First-stage assembler of extracted unaligned/poorly-aligned reads into orphan
    insertion contigs (k-mer length 63, MAXKMERLENGTH=63 for 10x data)
  repo: https://github.com/dzerbino/velvet
- name: SPAdes
  role: Reassembles reads carrying candidate barcodes (k=77, cov-cutoff=3) to build
    anchored insertion contigs
- name: Minimap2
  role: Aligns reassembled contigs to the GRCh38 reference for placement and QUAST-like
    best-alignment-subset filtering
- name: LongRanger
  role: Produces the aligned BAM with barcode information from which informative reads
    and barcodes are extracted
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:d07f7d172929439a2e79805abc52df2ff466e28cae1242eb797769babd6d2924
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LongRanger
  - Minimap2
  - Novel-X
  - SPAdes
  - Velvet
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# barcode-aware-local-assembly

## Summary

Detects and assembles non-reference sequence (NRS) insertions from synthetic long-read (linked-read) data by combining non-trivial read alignment signatures with barcode information, avoiding whole-genome de novo assembly. Novel-X implements this as a Snakemake pipeline for 10X Genomics, TELL-Seq and stLFR datasets.

## When to use

Use when detecting and placing novel sequence insertions (>300 bp) in linked-read data (10X Genomics barcoded BAMs), when short-read SV callers fail to resolve large insertions.

## When NOT to use

- Insertions of interest are shorter than ~300 bp: Novel-X needs a substantial number of distinct barcodes per insertion, and small insertions usually have too few associated barcodes and insufficient coverage for anchor assembly
- The input is a whole-genome de novo assembly task or a general SV discovery problem beyond NRS insertions (e.g. deletions, inversions, duplications)
- Data lacks barcoded synthetic long-read structure (e.g. standard short-read or long-read-only data without barcodes) — the method depends on barcode and long-fragment information

## Inputs

- 10X Genomics (or TELL-Seq/stLFR) linked-read dataset with barcode information in FASTQ/BAM form
- BAM aligned with LongRanger (or an equivalent linked-read aligner) to a reference genome such as GRCh38
- Reference genome FASTA (e.g. GRCh38)

## Outputs

- Per-sample VCF of non-reference sequence (NRS) insertions with genomic placement from assembled anchors
- Merged cohort-level VCF/callset with aggregate statistics (insertion count, total length, longest insertion)
- Assembled left-anchor--insertion--right-anchor contigs from SPAdes reassembly

## How to apply

Run Novel-X as a Snakemake pipeline against a LongRanger-aligned BAM and a reference (e.g. GRCh38). First extract paired-end reads with at least one unmapped end, MAPQ < 10, or >20% soft-clipped bases, keeping only reads with average base quality >20, then assemble these unaligned reads with Velvet (k=63, MAXKMERLENGTH=63) to obtain orphan insertion contigs. Extract barcode lists per contig from filtered alignments (discarding alignments with >=20% soft/hard-clipped bases), discard contigs recruiting fewer than t=5 barcodes, and reassemble all reads carrying those barcodes with SPAdes (k=77, cov-cutoff=3) to build left-anchor--insertion--right-anchor contigs, retaining only contigs whose best Minimap2 match corresponds to an orphan contig. Align reassembled contigs to the reference with Minimap2, apply QUAST-like best-alignment-subset filtering (maximizing continuously covered base pairs), and output only insertions longer than 300 bp with summed anchor lengths exceeding 300 bp, writing per-sample VCFs that can be merged cohort-wide.

## Related tools

- **Novel-X** (Main Snakemake pipeline that integrates read-alignment signature extraction, barcode list extraction, insertion reassembly, and reference placement of NRS insertions) — github.com/1dayac/Novel-X
- **Velvet** (First-stage assembler of extracted unaligned/poorly-aligned reads into orphan insertion contigs (k-mer length 63, MAXKMERLENGTH=63 for 10x data)) — https://github.com/dzerbino/velvet
- **SPAdes** (Reassembles reads carrying candidate barcodes (k=77, cov-cutoff=3) to build anchored insertion contigs)
- **Minimap2** (Aligns reassembled contigs to the GRCh38 reference for placement and QUAST-like best-alignment-subset filtering)
- **LongRanger** (Produces the aligned BAM with barcode information from which informative reads and barcodes are extracted)

## Examples

```
git clone --recursive git@github.com:1dayac/Novel-X.git  # recursive clone required; pipeline runs via Snakemake on LongRanger-aligned 10X BAMs
```

## Evaluation signals

- Cohort-level invariant: applying the pipeline to the 68-sample Polaris HiSeq 4000 PGx cohort should yield approximately 16,691 NRS insertions >300 bp totalling 18.2 Mb, with a longest insertion of 85,908 bp
- Filter invariant: every reported insertion is >300 bp and the sum of its two anchor lengths exceeds 300 bp
- Reassembly consistency check: each retained reassembled contig's best Minimap2 match must correspond to an orphan contig from the Velvet stage
- Barcode-support check: each orphan contig must recruit at least t=5 distinct barcodes before reassembly; contigs below this threshold are discarded
- Callset reproducibility: per-sample VCFs merge cleanly into a cohort callset, and benchmark VCFs should match those in the 1dayac/novel_insertions_supplementary repository (NA12878, NA19240, CHM1, CHM13, HG002, TELL-Seq and stLFR variants)

## Limitations

- Novel-X needs a substantial number of distinct barcodes associated with an insertion; for small insertions the number of associated barcodes is usually small and coverage is not sufficient for anchor assembly
- SuperNova assembler (used in related workflows) offers no user control over parameters, with defaults maximally tuned for whole human genome diploid assembly
- There is a known problem on the filter_target_contig stage at the moment (per the article's methods note)

## Evidence

- [abstract] We propose a novel integrated alignment-based and local assembly-based algorithm, Novel-X, that uses the barcode information encoded in synthetic long reads: "We propose a novel integrated alignment-based and local assembly-based algorithm, Novel-X, that uses the barcode information encoded in synthetic long reads"
- [methods] We choose paired-end reads with at least one end not aligned to the reference genome, or aligned but with a mapping quality below 10, or that have > 20% of soft-clipped bases: "We choose paired-end reads with at least one end not aligned to the reference genome, or aligned but with a mapping quality below 10, or that have > 20% of soft-clipped bases"
- [methods] k=77 (that empirically works better than other standard single k-mer lengths) and cov-cutoff = 3 (to remove low-covered spurious sequences): "k=77 (that empirically works better than other standard single k-mer lengths) and cov-cutoff = 3 (to remove low-covered spurious sequences)"
- [methods] We only output insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp to prevent false calls: [evidence span withheld — anchor status: not_found]
- [abstract] Novel-X discovered 16 691 NRS insertions of size > 300 bp (total length 18.2 Mb).: "Novel-X discovered 16 691 NRS insertions of size > 300 bp (total length 18.2 Mb)."
- [readme] If you clone repository non-recursively Novel-X will not work. To fix this run from Novel-X folder:: "If you clone repository non-recursively Novel-X will not work. To fix this run from Novel-X folder:"
