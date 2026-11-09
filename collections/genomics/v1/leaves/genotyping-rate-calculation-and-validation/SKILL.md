---
name: genotyping-rate-calculation-and-validation
description: Use when per-cell genotyping rates for targeted somatic SNVs (e.g., AML
  mutations in DNMT3A, RUNX1, SF3B1) must be computed from nanoranger output and compared
  to published GoT/nanoranger benchmarks.
when_to_use_negative:
- Target loci are not near the 5' end of highly expressed genes or transcripts exceed
  ~4 kB, since nanoranger genotyping sensitivity depends on transcript representation
- Input is bulk (non-single-cell) sequencing data without 10x cell barcodes/UMIs,
  so no per-cell genotyping matrix can be constructed
- The mutation of interest is rare/private and not covered by the limited primer panel,
  as the genotyping ceiling is set by low, incomplete transcript representation
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: nanoranger
  role: Aligns long-read amplicon reads (5p10XGEX mode) and calls single-nucleotide
    variants from 10x Genomics cDNA libraries
  repo: https://github.com/mehdiborji/nanoranger
- name: liviuspenter/nanoranger.R
  role: R package for downstream per-cell genotype matrix construction, UMI collapsing
    via starcode (Levenshtein distance 3), knee plots, and mutation extraction
  repo: https://github.com/liviuspenter/nanoranger.R
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform used to sequence amplified target genes (DNMT3A,
    RUNX1, SF3B1)
- name: 10x Genomics
  role: Source of 5'-anchored whole-transcriptome cDNA libraries providing cell barcodes
    and UMIs for per-cell genotyping
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:3629d8e70d8ddec5d4235c349ae4470b3b490452d0362578a80b5b30f2460d8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - 10x Genomics
  - Oxford Nanopore Technologies (ONT)
  - liviuspenter/nanoranger.R
  - nanoranger
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# genotyping-rate-calculation-and-validation

## Summary

Collapse nanoranger long-read variant calls to a per-cell genotyping matrix keyed by 10x Genomics cell barcodes and compute per-mutation genotyping rates, validating them against reported ranges (e.g., 4.8–19.2% for DNMT3AR882H/SF3B1K700E in AML1022).

## When to use

Use when per-cell genotyping rates for targeted somatic SNVs (e.g., AML mutations in DNMT3A, RUNX1, SF3B1) must be computed from nanoranger output and compared to published GoT/nanoranger benchmarks.

## When NOT to use

- Target loci are not near the 5' end of highly expressed genes or transcripts exceed ~4 kB, since nanoranger genotyping sensitivity depends on transcript representation
- Input is bulk (non-single-cell) sequencing data without 10x cell barcodes/UMIs, so no per-cell genotyping matrix can be constructed
- The mutation of interest is rare/private and not covered by the limited primer panel, as the genotyping ceiling is set by low, incomplete transcript representation

## Inputs

- Sample FASTQ from 10x Genomics whole-transcriptome cDNA library or targeted amplicon long-read sequencing (e.g., 1022_DNMT3A_RUNX1_SF3B1.fastq.gz, ONT or PacBio)
- Reference genome for alignment
- Target variant list (e.g., DNMT3AR882H, RUNX1I177S, SF3B1K700E)

## Outputs

- Per-cell genotyping matrix keyed by 10x cell barcodes
- genotyping_summary.csv with per-cell genotype calls and per-mutation genotyping rate columns
- Reported per-mutation genotyping rates (e.g., 4.8–19.2% for DNMT3AR882H/SF3B1K700E; ~39.2% for RUNX1I177S)

## How to apply

Load the sample FASTQ (e.g., 1022_DNMT3A_RUNX1_SF3B1.fastq.gz from a 10x Genomics whole-transcriptome cDNA library) and run nanoranger in 5p10XGEX mode to align long-read amplicon reads to the reference genome. Call single-nucleotide variants (DNMT3AR882H, RUNX1I177S, SF3B1K700E) with the nanoranger variant-calling module, then use the nanoranger.R downstream package to collapse variant calls to a per-cell genotyping matrix keyed by 10x cell barcodes. UMIs arising from sequencing error are collapsed with starcode in groups within a Levenshtein distance of 3, since 10x UMIs (unlike cell barcodes) lack a whitelist for error correction. Compute the fraction of cells carrying each mutation, and optionally run a knee_plot() to determine the optimal cell-calling cutoff. Validate by checking that computed rates fall within reported ranges — 4.8–19.2% for DNMT3AR882H and SF3B1K700E (nanoranger vs. 5.5–19.1% for Illumina-based GoT) and approximately 39.2% for RUNX1I177S — and that a genotyping_summary.csv with per-cell genotype calls and per-mutation rate columns is produced.

## Related tools

- **nanoranger** (Aligns long-read amplicon reads (5p10XGEX mode) and calls single-nucleotide variants from 10x Genomics cDNA libraries) — https://github.com/mehdiborji/nanoranger
- **liviuspenter/nanoranger.R** (R package for downstream per-cell genotype matrix construction, UMI collapsing via starcode (Levenshtein distance 3), knee plots, and mutation extraction) — https://github.com/liviuspenter/nanoranger.R
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform used to sequence amplified target genes (DNMT3A, RUNX1, SF3B1))
- **10x Genomics** (Source of 5'-anchored whole-transcriptome cDNA libraries providing cell barcodes and UMIs for per-cell genotyping)

## Examples

```
Rscript -e 'nanorangerR::knee_plot(cb_counts); g <- extract_mutation(vcf, "DNMT3A_R882H"); write.csv(genotyping_summary, "genotyping_summary.csv")'
```

## Evaluation signals

- Computed genotyping rates for DNMT3AR882H and SF3B1K700E fall within the reported range 4.8–19.2% (nanoranger) and are comparable to GoT's 5.5–19.1%
- Computed genotyping rate for RUNX1I177S is approximately 39.2%
- genotyping_summary.csv exists and contains per-cell genotype calls plus per-mutation rate columns
- UMI collapsing with starcode applied Levenshtein distance 3 grouping, and a knee_plot() was used to set the cell-calling cutoff
- Per-cell genotyping matrix is keyed by valid 10x cell barcodes

## Limitations

- The ceiling for any genotyping approach is determined by the low, incomplete representation of many transcripts (including absence of longer variants) and cannot be overcome through amplicon optimization or deeper sequencing
- Optimal nanoranger targets are loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB; other loci genotype poorly
- UMIs in 10x Genomics libraries have no whitelist, so sequencing-error-derived UMIs must be collapsed heuristically (starcode, Levenshtein distance 3)

## Evidence

- [results] Nanoranger and GoT sequenced with Illumina achieved similar genotyping rates for DNMT3AR882H and SF3B1K700E: 4.8–19.2% for nanoranger versus 5.5–19.1% for GoT.: [evidence span withheld — anchor status: not_found]
- [readme] By using starcode, UMIs that arise due to sequencing error are collapsed in groups within a Levenshtein distance of 3.: [evidence span withheld — anchor status: not_found]
- [methods] The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification"
- [discussion] the ceiling for any genotyping approach is currently determined by the low, incomplete representation of many transcripts including absence of longer variants: "the ceiling for any genotyping approach is currently determined by the low, incomplete representation of many transcripts including absence of longer variants"
- [abstract] we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB: "we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB"
