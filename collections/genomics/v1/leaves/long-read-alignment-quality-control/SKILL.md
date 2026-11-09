---
name: long-read-alignment-quality-control
description: Use when you need to generate, inspect, or validate the interactive HTML
  QC report of a scywalker long-read single-cell (10x ONT or PacBio) run from its
  BAM and per-cell count outputs.
when_to_use_negative:
- The run has not completed alignment (no minimap2_splice BAM) or cell calling (no
  _filtered/_raw gene count files) yet — the report modules require these outputs.
- You need the primary isoform/gene reconstruction or cell-barcode demultiplexing
  itself; this skill only summarizes and reports QC on existing outputs.
- Input is short-read single-cell data — scywalker is designed for full-length single-cell
  or single-nuclei cDNA long-read (ONT, or PacBio with appropriate options) data.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: scywalker
  role: end-to-end pipeline whose reporting module assembles cramino metrics, QC plots,
    and the UMAP panel into report.html
  repo: https://github.com/derijkp/scywalker
- name: cramino
  role: computes alignment metrics from the minimap2_splice BAM in spliced mode for
    the QC report
- name: minimap2
  role: default aligner (minimap2_splice, splice preset) producing the BAM whose metrics
    and interval overlaps are reported
- name: genomecomb
  role: framework in which scywalker (and its reporting) is largely implemented; cg/sw
    executables provide supporting tools
  repo: https://github.com/derijkp/genomecomb
- name: scsorter
  role: cell-type assignments shown on the UMAP panel when a cellmarkerfile is provided
- name: sctype
  role: alternative cell typing using its internal tissue database; assignments feed
    the UMAP identification panel
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:f23f9119bae101136d23ef4b7f6330f4278a48277f9c954dfb59f85832feaa4b
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# long-read-alignment-quality-control

## Summary

scywalker's QC reporting module summarizes a long-read single-cell run into an interactive HTML report: cramino alignment metrics computed in spliced mode from the minimap2_splice BAM, knee-plot with emptydrops-approved cells, read-length and genes-per-cell histograms, genomic interval overlap, and a UMAP panel of assigned cell types.

## When to use

Use when you need to generate, inspect, or validate the interactive HTML QC report of a scywalker long-read single-cell (10x ONT or PacBio) run from its BAM and per-cell count outputs.

## When NOT to use

- The run has not completed alignment (no minimap2_splice BAM) or cell calling (no _filtered/_raw gene count files) yet — the report modules require these outputs.
- You need the primary isoform/gene reconstruction or cell-barcode demultiplexing itself; this skill only summarizes and reports QC on existing outputs.
- Input is short-read single-cell data — scywalker is designed for full-length single-cell or single-nuclei cDNA long-read (ONT, or PacBio with appropriate options) data.

## Inputs

- minimap2_splice BAM file from the scywalker run
- per-cell gene count files (sc_gene_counts_filtered / _raw TSV) from the calling module
- per-cell cell-type group assignments (group / group_filtered) from scsorter or sctype typing
- emptydrops cell-barcode classifications (is_cell flags)

## Outputs

- interactive report.html containing cramino-derived alignment metrics
- knee plot with emptydrops-approved cells marked
- read-length histograms (overall and versus number of exons per read)
- genes-per-cell histogram
- genomic interval overlap summary of alignments
- UMMAP embedding panel of assigned cell types over the per-cell count matrix

## How to apply

1. Load the minimap2_splice BAM produced by the scywalker run and compute alignment metrics with cramino using its spliced mode, so spliced long RNA reads are scored appropriately. 2. Load the per-cell gene counts (the emptydrops-approved _filtered set) and per-cell cell-type group assignments from the calling and typing modules; note _raw files contain all detected 'cells'. 3. Compute knee-plot thresholds and apply the emptydrops-based cell QC filters to mark approved cells (is_cell = 1) on the plot. 4. Build the QC panels: overall read-length histograms, read length versus number of exons per read, genes-per-cell histogram, and a genomic interval overlap summary of the alignments. 5. Project the assigned cell types (scsorter or sctype, using markerset or the tissue-specific internal database) onto a UMAP embedding over the per-cell count matrix. 6. Assemble all metrics and plots into a single interactive HTML report.html; multimapping gene reads are weighted 1/N, and isoform strict counts require >= 90% transcript coverage, which contextualizes the reported metrics.

## Related tools

- **scywalker** (end-to-end pipeline whose reporting module assembles cramino metrics, QC plots, and the UMAP panel into report.html) — https://github.com/derijkp/scywalker
- **cramino** (computes alignment metrics from the minimap2_splice BAM in spliced mode for the QC report)
- **minimap2** (default aligner (minimap2_splice, splice preset) producing the BAM whose metrics and interval overlaps are reported)
- **genomecomb** (framework in which scywalker (and its reporting) is largely implemented; cg/sw executables provide supporting tools) — https://github.com/derijkp/genomecomb
- **scsorter** (cell-type assignments shown on the UMAP panel when a cellmarkerfile is provided)
- **sctype** (alternative cell typing using its internal tissue database; assignments feed the UMAP identification panel)

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- report.html exists in the sample/project output and is parseable HTML
- the report contains cramino-derived alignment metrics computed with cramino in spliced mode
- knee plot marks exactly the cells flagged as real (is_cell = 1) by emptydrops, and _filtered count files contain only emptydrops-approved cells
- read-length panels render both overall read length and read length versus number of exons per read
- genes-per-cell histogram, genomic interval overlap summary, and the cell-type UMAP over the per-cell count matrix are all present

## Limitations

- The isoquant-based code often hangs or crashes on organelle sequences, so organelles must be specified in the refdir (scywalker_makerefdir -organelles) before running; chromosome grouping may need -groupchromosomes for non-standard genome naming.
- Cell typing (and thus the UMAP panel) is only meaningful when good cell-specific markersets are provided; on the limited test dataset the included markers are 'not good markers, they are just made to allow celltyping to at least run'.
- Multimapping gene reads are weighted (1/N per gene), so gene-count-based QC panels reflect weighted counts rather than unique assignments.

## Evidence

- [abstract] Scywalker includes data quality control, cell type identification, and an interactive report.: "Scywalker includes data quality control, cell type identification, and an interactive report."
- [methods] The reporting module computes alignment metrics with cramino spliced mode: [evidence span withheld — anchor status: not_found]
- [results] _filtered vs _raw emptydrops cells: "In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells"."
- [full_text] read-length and knee panels from workflow: [evidence span withheld — anchor status: not_found]
- [readme] readme scywalker long-read scope: "scywalker is a package designed to analyse single cell (10x) Oxford nanopore long read data (without the need for matching short read data),"
