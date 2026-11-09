---
name: single-cell-barcode-umi-annotation
description: Use when processing long-read FASTQ from 10x Genomics 5'-anchored whole-transcriptome
  cDNA libraries or targeted amplicons and you need per-cell, per-UMI assignment of
  reads before calling SNVs, fusions, isoforms, or CAR/TCR sequences.
when_to_use_negative:
- Input is bulk (non-single-cell) amplicon or short-read Illumina data without 10x
  cell barcodes and UMIs embedded in reads — there is nothing to demultiplex per cell
- The transcript/locus of interest is far from the 5' end or longer than ~4 kB, where
  nanoranger's 5'-anchored capture is suboptimal
- Standard short-read scRNA-seq expression quantification is sufficient and no lineage-defining
  variant (SNV, fusion, isoform, CAR/TCR) is being genotyped
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: nanoranger
  role: Main pipeline that processes long-read 10x single-cell FASTQ, extracting cell
    barcodes and UMIs and assigning reads to transcript classes in 5p10XGEX/fusion
    mode
  repo: https://github.com/mehdiborji/nanoranger
- name: nanoranger.R
  role: Downstream R package that collapses UMIs with starcode (Levenshtein distance
    3), runs knee_plot() cell calling, and extracts mutations, fusions/CARs, and isoforms
    per cell
  repo: https://github.com/liviuspenter/nanoranger.R
- name: starcode
  role: UMI clustering/correction tool used to collapse sequencing-error UMIs within
    Levenshtein distance 3
  repo: https://github.com/gui11aume/starcode
- name: minimap2
  role: Splice-aware long-read aligner (e.g., -ax splice presets for ONT/PacBio cDNA
    reads) producing alignments onto which CB/UMI information is carried
  repo: https://github.com/lh3/minimap2
- name: STAR
  role: Alternative splice-aware aligner referenced by the nanoranger workflow
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:de434dec169cd2e66b577261ba806b561a90dadb8e8afab536e697a01bce90f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - 10x Genomics
  - STAR
  - minimap2
  - nanoranger
  - nanoranger.R
  - starcode
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell-barcode-umi-annotation

## Summary

Extracts and annotates 10x Genomics cell barcodes (CBs) and unique molecular identifiers (UMIs) from long-read (ONT/PacBio) single-cell sequencing data, then error-corrects UMIs and calls cells so reads can be assigned per-cell to variants such as SNVs, fusion genes, CAR/TCR sequences, and isoforms.

## When to use

Use when processing long-read FASTQ from 10x Genomics 5'-anchored whole-transcriptome cDNA libraries or targeted amplicons and you need per-cell, per-UMI assignment of reads before calling SNVs, fusions, isoforms, or CAR/TCR sequences.

## When NOT to use

- Input is bulk (non-single-cell) amplicon or short-read Illumina data without 10x cell barcodes and UMIs embedded in reads — there is nothing to demultiplex per cell
- The transcript/locus of interest is far from the 5' end or longer than ~4 kB, where nanoranger's 5'-anchored capture is suboptimal
- Standard short-read scRNA-seq expression quantification is sufficient and no lineage-defining variant (SNV, fusion, isoform, CAR/TCR) is being genotyped

## Inputs

- Long-read FASTQ from 10x Genomics 5'-anchored whole-transcriptome cDNA libraries (e.g., K562_Kasumi1_BCRABL1_RUNX1_RUNX1T1.fastq.gz, 97_6_CAR.fastq.gz)
- Targeted amplicon FASTQ generated from intermediate 10x cDNA libraries
- Reference transcript/genome set spanning features of interest (fusion junctions, CAR_CD28 construct, SNV loci)

## Outputs

- Alignments tagged with cell barcode (CB) and UMI (BAM tags or read-ID-embedded sequences)
- Per-cell, per-feature UMI-deduplicated count tables (e.g., fusion_calls.tsv, car_wt_quantification.csv)
- Cell-calling diagnostics (knee plot with optimal cell cutoff)
- Per-cell classification of transcript classes (e.g., fusion-positive vs negative, CAR+ vs wildtype CD28)

## How to apply

Run nanoranger on the long-read FASTQ (5p10XGEX or fusion mode) so cell barcode and UMI sequences are extracted from the reads and carried as tags on the transcript alignments, which are produced by a splice-aware long-read aligner such as minimap2 or STAR. Deduplicate molecule-level counts by collapsing UMIs: since UMIs in 10x Genomics libraries have no whitelist for correction, use starcode to collapse sequencing-error UMIs within a Levenshtein distance of 3, as implemented in the nanoranger.R downstream package. Call cells with a knee-plot cutoff (nanoranger.R's knee_plot()) rather than an arbitrary threshold, then tabulate barcode-UMI pairs per cell for each feature class (e.g., fusion-supporting reads via fusion_gene.py, or CAR vs wildtype CD28 transcript class). Validate output by checking that the per-cell tables (e.g., fusion_calls.tsv, car_wt_quantification.csv) contain barcode and UMI fields and that fusion/genotype-positive cell fractions are in the expected range.

## Related tools

- **nanoranger** (Main pipeline that processes long-read 10x single-cell FASTQ, extracting cell barcodes and UMIs and assigning reads to transcript classes in 5p10XGEX/fusion mode) — https://github.com/mehdiborji/nanoranger
- **nanoranger.R** (Downstream R package that collapses UMIs with starcode (Levenshtein distance 3), runs knee_plot() cell calling, and extracts mutations, fusions/CARs, and isoforms per cell) — https://github.com/liviuspenter/nanoranger.R
- **starcode** (UMI clustering/correction tool used to collapse sequencing-error UMIs within Levenshtein distance 3) — https://github.com/gui11aume/starcode
- **minimap2** (Splice-aware long-read aligner (e.g., -ax splice presets for ONT/PacBio cDNA reads) producing alignments onto which CB/UMI information is carried) — https://github.com/lh3/minimap2
- **STAR** (Alternative splice-aware aligner referenced by the nanoranger workflow) — https://github.com/alexdobin/STAR

## Examples

```
starcode -d 3 umis.fa > collapsed_umis.txt  # collapse sequencing-error UMIs at Levenshtein distance 3 as in nanoranger.R
```

## Evaluation signals

- Output tables (e.g., fusion_calls.tsv) contain entries for the expected features (BCR::ABL1, RUNX1::RUNX1T1) with populated cell barcode and UMI fields
- Fraction of feature-positive cells falls in the reported range, e.g., ~2.7% BCR::ABL1+ K562 cells (124 cells) and ~3.9% RUNX1::RUNX1T1+ Kasumi-1 cells (88 cells) in the mixing-library benchmark
- CAR+ cell fraction exceeds 0.75 when classifying CAR vs wildtype CD28 from targeted amplicon data, consistent with the reported >75% detection
- UMI collapsing is restricted to Levenshtein distance ≤ 3, and knee-plot cell calling yields a defensible cell cutoff rather than arbitrary inclusion
- Every retained chimeric/genotype-supporting read is traceable to a deduplicated barcode-UMI pair per cell (no double counting of sequencing-error UMIs)

## Limitations

- Fusion/genotype detection is limited by 5'-anchored capture: fusion breakpoints distant from the 5' end yielded only 2.7% (K562) and 3.9% (Kasumi-1) fusion-positive cells
- Optimal targets are loci close to the 5' end of highly expressed genes with transcripts shorter than 4 kB; poorly expressed or long loci yield low recovery
- The genotyping ceiling is set by the low, incomplete representation of many transcripts and cannot be overcome merely by amplicon optimization or deeper sequencing

## Evidence

- [readme] UMI error correction via starcode clustering: [evidence span withheld — anchor status: not_found]
- [readme] UMIs lack a whitelist unlike cell barcodes: [evidence span withheld — anchor status: external_doc]
- [readme] knee_plot cell calling: [evidence span withheld — anchor status: external_doc]
- [readme] Input data origin: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification"
- [abstract] Long-read detection of lineage-defining features from intermediate cDNA libraries: "starts from intermediate single-cell cDNA libraries to detect cell lineage-defining features, including single-nucleotide variants, fusion genes, isoforms"
