---
name: tenx-cdna-library-processing
description: Use when the input is a FASTQ from a 5'-anchored 10x Genomics scRNA-seq
  cDNA library (or targeted amplicon thereof) sequenced on ONT/PacBio and per-cell
  genotype or TCR calls are needed.
when_to_use_negative:
- Input reads are Illumina short reads of a TCR library — the study's comparator (5767
  TCR-positive cells) was produced by short-read sequencing, not nanoranger long-read
  processing
- Input is a completed feature/call table; this skill operates on raw long-read FASTQ
  from 10x cDNA libraries or targeted amplicons
- Optimal nanoranger targets are not met (loci not close to the 5' end, poorly expressed,
  or transcripts longer than 4 kB), where detection is expected to be weak
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: nanoranger
  role: Primary pipeline that aligns long reads from 10x cDNA libraries and calls
    cell-barcode-associated features such as TCR sequences (5p10X_XCR mode)
  repo: github.com/mehdiborji/nanoranger
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform used to sequence the 10x cDNA/amplicon libraries
    processed by nanoranger
- name: 10x Genomics
  role: Source of 5'-anchored whole-transcriptome scRNA-seq cDNA libraries and cell
    barcodes from which targeted genes are amplified
- name: nanoranger.R
  role: R downstream processing of nanoranger output; collapses sequencing-error UMIs
    with starcode (Levenshtein distance 3) and extracts SNVs, indels, fusions/CAR
    and isoforms
  repo: liviuspenter/nanoranger.R
- name: minimap2
  role: Long-read aligner used for splice-aware/cDNA mapping of ONT/PacBio reads within
    the long-read analysis toolchain
  repo: github.com/lh3/minimap2
- name: mixcr
  role: Alternative TCR/BCR repertoire analysis software for receptor sequence data
  repo: github.com/milaboratory/mixcr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:94a59b325b23327f089796ab22ee8bfde79947658047229af73029f0a5b407a3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Genomics
  - Oxford Nanopore Technologies (ONT)
  - minimap2
  - mixcr
  - nanoranger
  - nanoranger.R
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# tenx-cdna-library-processing

## Summary

Process 10x Genomics whole-transcriptome or targeted-amplification cDNA libraries sequenced as long reads (ONT or PacBio) with the nanoranger pipeline, which aligns long reads and attributes detected features such as TCR sequences to 10x cell barcodes.

## When to use

Use when the input is a FASTQ from a 5'-anchored 10x Genomics scRNA-seq cDNA library (or targeted amplicon thereof) sequenced on ONT/PacBio and per-cell genotype or TCR calls are needed.

## When NOT to use

- Input reads are Illumina short reads of a TCR library — the study's comparator (5767 TCR-positive cells) was produced by short-read sequencing, not nanoranger long-read processing
- Input is a completed feature/call table; this skill operates on raw long-read FASTQ from 10x cDNA libraries or targeted amplicons
- Optimal nanoranger targets are not met (loci not close to the 5' end, poorly expressed, or transcripts longer than 4 kB), where detection is expected to be weak

## Inputs

- FASTQ of a 10x Genomics whole-transcriptome cDNA library sequenced on Oxford Nanopore Technologies (ONT)
- FASTQ of amplicons obtained through targeted amplification of 10x Genomics cDNA (ONT or Pacific Biosciences)
- Sample FASTQ of the melanoma T cell TCR cDNA library shipped with the nanoranger repository

## Outputs

- Per-cell barcode TCR/genotype calls from nanoranger in 5p10X_XCR mode
- Aggregated cell count table (e.g. tcr_cell_counts.csv)
- Downstream feature tables (SNVs, indels, fusion genes/CAR sequences, isoforms) via nanoranger.R

## How to apply

Start from the sample FASTQ produced by sequencing 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification, generated from 5'-anchored 10x scRNA-seq libraries using limited primer sets. Run the nanoranger workflow in the mode matching the protocol (e.g. 5p10X_XCR for TCR), which aligns the long reads (leveraging long-read mappers such as minimap2 in the broader toolchain) and assigns aligned targets to 10x cell barcodes per read. Aggregate per-barcode calls to count cells with a detected feature (e.g. a productive TCR sequence) and write tabular outputs such as tcr_cell_counts.csv. Judge success by concordance with short-read ground truth: in the melanoma TCR benchmark, nanoranger on ONT detected TCRs in 5653 cells versus 5767 by Illumina, with reported ONT–Illumina concordance of r=0.8–0.9. Optionally process nanoranger output downstream with nanoranger.R, which collapses error-derived UMIs with starcode using a Levenshtein distance of 3 before extracting SNVs, indels, fusions/CAR sequences, and isoforms.

## Related tools

- **nanoranger** (Primary pipeline that aligns long reads from 10x cDNA libraries and calls cell-barcode-associated features such as TCR sequences (5p10X_XCR mode)) — github.com/mehdiborji/nanoranger
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform used to sequence the 10x cDNA/amplicon libraries processed by nanoranger)
- **10x Genomics** (Source of 5'-anchored whole-transcriptome scRNA-seq cDNA libraries and cell barcodes from which targeted genes are amplified)
- **nanoranger.R** (R downstream processing of nanoranger output; collapses sequencing-error UMIs with starcode (Levenshtein distance 3) and extracts SNVs, indels, fusions/CAR and isoforms) — liviuspenter/nanoranger.R
- **minimap2** (Long-read aligner used for splice-aware/cDNA mapping of ONT/PacBio reads within the long-read analysis toolchain) — github.com/lh3/minimap2
- **mixcr** (Alternative TCR/BCR repertoire analysis software for receptor sequence data) — github.com/milaboratory/mixcr

## Evaluation signals

- Cell count check: nanoranger in 5p10X_XCR mode on the melanoma TCR FASTQ should yield 5653 ONT cells with a detected TCR, comparable to the 5767 Illumina TCR-positive cells
- Concordance check: ONT–Illumina per-cell TCR agreement should approximate the reported r=0.8 and r=0.9
- Barcode validity: all assigned calls must map to 10x cell barcodes; outputs must be a per-barcode table writable to tcr_cell_counts.csv
- UMI collapsing invariant in nanoranger.R: error-derived UMIs collapse into groups within a Levenshtein distance of 3 via starcode
- Knee-plot sanity check: knee_plot() in nanoranger.R should identify an optimal cell cutoff consistent with the expected number of captured cells

## Limitations

- Optimal nanoranger targets are loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB; targets outside this space detect poorly
- The ceiling for genotyping is determined by the low, incomplete representation of many transcripts and cannot be merely overcome through amplicon optimization or deeper sequencing
- Current long-read flow cells generating 5–50 million reads limit the size of primer panels that can be used to track isoforms or immune cells

## Evidence

- [abstract] nanoranger pipeline on 10x cDNA FASTQ: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [methods] input data definition: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification"
- [intro] 5'-anchored targeted amplification from 10x libraries: "amplify target genes from 5′-anchored 10x Genomics scRNA-seq whole-transcriptome cDNA libraries"
- [readme] nanoranger tool description from README: "nanoranger is a processing tool for long-read single-cell transcriptomics as described in our Nature Communications paper"
- [readme] nanoranger.R UMI error correction strategy: [evidence span withheld — anchor status: not_found]
