---
name: cohort-vcf-callset-merging-and-statistics
description: Use when per-sample SV/insertion VCFs (e.g. Novel-X output for many samples)
  must be combined into one cohort callset and summarized with count, total length,
  and max-size metrics.
when_to_use_negative:
- Input is a single-sample dataset with no cohort merging or aggregate statistics
  required
- Calls are small (<300 bp) variants or SNPs/indels already handled by standard short-read
  callers rather than NRS insertions
- Samples are not synthetic long-read (10x Genomics/TELL-Seq/stLFR) datasets with
  usable barcode information
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Novel-X
  role: Produces the per-sample NRS insertion VCFs that are merged into the cohort
    callset
  repo: github.com/1dayac/Novel-X
- name: Minimap2
  role: Aligns reassembled insertion contigs to GRCh38; feeds the QUAST-like best-alignment-subset
    filtering prior to callset output
- name: Polaris PGx cohort
  role: Source of the 68 HiSeq 4000 whole-genome samples whose VCFs form the cohort
    callset
  repo: https://github.com/Illumina/Polaris/wiki/HiSeqX-PGx-Cohort
- name: novel_insertions_supplementary
  role: Hosts the benchmark VCF files (Figures 4-5, Tables S1-S3) generated from merged
    callsets
  repo: https://github.com/1dayac/novel_insertions_supplementary
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:ab13b518738de104609f62d6b4cfd0f93bf233220d3c8c47b66ac32a467651d0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LongRanger
  - Minimap2
  - Novel-X
  - Polaris PGx cohort
  - SPAdes
  - Velvet
  - novel_insertions_supplementary
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cohort-vcf-callset-merging-and-statistics

## Summary

Merges per-sample structural-variant VCFs (here, Novel-X non-reference sequence insertion calls from the 68-sample Polaris HiSeq 4000 PGx cohort) into a single cohort-level callset and computes aggregate statistics such as insertion count, total inserted length, and longest insertion.

## When to use

Use when per-sample SV/insertion VCFs (e.g. Novel-X output for many samples) must be combined into one cohort callset and summarized with count, total length, and max-size metrics.

## When NOT to use

- Input is a single-sample dataset with no cohort merging or aggregate statistics required
- Calls are small (<300 bp) variants or SNPs/indels already handled by standard short-read callers rather than NRS insertions
- Samples are not synthetic long-read (10x Genomics/TELL-Seq/stLFR) datasets with usable barcode information

## Inputs

- LongRanger-aligned 10x Genomics BAM files (one per cohort sample)
- GRCh38 reference genome
- Per-sample Novel-X VCF files of NRS insertions >300 bp

## Outputs

- Merged cohort-level VCF callset of non-reference sequence insertions
- Aggregate statistics: insertion count (>300 bp), total inserted length (bp/Mb), longest insertion (bp)
- Cohort-level summary supporting population-specificity / functional-impact interpretation

## How to apply

Run the SV caller (Novel-X, a Snakemake pipeline using Velvet k=63 orphan-contig assembly, barcode-list extraction with t=5 minimum barcodes, SPAdes k=77 cov-cutoff=3 reassembly, and Minimap2 alignment to GRCh38) on each sample to produce per-sample VCFs restricted to insertions longer than 300 bp with anchor-length sums exceeding 300 bp. Merge the per-sample VCFs (68 for the Polaris HiSeq 4000 PGx cohort) into a cohort-level callset, collapsing records that represent the same insertion event across samples. Compute aggregate statistics over the merged callset: number of insertions >300 bp, their total sequence length, and the longest insertion. Use the QUAST-like best-alignment-subset filtering (choosing alignment subsets that maximize continuously covered base pairs) on Minimap2 alignments before finalizing calls to prevent false calls from redundant or overlapping alignments.

## Related tools

- **Novel-X** (Produces the per-sample NRS insertion VCFs that are merged into the cohort callset) — github.com/1dayac/Novel-X
- **Minimap2** (Aligns reassembled insertion contigs to GRCh38; feeds the QUAST-like best-alignment-subset filtering prior to callset output)
- **Polaris PGx cohort** (Source of the 68 HiSeq 4000 whole-genome samples whose VCFs form the cohort callset) — https://github.com/Illumina/Polaris/wiki/HiSeqX-PGx-Cohort
- **novel_insertions_supplementary** (Hosts the benchmark VCF files (Figures 4-5, Tables S1-S3) generated from merged callsets) — https://github.com/1dayac/novel_insertions_supplementary

## Evaluation signals

- Merged callset contains approximately 16,691 NRS insertions of size >300 bp for the 68-sample Polaris HiSeq 4000 PGx cohort
- Total inserted length of the merged callset is approximately 18.2 Mb
- Longest insertion in the merged callset is 85,908 bp
- Every retained record has insertion length >300 bp and sum of anchor lengths >300 bp (Novel-X output criterion)
- Per-sample VCFs are valid VCF format and reconcile 1:1 with samples contributing to the merged callset

## Limitations

- Novel-X requires a substantial number of distinct barcodes; small insertions usually lack sufficient barcode coverage, so the merged callset under-represents small events
- Known pipeline issue on the filter_target_contig stage (documented in the repository)
- SPAdes reassembly uses cov-cutoff=3 and k=77, which may drop low-coverage insertions; no changelog exists in the source repository

## Evidence

- [abstract] Novel-X discovered 16 691 NRS insertions of size > 300 bp (total length 18.2 Mb).: "Novel-X discovered 16 691 NRS insertions of size > 300 bp (total length 18.2 Mb)."
- [methods] We only output insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp to prevent false calls: [evidence span withheld — anchor status: not_found]
- [methods] a filtering procedure that resembles the QUAST (37) procedure, i.e. choosing best alignment subsets from a given alignment set that maximizes the number of continuously covered base pairs: "a filtering procedure that resembles the QUAST (37) procedure, i.e. choosing best alignment subsets from a given alignment set that maximizes the number of continuously covered base pairs"
- [supplementary] The supplementary data including VCFs that were used for benchmarks are available at https://github.com/1dayac/novel_insertions_supplementary.: "The supplementary data including VCFs that were used for benchmarks are available at https://github.com/1dayac/novel_insertions_supplementary."
- [abstract] We applied Novel-X to a diverse set of 68 samples from the Polaris HiSeq 4000 PGx cohort.: "We applied Novel-X to a diverse set of 68 samples from the Polaris HiSeq 4000 PGx cohort."
