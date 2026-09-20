---
name: single-cell-knee-plot-qc-filtering
description: Use when analyzing 10x single-cell/nuclei Oxford nanopore (or PacBio)
  long-read cDNA and you need to call real cells via knee-plot thresholds plus emptydrops,
  or to build/interpret the scywalker HTML QC report.
when_to_use_negative:
- Input is bulk long-read RNA-seq without cell barcodes — emptydrops-based cell calling
  and knee plots are not applicable
- You only need pseudobulk per-cell-type isoform counts; the knee-plot QC step concerns
  per-cell (droplet) quality assessment, not pseudobulk aggregation
- Short-read scRNA-seq data is available and a matched short-read pipeline is preferred
  — scywalker is designed to analyse 10x nanopore data without matching short reads
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scywalker
  role: End-to-end pipeline that performs cell barcode demultiplexing, emptydrops-based
    cell filtering, and assembles the interactive HTML QC report with knee plot and
    other QC panels
  repo: https://github.com/derijkp/scywalker
- name: minimap2
  role: Default aligner (minimap2_splice preset) producing the BAM from which cramino
    computes alignment metrics for the QC report
- name: cramino
  role: Computes alignment metrics from the minimap2 BAM in spliced mode for inclusion
    in the HTML QC report
- name: genomecomb
  role: Framework in which scywalker (including its QC/reporting module) is largely
    implemented
  repo: https://github.com/derijkp/genomecomb/
- name: scsorter
  role: Optional cell-type assignment feeding the per-cell group assignments shown
    on the UMAP panel; not used if no markerset is given
- name: sctype
  role: Alternative cell-typing tool using its internal tissue database, providing
    group assignments for the QC report UMAP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:a12fe704f38ba687a959a517dda0aca4edff89a5c0c9ac93c5a4361460f743bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - cramino
  - genomecomb
  - minimap2
  - scsorter
  - sctype
  - scywalker
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# single-cell-knee-plot-qc-filtering

## Summary

Computes knee-plot thresholds and applies emptydrops-based cell QC to distinguish real cells from empty droplets in scywalker's long-read single-cell analysis, then visualizes the approved cells in the interactive HTML QC report alongside read-length, genes-per-cell and UMAP panels.

## When to use

Use when analyzing 10x single-cell/nuclei Oxford nanopore (or PacBio) long-read cDNA and you need to call real cells via knee-plot thresholds plus emptydrops, or to build/interpret the scywalker HTML QC report.

## When NOT to use

- Input is bulk long-read RNA-seq without cell barcodes — emptydrops-based cell calling and knee plots are not applicable
- You only need pseudobulk per-cell-type isoform counts; the knee-plot QC step concerns per-cell (droplet) quality assessment, not pseudobulk aggregation
- Short-read scRNA-seq data is available and a matched short-read pipeline is preferred — scywalker is designed to analyse 10x nanopore data without matching short reads

## Inputs

- FASTQ files for 10x single-cell/nuclei long-read cDNA (in a sample directory fastq/ subdirectory or project directory)
- scywalker reference directory (e.g. built with scywalker_makerefdir from genome fasta + transcripts.gtf, or a genomecomb refdb)
- Per-cell gene count files (sc_gene_counts_filtered with emptydrops-approved cells; _raw for all detected cells)
- Per-cell cell-type group assignments from scsorter/sctype calling and typing modules
- minimap2_splice BAM alignments (input to cramino spliced-mode metric computation)
- Optional cell marker file (-cellmarkerfile) for cell-type assignment

## Outputs

- Interactive HTML QC report (report.html) containing cramino alignment metrics, knee plot with emptydrops-approved cells marked, read-length histograms, genes-per-cell histogram, interval-overlap summary and cell-type UMAP panel
- Filtered per-cell gene count files (sc_gene_counts_filtered) restricted to emptydrops-approved cells, alongside _raw counts for all detected cells
- Per-cell QC table including the is_cell flag (1 if emptydrops categorized the cell as real, 0 if not)

## How to apply

Run scywalker on the sample/project directory so that cellbarcodes are found and assigned, reads are aligned with minimap2_splice, and per-cell gene counts are produced. The reporting module loads the per-cell gene counts (emptydrops-approved filtered set) and per-cell cell-type group assignments, computes knee-plot thresholds, and applies the emptydrops-based cell QC filters to mark approved cells on the plot (the is_cell flag is 1 if emptydrops categorized the cell as real). Alignment metrics are computed from the minimap2 BAM with cramino in spliced mode. The QC panels — knee plot, read-length histograms (overall and versus number of exons per read), genes-per-cell histogram, genomic interval overlap summary, and a UMAP of assigned cell types over the per-cell count matrix — are assembled into a single interactive HTML document (report.html).

## Related tools

- **scywalker** (End-to-end pipeline that performs cell barcode demultiplexing, emptydrops-based cell filtering, and assembles the interactive HTML QC report with knee plot and other QC panels) — https://github.com/derijkp/scywalker
- **minimap2** (Default aligner (minimap2_splice preset) producing the BAM from which cramino computes alignment metrics for the QC report)
- **cramino** (Computes alignment metrics from the minimap2 BAM in spliced mode for inclusion in the HTML QC report)
- **genomecomb** (Framework in which scywalker (including its QC/reporting module) is largely implemented) — https://github.com/derijkp/genomecomb/
- **scsorter** (Optional cell-type assignment feeding the per-cell group assignments shown on the UMAP panel; not used if no markerset is given)
- **sctype** (Alternative cell-typing tool using its internal tissue database, providing group assignments for the QC report UMAP)

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- report.html exists, is parseable HTML, and contains the cramino-derived metrics
- report.html contains the knee plot, read-length histogram, genes-per-cell histogram, interval-overlap and UMAP panels
- sc_gene_counts_filtered contains only emptydrops-approved cells while sc_gene_counts_raw covers all detected cells; per-cell rows carry a consistent is_cell flag (1 = real cell)
- Knee-plot threshold visually separates the emptydrops-approved cells from the background droplets on the plot
- Read-length histograms (overall and versus exons per read) are populated with reads from the minimap2_splice BAM

## Limitations

- The isoquant-based code often hangs or crashes on organelle sequences, so organelles must be specified at reference-building time (scywalker_makerefdir -organelles)
- Cell typing (and hence the UMAP panel) requires a cell-specific markerset; on limited test data the markers may not be biologically meaningful
- Multimapping reads are weighed (a read mapping to N genes gives each gene 1/N), and strict isoform counts require >= 90% transcript coverage, so count values depend on the counting mode chosen

## Evidence

- [abstract] Scywalker includes data quality control, cell type identification, and an interactive report.: "Scywalker includes data quality control, cell type identification, and an interactive report."
- [results] is_cell flag marks real cells: "is_cell : 1 if emptydrops categorized the cell as real, 0 if not"
- [results] filtered files contain only approved cells: "In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells"."
- [methods] emptydrops used for cell filtering: "This information is required for filtering cells using emptydrops"
- [readme] scywalker scope: 10x nanopore long-read data: "scywalker is a package designed to analyse single cell (10x) Oxford nanopore long read data (without the need for matching short read data)"
