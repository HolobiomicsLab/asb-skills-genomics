---
name: novel-insertion-variant-calling
description: Use when you have barcode-aware synthetic long-read data (10x Genomics
  Chromium, TELL-Seq, or stLFR) aligned to a reference genome such as GRCh38, and
  the goal is to discover non-reference sequence insertions >300 bp that short-read
  callers miss.
when_to_use_negative:
- 'The goal is to call small insertions: Novel-X needs a substantial number of distinct
  barcodes per insertion, and for small insertions the number of associated barcodes
  is usually small and coverage is insufficient for anchor assembly.'
- The data are plain short reads without barcode/linked-read information, since the
  method fundamentally depends on barcode and long-fragment information encoded in
  synthetic long reads.
- A full whole-genome de novo assembly is already available or required; Novel-X is
  designed to avoid whole-genome de novo assembly and only processes a small fraction
  of informative linked-reads.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Novel-X
  role: Main pipeline for NRS insertion detection and genotyping using synthetic long-read
    barcode information
  repo: github.com/1dayac/Novel-X
- name: Velvet
  role: Local assembler used within the Novel-X pipeline for contig assembly
  repo: github.com/dzerbino/velvet
- name: stlfr2supernova_pipeline
  role: Converts stLFR raw reads into a format compatible with 10X/LongRanger-style
    processing
  repo: github.com/BGI-Qingdao/stlfr2supernova_pipeline
- name: LongRanger
  role: Alignment/processing tool used to produce LongRanger-compatible BAMs from
    10x and TELL-Seq linked-read data
- name: SPAdes
  role: Assembler used with k=77 and cov-cutoff=3 to build insertion candidate contigs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:763c58002469ba194dbf305cc1de777d46e234d7684f17d9eeb70757e069d086
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LongRanger
  - Novel-X
  - SPAdes
  - Velvet
  - stlfr2supernova
  - stlfr2supernova_pipeline
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# novel-insertion-variant-calling

## Summary

Detects and genotypes non-reference sequence (NRS) insertions longer than 300 bp in synthetic long-read (linked-read) data using Novel-X, an integrated alignment-based and local assembly-based algorithm that leverages barcode information. It recovers large novel insertions that short-read methods fail to assemble and place, without whole-genome de novo assembly.

## When to use

Use when you have barcode-aware synthetic long-read data (10x Genomics Chromium, TELL-Seq, or stLFR) aligned to a reference genome such as GRCh38, and the goal is to discover non-reference sequence insertions >300 bp that short-read callers miss.

## When NOT to use

- The goal is to call small insertions: Novel-X needs a substantial number of distinct barcodes per insertion, and for small insertions the number of associated barcodes is usually small and coverage is insufficient for anchor assembly.
- The data are plain short reads without barcode/linked-read information, since the method fundamentally depends on barcode and long-fragment information encoded in synthetic long reads.
- A full whole-genome de novo assembly is already available or required; Novel-X is designed to avoid whole-genome de novo assembly and only processes a small fraction of informative linked-reads.

## Inputs

- BAM/CRAM of synthetic long-read data (10x Genomics Chromium, TELL-Seq, or stLFR) aligned to a reference genome (e.g. GRCh38) and made LongRanger-compatible
- Reference genome FASTA (e.g. GRCh38)
- For stLFR: raw stLFR reads processed through stlfr2supernova_pipeline

## Outputs

- VCF callset of non-reference sequence insertions >300 bp with anchors summing to >300 bp
- Per-sample insertion calls for downstream benchmarking or population analysis

## How to apply

Clone the Novel-X repository recursively (git clone --recursive git@github.com:1dayac/Novel-X.git) and run novel-x.py run with a Snakemake-based pipeline using a reference such as GRCh38 and a LongRanger-compatible BAM (LongRanger for 10x/TELL-Seq; stlfr2supernova for stLFR). The pipeline extracts paired-end reads with at least one end unaligned or MAPQ <10 or >20% soft-clipped, filters to reads with average base quality >20, assembles candidates with SPAdes (k=77, cov-cutoff=3), extracts barcodes per insertion candidate, reassembles reads with those barcodes to obtain anchors, and locates anchors on the reference. Discard orphan contigs recruiting fewer than t=5 barcodes and apply QUAST-like best-alignment-subset filtering on contig alignments. Output only insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp to prevent false calls.

## Related tools

- **Novel-X** (Main pipeline for NRS insertion detection and genotyping using synthetic long-read barcode information) — github.com/1dayac/Novel-X
- **Velvet** (Local assembler used within the Novel-X pipeline for contig assembly) — github.com/dzerbino/velvet
- **stlfr2supernova_pipeline** (Converts stLFR raw reads into a format compatible with 10X/LongRanger-style processing) — github.com/BGI-Qingdao/stlfr2supernova_pipeline
- **LongRanger** (Alignment/processing tool used to produce LongRanger-compatible BAMs from 10x and TELL-Seq linked-read data)
- **SPAdes** (Assembler used with k=77 and cov-cutoff=3 to build insertion candidate contigs)

## Examples

```
python novel-x.py run --reference GRCh38.fa --bam sample_linked_reads.bam --output sample_novelx_calls/
```

## Evaluation signals

- VCF calls should be restricted to insertions >300 bp whose anchor lengths sum to >300 bp.
- Benchmark against a T2T-CHM13 assembly v2.0-derived insertion set: ~78% of Novel-X CHM13 calls should be confirmed by aligning the T2T assembly to GRCh38 with Minimap2 and matching long insertion calls.
- Downsampled simulation (20/40/60/80% read removal) should show sensitivity consistent with the paper's simulated VCF benchmarks in the supplementary repository.
- Orphan contigs recruiting fewer than 5 barcodes must be discarded; surviving calls should have sufficient barcode support.
- Callsets should match those deposited in the novel_insertions_supplementary repository (NA12878, NA19240, CHM1, CHM13, HG002, and TELL-Seq/stLFR variants).

## Limitations

- Novel-X requires a substantial number of distinct barcodes; small insertions often lack sufficient coverage for anchor assembly.
- There is a known issue on the filter_target_contig stage (per the article's methods notes).
- For stLFR, only paired reads with equal-length ends work with the current setup, and running Pamir on additional datasets was limited for the same reason.
- The Supernova assembler used in stlfr2supernova offers no user control over parameters; defaults are tuned for whole human genome diploid assembly.
- A dedicated local assembler designed specifically for SV detection in synthetic long-read data does not yet exist; assembly of other classes of complex SVs remains unexplored.

## Evidence

- [intro] Novel-X detects and genotypes novel sequence insertions in 10X sequencing dataset using non-trivial read alignment signatures and barcode information.: "Novel-X detects and genotypes novel sequence insertions in 10X sequencing dataset using non-trivial read alignment signatures and barcode information."
- [methods] Output only insertions longer than 300 bp with anchor sum exceeding 300 bp: [evidence span withheld — anchor status: not_found]
- [methods] Extraction of unaligned/poorly-aligned paired-end reads: "We choose paired-end reads with at least one end not aligned to the reference genome, or aligned but with a mapping quality below 10, or that have > 20% of soft-clipped bases"
- [methods] SPAdes k and cov-cutoff parameters: "k=77 (that empirically works better than other standard single k-mer lengths) and cov-cutoff = 3 (to remove low-covered spurious sequences)"
- [discussion] Small-insertion limitation: "Novel-X needs a substantial number of distinct barcodes to be associated with it. For small insertions, the number of associated barcodes is usually small"
- [readme] Repository clone instructions: "git clone --recursive git@github.com:1dayac/Novel-X.git"
- [readme] Supplementary VCF availability: [evidence span withheld — anchor status: external_doc]
