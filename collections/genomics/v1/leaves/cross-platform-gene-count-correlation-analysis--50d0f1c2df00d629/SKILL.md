---
name: cross-platform-gene-count-correlation-analysis
description: Use when a long-read (Nanopore/PacBio) single-cell count matrix must
  be validated against a short-read (Illumina Cell Ranger) matrix from the same sample,
  e.g. checking cellBC/UMI-assigned gene counts agree across platforms before isoform-level
  analyses.
when_to_use_negative:
- Input is raw Nanopore/Illumina reads or SAM/BAM records rather than an already-generated
  gene x cellBC count matrix (use the Sicelore matrix-generation workflow first)
- The goal is isoform, SNP/RNA-editing, or fusion calling rather than validating gene-count
  agreement between platforms
- The Nanopore and Illumina data come from different samples, so shared cell barcodes
  do not represent the same cells
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Sicelore
  role: Produces the Nanopore gene x cellBC count matrix (PREFIX_genematrix.txt) via
    cell barcode/UMI assignment and matrix generation, the long-read side of the correlation
  repo: https://github.com/ucagenomix/sicelore
- name: minimap2
  role: Maps Nanopore reads/molecule consensus sequences to the reference genome upstream
    of barcode/UMI assignment and count matrix generation
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Manipulates SAM/BAM records (Illumina possorted_genome_bam.bam and Nanopore
    SAM) parsed for cell barcode and UMI information
  repo: https://github.com/samtools/samtools
- name: 10x Genomics Cell Ranger
  role: Generates the Illumina short-read filtered matrix and barcodes.tsv used as
    the reference platform for correlation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:fcd17b6798e35c76d2b75516153a5f4f97197dc8716fe88a0f7ef667c7cf2796
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore
  tools:
  - 10x Genomics Cell Ranger
  - Illumina
  - Sicelore
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cross-platform-gene-count-correlation-analysis

## Summary

Benchmarks Nanopore long-read single-cell gene/UMI count matrices (Sicelore PREFIX_genematrix.txt) against same-sample Illumina Cell Ranger matrices by intersecting cell barcodes, computing per-cell gene/UMI totals and per-gene/per-cell Pearson correlations, to verify the long-read data faithfully represent the 10x Genomics transcriptome despite additional full-length PCR and library preparation.

## When to use

Use when a long-read (Nanopore/PacBio) single-cell count matrix must be validated against a short-read (Illumina Cell Ranger) matrix from the same sample, e.g. checking cellBC/UMI-assigned gene counts agree across platforms before isoform-level analyses.

## When NOT to use

- Input is raw Nanopore/Illumina reads or SAM/BAM records rather than an already-generated gene x cellBC count matrix (use the Sicelore matrix-generation workflow first)
- The goal is isoform, SNP/RNA-editing, or fusion calling rather than validating gene-count agreement between platforms
- The Nanopore and Illumina data come from different samples, so shared cell barcodes do not represent the same cells

## Inputs

- Sicelore Nanopore gene-level count matrix (PREFIX_genematrix.txt, geneId x cellBC)
- Illumina Cell Ranger filtered gene-cell count matrix
- 10x Genomics Cell Ranger cell-associated barcode list (barcodes.tsv) for the 1121-cell dataset

## Outputs

- Per-cell detected gene counts and UMI totals for Nanopore and Illumina platforms
- Pearson correlation of gene counts across cells and of UMI counts across cells
- Per-cell per-gene expression Pearson correlations and their mean across shared cells
- comparison_report.csv of computed metrics vs reported benchmark values

## How to apply

Load the Sicelore gene-level Nanopore count matrix (PREFIX_genematrix.txt, geneId x cellBC) and the Illumina Cell Ranger filtered matrix with pandas or R. Intersect the shared cell barcodes (here the 1121-cell dataset) and align the gene identifier space between the two matrices. Compute per-cell detected gene counts and per-cell UMI totals for both platforms, then Pearson-correlate gene counts and UMI counts across cells; additionally compute the per-gene expression Pearson correlation for each shared cell and average across cells. Accept the comparison when gene-count and UMI-count Pearson r approximate 0.99 and mean per-cell expression correlation approximates 0.90, matching the reported benchmark of a median 2427 genes and 6047 UMIs per cell. Rationale: this demonstrates that the additional full-length PCR amplification and Nanopore library preparation do not skew the long-read dataset relative to the same-sample Illumina data.

## Related tools

- **Sicelore** (Produces the Nanopore gene x cellBC count matrix (PREFIX_genematrix.txt) via cell barcode/UMI assignment and matrix generation, the long-read side of the correlation) — https://github.com/ucagenomix/sicelore
- **minimap2** (Maps Nanopore reads/molecule consensus sequences to the reference genome upstream of barcode/UMI assignment and count matrix generation) — https://github.com/lh3/minimap2
- **samtools** (Manipulates SAM/BAM records (Illumina possorted_genome_bam.bam and Nanopore SAM) parsed for cell barcode and UMI information) — https://github.com/samtools/samtools
- **10x Genomics Cell Ranger** (Generates the Illumina short-read filtered matrix and barcodes.tsv used as the reference platform for correlation)

## Examples

```
import pandas as pd; np_mat = pd.read_csv('PREFIX_genematrix.txt', sep='\t', index_col=0); il_mat = pd.read_csv('filtered_matrix.tsv', sep='\t', index_col=0); shared = np_mat.columns.intersection(il_mat.columns); r = np_mat.sum(0)[shared].corr(il_mat.sum(0)[shared])
```

## Evaluation signals

- Gene-count Pearson correlation across shared cells approximates the reported r = 0.99
- UMI-count Pearson correlation across shared cells approximates the reported r = 0.99
- Mean per-cell per-gene expression correlation approximates the reported r = 0.90
- Nanopore per-cell metrics center on the reported median of 2427 genes and 6047 UMIs per cell
- Number of intersected cell barcodes matches the expected 1121-cell dataset before computing correlations

## Limitations

- Correlation is computed only on cell barcodes shared between the Nanopore and Cell Ranger matrices; Nanopore-only or Illumina-only cells are excluded
- Per-cell expression correlation (~0.90) is lower than aggregate gene/UMI-count correlations (~0.99), so per-gene agreement within single cells is expectedly noisier
- The Sicelore workflow depends on accurate cellBC/UMI assignment; sequencing errors can mis-assign UMIs without UMI-guided error correction

## Evidence

- [results] The Nanopore data reflect a median of 2427 genes and 6047 UMIs per cell, with a good correlation between Nanopore and Illumina gene counts (r = 0.99): [evidence span withheld — anchor status: not_found]
- [abstract] ScNaUmi-seq combines Nanopore throughput with accurate cellBC/UMI assignment: "We introduce ScNaUmi-seq, an approach that combines the high throughput of Oxford Nanopore sequencing with an accurate cell barcode and UMI assignment strategy"
- [methods] Step 4a generates gene matrices from annotated long-reads: [evidence span withheld — anchor status: not_found]
- [readme] Sicelore produces isoform/gene/junction count matrices for standard single-cell analysis: [evidence span withheld — anchor status: external_doc]
