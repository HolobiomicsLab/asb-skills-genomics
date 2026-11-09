---
name: long-read-single-cell-variant-calling
description: Use when single-cell cDNA library FASTQs (10x Genomics whole-transcriptome
  or targeted amplicon long reads) must be genotyped for somatic SNVs per cell barcode,
  e.g., tracking AML subclones or comparing genotyping rates to Illumina-based GoT.
when_to_use_negative:
- Targets are loci far from the 5′ end of lowly expressed genes or with transcripts
  longer than 4 kB — the genotyping ceiling is determined by low, incomplete transcript
  representation and cannot be overcome by amplicon optimization or deeper sequencing
- The goal is resolving structural variants that short reads miss but whose cDNA is
  not representable in 10x 3′/5′ libraries — co-sequencing of DNA and RNA would be
  needed instead
- Input is already a per-cell genotype matrix or feature table rather than raw long-read
  sequencing data
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: nanoranger
  role: 'Core pipeline: aligns long-read amplicon reads (5p10XGEX mode), calls SNVs,
    and collapses variant calls to a per-cell genotyping matrix keyed by 10x cell
    barcodes'
  repo: https://github.com/mehdiborji/nanoranger
- name: nanoranger.R
  role: R downstream package that reads SNVs, indels, fusions/CAR sequences, and isoforms
    from nanoranger amplicon output; collapses UMIs with starcode (Levenshtein distance
    3)
  repo: https://github.com/liviuspenter/nanoranger.R
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform used to sequence 10x Genomics cDNA libraries
    or targeted amplicons
- name: 10x Genomics
  role: Source of 5′-anchored whole-transcriptome cDNA libraries providing cell barcodes
    and UMIs for single-cell genotyping
- name: minimap2
  role: Splice-aware long-read aligner (e.g., -ax splice presets for ONT/PacBio cDNA
    reads) used for read alignment in long-read single-cell workflows
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:2fa729d3312b571d0adf2904f74bcd523d545a07770e6b117f4f4ba95d875add
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - 10x Genomics
  - Oxford Nanopore Technologies (ONT)
  - minimap2
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
n_claims: 10
---

# long-read-single-cell-variant-calling

## Summary

Call single-nucleotide variants per cell from long-read (ONT or PacBio) sequencing of 10x Genomics 5′-anchored whole-transcriptome cDNA libraries or targeted amplicons using the nanoranger pipeline, enabling natural-barcode genotyping (e.g., AML mutations DNMT3AR882H, SF3B1K700E, RUNX1I177S) with limited primer sets.

## When to use

Use when single-cell cDNA library FASTQs (10x Genomics whole-transcriptome or targeted amplicon long reads) must be genotyped for somatic SNVs per cell barcode, e.g., tracking AML subclones or comparing genotyping rates to Illumina-based GoT.

## When NOT to use

- Targets are loci far from the 5′ end of lowly expressed genes or with transcripts longer than 4 kB — the genotyping ceiling is determined by low, incomplete transcript representation and cannot be overcome by amplicon optimization or deeper sequencing
- The goal is resolving structural variants that short reads miss but whose cDNA is not representable in 10x 3′/5′ libraries — co-sequencing of DNA and RNA would be needed instead
- Input is already a per-cell genotype matrix or feature table rather than raw long-read sequencing data

## Inputs

- Long-read FASTQ (e.g., 1022_DNMT3A_RUNX1_SF3B1.fastq.gz) from ONT or Pacific Biosciences sequencing of 10x Genomics whole-transcriptome cDNA libraries or targeted amplicons
- Reference genome for alignment
- Target variant list (e.g., DNMT3AR882H, RUNX1I177S, SF3B1K700E)

## Outputs

- Per-cell genotyping matrix keyed by 10x cell barcodes
- genotyping_summary.csv with per-cell genotype calls and per-mutation rate columns
- Per-mutation genotyping rates (e.g., 4.8–19.2% for DNMT3AR882H/SF3B1K700E)

## How to apply

Load a long-read FASTQ (e.g., 1022_DNMT3A_RUNX1_SF3B1.fastq.gz derived from a 10x Genomics whole-transcriptome cDNA library) into nanoranger and run in 5p10XGEX mode to align amplicon reads targeting genes such as DNMT3A, RUNX1, and SF3B1 to the reference genome. Call single-nucleotide variants (e.g., DNMT3AR882H, RUNX1I177S, SF3B1K700E) from the aligned reads using nanoranger's variant-calling module, then collapse calls into a per-cell genotyping matrix keyed by 10x cell barcodes. For downstream processing, use the nanoranger.R package, which collapses UMIs (which have no whitelist, unlike cell barcodes) with starcode within a Levenshtein distance of 3 to correct sequencing error. Compute per-mutation genotyping rates; expected performance is comparable to Illumina-based GoT (e.g., 4.8–19.2% for nanoranger vs. 5.5–19.1% for GoT on DNMT3AR882H/SF3B1K700E, ~39.2% for RUNX1I177S). Optimal target loci are close to the 5′ end of highly expressed genes with transcript lengths shorter than 4 kB.

## Related tools

- **nanoranger** (Core pipeline: aligns long-read amplicon reads (5p10XGEX mode), calls SNVs, and collapses variant calls to a per-cell genotyping matrix keyed by 10x cell barcodes) — https://github.com/mehdiborji/nanoranger
- **nanoranger.R** (R downstream package that reads SNVs, indels, fusions/CAR sequences, and isoforms from nanoranger amplicon output; collapses UMIs with starcode (Levenshtein distance 3)) — https://github.com/liviuspenter/nanoranger.R
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform used to sequence 10x Genomics cDNA libraries or targeted amplicons)
- **10x Genomics** (Source of 5′-anchored whole-transcriptome cDNA libraries providing cell barcodes and UMIs for single-cell genotyping)
- **minimap2** (Splice-aware long-read aligner (e.g., -ax splice presets for ONT/PacBio cDNA reads) used for read alignment in long-read single-cell workflows) — https://github.com/lh3/minimap2

## Examples

```
nanoranger --mode 5p10XGEX --input 1022_DNMT3A_RUNX1_SF3B1.fastq.gz --reference GRCh38.fa --variants DNMT3AR882H,RUNX1I177S,SF3B1K700E
```

## Evaluation signals

- Computed per-mutation genotyping rates for DNMT3AR882H and SF3B1K700E fall within the reported 4.8–19.2% range and are comparable to Illumina-based GoT rates (5.5–19.1%)
- RUNX1I177S genotyping rate is approximately 39.2%, matching the reported value
- genotyping_summary.csv exists and contains per-cell genotype calls keyed by 10x cell barcodes plus per-mutation rate columns
- Downstream nanoranger.R processing successfully collapses UMIs into starcode groups within a Levenshtein distance of 3, indicating sequencing-error correction was applied
- Knee plot from nanoranger.R identifies a sensible cell-calling cutoff, confirming valid cell barcode recovery

## Limitations

- The ceiling for any genotyping approach is determined by the low, incomplete representation of many transcripts, including absence of longer variants, and cannot be merely overcome through amplicon optimization or deeper sequencing
- Optimal targets are restricted to loci close to the 5′ end of highly expressed genes with transcript lengths shorter than 4 kB
- Genotyping rates are modest (e.g., 4.8–19.2% for DNMT3AR882H/SF3B1K700E), bounded by transcript capture rather than pipeline accuracy

## Evidence

- [results] Nanoranger and GoT sequenced with Illumina achieved similar genotyping rates for DNMT3AR882H and SF3B1K700E: 4.8–19.2% for nanoranger versus 5.5–19.1% for GoT.: [evidence span withheld — anchor status: not_found]
- [abstract] we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [readme] By using starcode, UMIs that arise due to sequencing error are collapsed in groups within a Levenshtein distance of 3.: [evidence span withheld — anchor status: not_found]
- [abstract] we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB: "we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB"
- [methods] The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification"
