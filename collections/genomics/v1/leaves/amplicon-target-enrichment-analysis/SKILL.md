---
name: amplicon-target-enrichment-analysis
description: Use when single-cell cDNA libraries (e.g., 10x Genomics 5' GEX FASTQs/amplicons)
  must be enriched for target genes and genotyped per-cell for somatic SNVs, fusions,
  or isoforms via long-read sequencing.
when_to_use_negative:
- When the question requires full-transcriptome coverage or detection of private,
  non-targeted somatic mutations — increasing genotyped features per sample to cover
  less frequently mutated genes remains future work.
- 'When genotyping must exceed the transcript-representation ceiling: the ceiling
  is set by low, incomplete representation of many transcripts (including absent longer
  variants) and cannot be overcome by amplicon optimization or deeper sequencing.'
- When targets are far from the 5' end, lowly expressed, or have transcripts longer
  than 4 kB — these loci are suboptimal for nanoranger.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: nanoranger
  role: 'Processing tool for long-read single-cell transcriptomics: aligns targeted
    amplicon reads (5p10XGEX mode) and calls per-cell variants from 10x barcoded cDNA
    libraries'
  repo: https://github.com/mehdiborji/nanoranger
- name: nanoranger.R
  role: R package for downstream processing of nanoranger output; extracts SNVs, indels,
    fusions/CARs, and isoforms, collapsing UMIs with starcode (Levenshtein distance
    3)
  repo: https://github.com/liviuspenter/nanoranger.R
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform used to sequence targeted amplicons from 10x
    Genomics cDNA libraries
- name: 10x Genomics
  role: Source of 5'-anchored whole-transcriptome cDNA libraries from which target
    genes are amplified
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:664474c5f19eb543d8545083d46a203b536a5b33748d1034e0361b414dbb24c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - 10x Genomics
  - Oxford Nanopore Technologies (ONT)
  - nanoranger
  - nanoranger.R
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# amplicon-target-enrichment-analysis

## Summary

A skill for targeted amplification and long-read sequencing of genes of interest from 5'-anchored 10x Genomics single-cell cDNA libraries, enabling per-cell genotyping of somatic mutations (e.g., DNMT3AR882H, RUNX1I177S, SF3B1K700E) using the nanoranger workflow.

## When to use

Use when single-cell cDNA libraries (e.g., 10x Genomics 5' GEX FASTQs/amplicons) must be enriched for target genes and genotyped per-cell for somatic SNVs, fusions, or isoforms via long-read sequencing.

## When NOT to use

- When the question requires full-transcriptome coverage or detection of private, non-targeted somatic mutations — increasing genotyped features per sample to cover less frequently mutated genes remains future work.
- When genotyping must exceed the transcript-representation ceiling: the ceiling is set by low, incomplete representation of many transcripts (including absent longer variants) and cannot be overcome by amplicon optimization or deeper sequencing.
- When targets are far from the 5' end, lowly expressed, or have transcripts longer than 4 kB — these loci are suboptimal for nanoranger.

## Inputs

- 10x Genomics whole-transcriptome cDNA library amplicon FASTQ (e.g., 1022_DNMT3A_RUNX1_SF3B1.fastq.gz)
- Reference genome for long-read alignment
- List of targeted somatic SNVs (e.g., DNMT3AR882H, RUNX1I177S, SF3B1K700E)
- 10x cell barcodes and UMIs embedded in read identifiers

## Outputs

- Aligned long-read BAM/SAM of amplicon reads
- Per-cell genotyping matrix keyed by 10x cell barcodes
- genotyping_summary.csv with per-cell genotype calls and per-mutation genotyping rate columns

## How to apply

Start from an intermediate 10x Genomics whole-transcriptome cDNA library or targeted amplicons sequenced on ONT (or PacBio) devices — e.g., the AML1022 relapse sample FASTQ 1022_DNMT3A_RUNX1_SF3B1.fastq.gz targeting DNMT3A, RUNX1, and SF3B1. Run nanoranger in 5p10XGEX mode to align long-read amplicon reads to the reference genome, then call single-nucleotide variants from the aligned reads using the nanoranger variant-calling module. Collapse variant calls to a per-cell genotyping matrix keyed by 10x cell barcodes (using nanoranger.R, which collapses UMIs with starcode at Levenshtein distance ≤3 to correct sequencing error, since 10x UMIs lack a whitelist). Compute per-mutation genotyping rates; choose target loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB, since ONT flow cells yield only 5–50 million reads so primer panels must be limited. Rationale: limited primer sets amplify target genes from 5'-anchored cDNA without spike-in of gene-specific primers during library preparation, enabling flexible detection of natural barcodes.

## Related tools

- **nanoranger** (Processing tool for long-read single-cell transcriptomics: aligns targeted amplicon reads (5p10XGEX mode) and calls per-cell variants from 10x barcoded cDNA libraries) — https://github.com/mehdiborji/nanoranger
- **nanoranger.R** (R package for downstream processing of nanoranger output; extracts SNVs, indels, fusions/CARs, and isoforms, collapsing UMIs with starcode (Levenshtein distance 3)) — https://github.com/liviuspenter/nanoranger.R
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform used to sequence targeted amplicons from 10x Genomics cDNA libraries)
- **10x Genomics** (Source of 5'-anchored whole-transcriptome cDNA libraries from which target genes are amplified)

## Examples

```
nanoranger align 1022_DNMT3A_RUNX1_SF3B1.fastq.gz --mode 5p10XGEX --genome GRCh38.fa
```

## Evaluation signals

- Computed genotyping rates for DNMT3AR882H and SF3B1K700E fall within the reported range 4.8–19.2% for nanoranger (comparable to 5.5–19.1% for Illumina-based GoT).
- RUNX1I177S genotyping rate is approximately 39.2%.
- genotyping_summary.csv exists and contains per-cell genotype calls plus per-mutation rate columns.
- Per-cell genotyping matrix keys match 10x cell barcodes from the input FASTQ read identifiers.
- UMI collapse (starcode, Levenshtein distance ≤3) removes sequencing-error-derived UMIs before counting genotypes.

## Limitations

- The genotyping ceiling is determined by low, incomplete transcript representation, including absence of longer variants, and cannot be overcome merely through amplicon optimization or deeper sequencing.
- Optimal targets are loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB; other loci perform poorly.
- Current long-read flow cells generate only 5–50 million reads, limiting the number of genotyped features per sample and excluding less frequently mutated or private somatic mutations.
- Co-sequencing of DNA and RNA that may overcome genotyping limitations is still under development and not yet available.

## Evidence

- [methods] Targeted amplification from cDNA libraries: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification"
- [intro] Limited primer sets from 5'-anchored libraries: "we provide a long-read based pipeline for the ONT platform using limited primer sets to amplify target genes from 5′-anchored 10x Genomics scRNA-seq whole-transcriptome cDNA libraries"
- [abstract] Optimal target definition: "we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB"
- [readme] UMI collapse strategy: [evidence span withheld — anchor status: not_found]
- [discussion] Genotyping ceiling limitation: "the ceiling for any genotyping approach is currently determined by the low, incomplete representation of many transcripts including absence of longer variants"
- [results] Per-cell genotyping rates from source card: [evidence span withheld — anchor status: not_found]
