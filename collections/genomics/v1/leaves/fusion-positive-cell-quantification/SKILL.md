---
name: fusion-positive-cell-quantification
description: Use when long reads from a 10x Genomics 5' GEX cDNA library of a mixed
  or tumor sample must be assigned to individual cells harboring specific fusion genes,
  and fusion-positive cell counts and fractions are needed.
when_to_use_negative:
- The fusion breakpoint lies distant from the 5' end of the transcript or the transcript
  exceeds 4 kb, since nanoranger sensitivity drops sharply outside its optimal target
  space
- Input is short-read Illumina data rather than ONT/PacBio long reads of 5'-anchored
  10x cDNA, which cannot resolve structural transcriptomic variants such as gene fusions
- The goal is transcriptome-wide unbiased fusion discovery rather than targeted detection
  of pre-specified fusion junctions
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0625
tools:
- name: nanoranger
  role: Long-read single-cell analysis pipeline run in 5p10XGEX fusion mode to align
    reads across fusion junctions and capture cell barcode/UMI-tagged chimeric reads
  repo: https://github.com/mehdiborji/nanoranger
- name: nanoranger.R
  role: R downstream package to read out fusion genes from nanoranger amplicons, collapsing
    sequencing-error UMIs with starcode within a Levenshtein distance of 3
  repo: https://github.com/liviuspenter/nanoranger.R
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform used to sequence the 10x Genomics cDNA libraries
    from which fusion-supporting reads are derived
- name: 10x Genomics
  role: Source chemistry of the 5'-anchored whole-transcriptome cDNA libraries providing
    cell barcodes and UMIs for per-cell fusion attribution
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:81fb0a3a5b37b2d8b07cef95fa7d8797954262ae321db85ec3aae748e9b31bed
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# fusion-positive-cell-quantification

## Summary

Quantifies single cells carrying fusion transcripts (e.g., BCR::ABL1, RUNX1::RUNX1T1) by processing long-read ONT sequencing of 5'-anchored 10x Genomics scRNA-seq cDNA libraries with the nanoranger pipeline in fusion mode, then deduplicating cell-barcode/UMI pairs per fusion with the fusion_gene.py downstream script.

## When to use

Use when long reads from a 10x Genomics 5' GEX cDNA library of a mixed or tumor sample must be assigned to individual cells harboring specific fusion genes, and fusion-positive cell counts and fractions are needed.

## When NOT to use

- The fusion breakpoint lies distant from the 5' end of the transcript or the transcript exceeds 4 kb, since nanoranger sensitivity drops sharply outside its optimal target space
- Input is short-read Illumina data rather than ONT/PacBio long reads of 5'-anchored 10x cDNA, which cannot resolve structural transcriptomic variants such as gene fusions
- The goal is transcriptome-wide unbiased fusion discovery rather than targeted detection of pre-specified fusion junctions

## Inputs

- ONT long-read FASTQ file from a 10x Genomics 5' whole-transcriptome cDNA library (e.g., K562_Kasumi1_BCRABL1_RUNX1_RUNX1T1.fastq.gz)
- Reference transcripts spanning the fusion junctions of interest (BCR-ABL1, RUNX1-RUNX1T1)
- nanoranger pipeline configured in 5p10XGEX fusion mode
- fusion_gene.py downstream processing script

## Outputs

- fusion_calls.tsv table listing detected fusions with cell barcode and UMI fields
- Per-fusion counts and fractions of fusion-positive cells (e.g., 124/2.7% K562 BCR::ABL1+; 88/3.9% Kasumi-1 RUNX1::RUNX1T1+)

## How to apply

Start from the sample FASTQ (e.g., K562_Kasumi1_BCRABL1_RUNX1_RUNX1T1.fastq.gz) produced from ONT sequencing of a 5'-anchored 10x Genomics whole-transcriptome cDNA library. Run nanoranger in 5p10XGEX fusion mode so reads are aligned to reference transcripts spanning the fusion junctions, retaining chimeric reads that cross either breakpoint together with their 10x cell barcode (CB) and UMI sequences. Execute the downstream fusion_gene.py script to collapse erroneous UMIs (UMIs lack a whitelist, unlike cell barcodes) and deduplicate barcode-UMI pairs per cell, tabulating fusion-supporting molecules per fusion gene. Interpret counts cautiously: because nanoranger targets are optimal only for loci near the 5' end of highly expressed genes with transcripts <4 kb, fusions whose breakpoints lie distant from the 5' end yield low detected fractions even in a pure population. Benchmark against the reported result of 124 (2.7%) BCR::ABL1+ K562 cells and 88 (3.9%) RUNX1::RUNX1T1+ Kasumi-1 cells.

## Related tools

- **nanoranger** (Long-read single-cell analysis pipeline run in 5p10XGEX fusion mode to align reads across fusion junctions and capture cell barcode/UMI-tagged chimeric reads) — https://github.com/mehdiborji/nanoranger
- **nanoranger.R** (R downstream package to read out fusion genes from nanoranger amplicons, collapsing sequencing-error UMIs with starcode within a Levenshtein distance of 3) — https://github.com/liviuspenter/nanoranger.R
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform used to sequence the 10x Genomics cDNA libraries from which fusion-supporting reads are derived)
- **10x Genomics** (Source chemistry of the 5'-anchored whole-transcriptome cDNA libraries providing cell barcodes and UMIs for per-cell fusion attribution)

## Examples

```
nanoranger --mode 5p10XGEX --fusion --fastq K562_Kasumi1_BCRABL1_RUNX1_RUNX1T1.fastq.gz; python fusion_gene.py --fusion-calls fusion_calls.tsv
```

## Evaluation signals

- fusion_calls.tsv exists and contains entries for both BCR::ABL1 and RUNX1::RUNX1T1 with populated cell barcode and UMI fields
- Fraction of fusion-positive cells falls in the reported range: ~2.7% for K562 (BCR::ABL1) and ~3.9% for Kasumi-1 (RUNX1::RUNX1T1)
- Absolute counts near the benchmark of 124 BCR::ABL1+ K562 cells and 88 RUNX1::RUNX1T1+ Kasumi-1 cells
- Every retained chimeric read carries a valid 10x cell barcode and a UMI that was deduplicated after error correction
- Detected fusions are restricted to the targeted junction references; no spurious off-target fusions dominate the call table

## Limitations

- Fusion breakpoints distant from the 5' end are captured inefficiently, as only 124 (2.7%) K562 and 88 (3.9%) Kasumi-1 cells were detected in a mixing library designed to be fusion-positive
- Optimal nanoranger targets are limited to loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB
- UMIs in 10x libraries lack a whitelist, so sequencing-error-derived UMIs must be collapsed computationally rather than corrected against a reference

## Evidence

- [intro] Short-read limitation motivating long-read fusion detection: "short-read sequencing is unable to easily resolve structural transcriptomic variants such as gene fusions, transgenes or even isoforms that characterize malignant and immune cell subpopulations."
- [abstract] Nanoranger pipeline definition: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [abstract] Lineage-defining features including fusion genes: "to detect cell lineage-defining features, including single-nucleotide variants, fusion genes, isoforms, sequences of chimeric antigen and TCRs"
- [abstract] Optimal target definition: "we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB"
- [readme] Input data provenance: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification, with Oxford Nanopore Technologies (ONT)"
- [readme] UMI error correction strategy: [evidence span withheld — anchor status: not_found]
