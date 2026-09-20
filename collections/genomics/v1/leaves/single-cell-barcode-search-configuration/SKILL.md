---
name: single-cell-barcode-search-configuration
description: Use when processing concatenated long-read single-cell cDNA data (PacBio
  MAS-Seq/MAS-ISO-seq or ONT) with nanoranger and a protocol-specific adapter barcode
  pattern (e.g., TruSeq adapter) must be configured to segment reads and recover cell
  barcodes/UMIs.
when_to_use_negative:
- Input is short-read scRNA-seq data with whitelisted 10x Genomics barcodes — nanoranger's
  barcode search targets concatenated long reads from intermediate single-cell cDNA
  libraries
- Reads are already deconcatenated into individual molecules with barcodes assigned,
  so no adapter-barcode-driven segmentation is needed
- Genomic (non-cDNA) long reads without single-cell barcode structure, where segment
  demultiplexing by adapter barcode is not applicable
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: nanoranger
  role: Long-read single-cell processing workflow whose barcode-search step (e.g.,
    TruSeq-adapter search for MAS-ISO-seq) is configured to segment concatenated reads
    and quantify molecules per gene/cell
  repo: github.com/mehdiborji/nanoranger
- name: Pacific Biosciences
  role: Sequencing platform producing the concatenated MAS-Seq/MAS-ISO-seq long reads
    that the configured barcode search processes
- name: liviuspenter/nanoranger.R
  role: Downstream R package that collapses UMIs (collapsed via starcode within a
    Levenshtein distance of 3) and extracts variants/isoforms from nanoranger output
  repo: https://github.com/liviuspenter/nanoranger.R
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:e6b99602cd94e3b041bf2b9a85ace5ee1d01ba889a035afda861528135d965d5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pacific Biosciences
  - liviuspenter/nanoranger.R
  - nanoranger
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# single-cell-barcode-search-configuration

## Summary

Configure the nanoranger workflow's barcode search (e.g., the TruSeq-adapter barcode search for PacBio MAS-Seq/MAS-ISO-seq runs) so concatenated long reads from single-cell cDNA libraries are correctly split into per-cell segments with cell barcodes and UMIs assigned before quantification.

## When to use

Use when processing concatenated long-read single-cell cDNA data (PacBio MAS-Seq/MAS-ISO-seq or ONT) with nanoranger and a protocol-specific adapter barcode pattern (e.g., TruSeq adapter) must be configured to segment reads and recover cell barcodes/UMIs.

## When NOT to use

- Input is short-read scRNA-seq data with whitelisted 10x Genomics barcodes — nanoranger's barcode search targets concatenated long reads from intermediate single-cell cDNA libraries
- Reads are already deconcatenated into individual molecules with barcodes assigned, so no adapter-barcode-driven segmentation is needed
- Genomic (non-cDNA) long reads without single-cell barcode structure, where segment demultiplexing by adapter barcode is not applicable

## Inputs

- Concatenated long-read single-cell sequencing data (PacBio MAS-Seq/MAS-ISO-sem, e.g., BAM/FASTQ from a Revio run such as DATA-Revio-PBMC-2)
- Protocol-specific adapter barcode pattern (e.g., TruSeq adapter) for the barcode search step
- Reference transcriptome annotation (e.g., GENCODE v44 FASTA/GTF)

## Outputs

- Segmented per-read output with cell barcodes and UMIs assigned (per-read segment counts)
- Per-gene molecule counts (e.g., nanoranger mean 141 vs skera 55 molecules/gene in the PBMC benchmark)
- Per-cell molecule counts (e.g., nanoranger mean 2580 vs skera 1650 molecules/cell)
- Side-by-side comparison table such as nanoranger_vs_skera_comparison.csv

## How to apply

Obtain the concatenated long-read dataset (e.g., the public PacBio Revio MAS-Seq run DATA-Revio-PBMC-2 from downloads.pacbcloud.com) and clone the nanoranger workflow from github.com/mehdiborji/nanoranger. Supply the reference transcriptome (e.g., GENCODE v44) as the alignment target, and configure the barcode search step to look for the adapter barcode appropriate to the library chemistry — for MAS-ISO-seq PBMC processing this is the TruSeq-adapter barcode search, run as described in the Methods. The nanoranger workflow starts from intermediate single-cell cDNA libraries, searches reads for the configured barcode to segment concatenated molecules, and aligns segments against the transcriptome to extract per-read segment counts and per-gene/per-cell molecule counts (with UMIs collapsed downstream, e.g., via starcode at a Levenshtein distance of 3 in nanoranger.R). Correct configuration is judged by whether segmentation and quantification metrics fall in expected ranges and correlate with an orthogonal tool: in the PBMC benchmark, nanoranger gave a median of 12 segments/read (range 0–37) versus skera's 15 (range 0–16), while molecules per gene (141 vs 55) and per cell (2580 vs 1650) were highly correlated (r = 0.88 and 0.97) and consistently higher with nanoranger.

## Related tools

- **nanoranger** (Long-read single-cell processing workflow whose barcode-search step (e.g., TruSeq-adapter search for MAS-ISO-seq) is configured to segment concatenated reads and quantify molecules per gene/cell) — github.com/mehdiborji/nanoranger
- **Pacific Biosciences** (Sequencing platform producing the concatenated MAS-Seq/MAS-ISO-seq long reads that the configured barcode search processes)
- **liviuspenter/nanoranger.R** (Downstream R package that collapses UMIs (collapsed via starcode within a Levenshtein distance of 3) and extracts variants/isoforms from nanoranger output) — https://github.com/liviuspenter/nanoranger.R

## Evaluation signals

- nanoranger_vs_skera_comparison.csv exists, is well-formed CSV, and contains rows for both segments-per-read and molecules-per-gene metrics
- Reported skera reference values (12 vs 15 segments/read; 141 vs 55 molecules/gene) appear as benchmark columns in the comparison output
- Nanoranger per-read segment counts fall in the reported range (median 12, range 0–37) for the PBMC MAS-Seq benchmark
- Molecule counts per gene and per cell show high correlation with the orthogonal skera-based processing (r = 0.88 and 0.97)
- Every segmented read carries an assigned cell barcode and UMI derived from the configured adapter barcode search

## Limitations

- Segment-per-read estimates differ between tools: nanoranger found consistently fewer segments per read (median 12 vs skera's median 15) despite higher molecule counts, so absolute segmentation counts are tool-dependent
- UMIs in 10x Genomics libraries lack a whitelist, so UMI error correction downstream relies on distance-based collapsing (Levenshtein distance of 3) rather than exact whitelist matching
- The genotyping ceiling is determined by the low, incomplete representation of many transcripts and cannot merely be overcome through deeper sequencing

## Evidence

- [readme] nanoranger is a processing tool for long-read single-cell transcriptomics: "nanoranger is a processing tool for long-read single-cell transcriptomics as described in our Nature Communications paper"
- [abstract] starts from intermediate single-cell cDNA libraries: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [readme] input data from 10x Genomics cDNA libraries with ONT or PacBio: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification"
- [readme] UMI collapsing at Levenshtein distance 3: [evidence span withheld — anchor status: not_found]
