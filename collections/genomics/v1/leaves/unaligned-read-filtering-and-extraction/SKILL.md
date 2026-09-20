---
name: unaligned-read-filtering-and-extraction
description: Use when starting from LongRanger-aligned 10x (or TELL-Seq/stLFR) linked-read
  BAMs and you need the small fraction of reads with unmapped/MAPQ<10/soft-clipped
  ends to assemble novel insertions.
when_to_use_negative:
- Input is a plain short-read dataset without linked-read barcodes — downstream Novel-X
  steps (barcode list extraction, reassembly) need synthetic long-read barcode information
  that short-read methods lack
- Goal is exhaustive variant calling from an existing de novo assembly or long-read
  (PacBio/ONT) data — alignment-signature extraction targets linked-read alignment
  BAMs specifically
- Reads are already assembled into contigs; this skill operates on read-level BAM
  alignments before assembly
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Novel-X
  role: Snakemake pipeline that implements the read-extraction filters (unmapped end,
    MAPQ < 10, > 20% soft-clipped, avg base quality > 20) as its first stage
  repo: github.com/1dayac/Novel-X
- name: LongRanger
  role: Produces the 10x aligned BAM whose alignment signatures (unmapped/low-MAPQ/soft-clipped
    ends) are the selection input
- name: Velvet
  role: Assembles the extracted unaligned read set into orphan insertion contigs (k-mer
    length 63 for 10x data)
  repo: github.com/dzerbino/velvet
- name: SPAdes
  role: Reassembles barcode-recruited reads into left-anchor--insertion--right-anchor
    contigs after extraction (k=77, cov-cutoff=3)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:280e193c3f17d731a4de5c0bc4fe83111e01c924a3e91783b48843fa0e6eb7be
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LongRanger
  - Novel-X
  - SPAdes
  - Velvet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# unaligned-read-filtering-and-extraction

## Summary

Extracts poorly-mapped or unmapped paired-end reads from linked-read (e.g. 10x Genomics LongRanger) alignments to feed NRS insertion assembly, keeping only informative, high-quality reads. It is the first stage of Novel-X's alignment-signature-based discovery of non-reference sequence insertions.

## When to use

Use when starting from LongRanger-aligned 10x (or TELL-Seq/stLFR) linked-read BAMs and you need the small fraction of reads with unmapped/MAPQ<10/soft-clipped ends to assemble novel insertions.

## When NOT to use

- Input is a plain short-read dataset without linked-read barcodes — downstream Novel-X steps (barcode list extraction, reassembly) need synthetic long-read barcode information that short-read methods lack
- Goal is exhaustive variant calling from an existing de novo assembly or long-read (PacBio/ONT) data — alignment-signature extraction targets linked-read alignment BAMs specifically
- Reads are already assembled into contigs; this skill operates on read-level BAM alignments before assembly

## Inputs

- LongRanger-aligned 10x Genomics paired-end BAM (e.g. Polaris HiSeq 4000 PGx cohort BAMs, GRCh38 reference)
- Reference genome FASTA (e.g. GRCh38)
- Sample index/BAR files with linked-read barcode information

## Outputs

- Filtered FASTQ/BAM of extracted unaligned or poorly-aligned paired-end reads (avg base quality > 20)
- Read set for downstream Velvet orphan insertion-contig assembly

## How to apply

Scan the LongRanger-aligned BAM of each sample and select paired-end reads where at least one end is unmapped, the mapping quality is below 10, or more than 20% of the read bases are soft-clipped — these non-trivial alignment signatures mark reads carrying non-reference sequence. Within this set, keep only reads whose average base quality score exceeds 20 to suppress spurious extraction. The extracted read subset is deliberately small (Novel-X processes 'a very small fraction of informative linked-reads' rather than whole-genome de novo assembly) and becomes the direct input for Velvet orphan-contig assembly (k-mer 63 for 10x data) and subsequent barcode list extraction. Rationale: only a tiny fraction of reads deviate from the reference, so filtering at the alignment level isolates insertion-bearing reads cheaply while barcode-aware downstream steps (t=5 barcodes per contig) then discard contamination.

## Related tools

- **Novel-X** (Snakemake pipeline that implements the read-extraction filters (unmapped end, MAPQ < 10, > 20% soft-clipped, avg base quality > 20) as its first stage) — github.com/1dayac/Novel-X
- **LongRanger** (Produces the 10x aligned BAM whose alignment signatures (unmapped/low-MAPQ/soft-clipped ends) are the selection input)
- **Velvet** (Assembles the extracted unaligned read set into orphan insertion contigs (k-mer length 63 for 10x data)) — github.com/dzerbino/velvet
- **SPAdes** (Reassembles barcode-recruited reads into left-anchor--insertion--right-anchor contigs after extraction (k=77, cov-cutoff=3))

## Evaluation signals

- Every retained read satisfies at least one of: unmapped end, mapping quality < 10, or > 20% soft-clipped bases
- All retained reads have average base quality score > 20
- The extracted read set is a small fraction of total reads, not a whole-genome subset
- Downstream orphan contigs assembled from the extract recruit at least t=5 barcodes; contigs below t are correctly discarded
- The cohort workflow (68 Polaris samples) yields a merged callset of ~16,691 insertions > 300 bp totaling 18.2 Mb, indicating the extraction stage fed a productive assembly

## Limitations

- Small insertions usually recruit few distinct barcodes, so their reads may be extracted but fail anchor assembly downstream
- Reads with >= 20% soft/hard-clipped alignments are further excluded at the barcode-list stage, so the coarse > 20% soft-clip pre-filter alone is not sufficient to guarantee contig retention
- The pipeline is sensitive to the reference choice (e.g. GRCh38); reads spanning regions absent from the reference inflate extraction counts

## Evidence

- [methods] We choose paired-end reads with at least one end not aligned to the reference genome, or aligned but with a mapping quality below 10, or that have > 20% of soft-clipped bases: "We choose paired-end reads with at least one end not aligned to the reference genome, or aligned but with a mapping quality below 10, or that have > 20% of soft-clipped bases"
- [methods] we choose only reads with the average base quality score > 20: "we choose only reads with the average base quality score > 20"
- [readme] Novel-X detects and genotypes novel sequence insertions in 10X sequencing dataset using non-trivial read alignment signatures and barcode information.: "Novel-X detects and genotypes novel sequence insertions in 10X sequencing dataset using non-trivial read alignment signatures and barcode information."
- [methods] Each contig that recruits less than t barcodes is discarded ... we set the parameter t to 5 as default: [evidence span withheld — anchor status: not_found]
