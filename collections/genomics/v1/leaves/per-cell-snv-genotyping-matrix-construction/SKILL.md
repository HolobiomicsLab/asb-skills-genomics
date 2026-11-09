---
name: per-cell-snv-genotyping-matrix-construction
description: Use when you have long-read (ONT/PacBio) amplicon reads from 10x Genomics
  5' scRNA-seq cDNA libraries targeting known somatic mutations and need per-cell
  genotype calls and genotyping rates keyed by cell barcode.
when_to_use_negative:
- The target loci are far from the 5' end of lowly expressed genes with transcripts
  longer than 4 kB — nanoranger performs best for loci close to the 5' end of highly
  expressed genes with transcript lengths shorter than 4 kB
- Input is short-read Illumina data requiring whole-gene primer panels — existing
  Illumina-based single-cell genotyping requires large numbers of primers and is suboptimal
  for long fragments
- You need to detect private or rarely mutated somatic mutations beyond the primed
  targets — increasing genotyped features per sample to such mutations remains future
  work
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: nanoranger
  role: Long-read analysis pipeline run in 5p10XGEX mode to align amplicon reads and
    call SNVs from 10x cDNA libraries
  repo: https://github.com/mehdiborji/nanoranger
- name: nanoranger.R
  role: R package for downstream processing of nanoranger output; extracts SNVs and
    collapses UMIs with starcode (Levenshtein distance 3) to build per-cell genotype
    calls
  repo: https://github.com/liviuspenter/nanoranger.R
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform used to sequence 10x Genomics whole-transcriptome
    cDNA library amplicons
- name: 10x Genomics
  role: Source of 5'-anchored scRNA-seq whole-transcriptome cDNA libraries and cell
    barcodes/UMIs used to key the genotyping matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:38d8ce5029d4885b3157e6e5afc78c57bb06accab60b86d9de53edf5cd341521
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
yes_rate: 1.0
n_claims: 13
---

# per-cell SNV genotyping matrix construction

## Summary

Constructs a per-cell single-nucleotide-variant genotyping matrix from long-read single-cell amplicon data by collapsing nanoranger variant calls to 10x Genomics cell barcodes, enabling quantification of per-mutation genotyping rates as demonstrated for AML1022 mutations (DNMT3AR882H, RUNX1I177S, SF3B1K700E).

## When to use

Use when you have long-read (ONT/PacBio) amplicon reads from 10x Genomics 5' scRNA-seq cDNA libraries targeting known somatic mutations and need per-cell genotype calls and genotyping rates keyed by cell barcode.

## When NOT to use

- The target loci are far from the 5' end of lowly expressed genes with transcripts longer than 4 kB — nanoranger performs best for loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB
- Input is short-read Illumina data requiring whole-gene primer panels — existing Illumina-based single-cell genotyping requires large numbers of primers and is suboptimal for long fragments
- You need to detect private or rarely mutated somatic mutations beyond the primed targets — increasing genotyped features per sample to such mutations remains future work

## Inputs

- FASTQ file of long-read amplicons from 10x Genomics whole-transcriptome cDNA libraries (e.g. 1022_DNMT3A_RUNX1_SF3B1.fastq.gz)
- Reference genome for alignment
- Targeted variant list (e.g. DNMT3AR882H, RUNX1I177S, SF3B1K700E)

## Outputs

- Per-cell genotyping matrix keyed by 10x cell barcodes
- genotyping_summary.csv with per-cell genotype calls and per-mutation rate columns
- Per-mutation genotyping rates

## How to apply

Start from a sample FASTQ derived from a 10x Genomics whole-transcriptome cDNA library (e.g. 1022_DNMT3A_RUNX1_SF3B1.fastq.gz) and run nanoranger in 5p10XGEX mode to align long-read amplicon reads targeting the genes of interest (DNMT3A, RUNX1, SF3B1) to the reference genome. Call single-nucleotide variants (e.g. DNMT3AR882H, RUNX1I177S, SF3B1K700E) from the aligned reads using the nanoranger variant-calling module. Collapse the variant calls to a per-cell genotyping matrix keyed by 10x cell barcodes, using the nanoranger.R downstream package where UMIs are collapsed with starcode into groups within a Levenshtein distance of 3 to correct sequencing error (UMIs lack a whitelist, unlike cell barcodes). Compute per-mutation genotyping rates from the matrix; the rationale is that nanoranger uses limited primer sets on 5'-anchored cDNA libraries to flexibly detect natural genetic barcodes without early commitment to targets.

## Related tools

- **nanoranger** (Long-read analysis pipeline run in 5p10XGEX mode to align amplicon reads and call SNVs from 10x cDNA libraries) — https://github.com/mehdiborji/nanoranger
- **nanoranger.R** (R package for downstream processing of nanoranger output; extracts SNVs and collapses UMIs with starcode (Levenshtein distance 3) to build per-cell genotype calls) — https://github.com/liviuspenter/nanoranger.R
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform used to sequence 10x Genomics whole-transcriptome cDNA library amplicons)
- **10x Genomics** (Source of 5'-anchored scRNA-seq whole-transcriptome cDNA libraries and cell barcodes/UMIs used to key the genotyping matrix)

## Examples

```
nanoranger --mode 5p10XGEX --fastq 1022_DNMT3A_RUNX1_SF3B1.fastq.gz --reference GRCh38 --variants DNMT3A_R882H,RUNX1_I177S,SF3B1_K700E
```

## Evaluation signals

- Computed genotyping rates for DNMT3AR882H and SF3B1K700E fall within the reported 4.8–19.2% range (nanoranger) and are comparable to the Illumina GoT protocol's 5.5–19.1%
- RUNX1I177S genotyping rate is approximately 39.2%, matching the reported value
- genotyping_summary.csv exists and contains per-cell genotype calls plus per-mutation rate columns
- Every genotype call in the matrix maps to a valid 10x cell barcode
- UMI collapsing by starcode resolves sequencing-error-derived UMIs within Levenshtein distance 3, and the knee plot identifies a sensible cell-calling cutoff

## Limitations

- The ceiling for any genotyping approach is set by low, incomplete transcript representation — absence of longer variants cannot be overcome through amplicon optimization or deeper sequencing
- Optimal targets are loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB
- UMIs in 10x Genomics libraries have no whitelist, so sequencing-error correction relies on Levenshtein-distance-based collapsing rather than whitelist correction
- Detecting less frequently mutated or private somatic mutations would require increasing the number of genotyped features per sample, which is future work

## Evidence

- [results] nanoranger and GoT sequenced with Illumina achieved similar genotyping rates: [evidence span withheld — anchor status: not_found]
- [readme] starcode UMI collapsing: [evidence span withheld — anchor status: external_doc]
- [abstract] optimal targets: "we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB"
- [discussion] genotyping ceiling: "the ceiling for any genotyping approach is currently determined by the low, incomplete representation of many transcripts including absence of longer variants"
- [methods] input data source: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries"
