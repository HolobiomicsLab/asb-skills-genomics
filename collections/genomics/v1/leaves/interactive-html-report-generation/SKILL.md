---
name: interactive-html-report-generation
description: Use when a scywalker run has produced the minimap2_splice BAM, emptydrops-filtered
  gene counts, and cell-type assignments, and an interactive QC report summarizing
  them is needed.
when_to_use_negative:
- No cell-type markerset was supplied (scsorter/sctype typing was not run), so the
  UMAP/typing panel cannot be populated meaningfully.
- The input is a downstream analysis of an already-produced report (e.g. re-plotting
  extracted tables) rather than summarizing scywalker pipeline outputs.
- The run used the organelle-specific algorithm branch on organellar data, where the
  isoquant-based code path may hang or crash and report completeness is not guaranteed.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scywalker
  role: generates the report module that assembles metrics and plots into the interactive
    HTML report
  repo: https://github.com/derijkp/scywalker
- name: cramino
  role: computes alignment metrics from the minimap2 BAM in spliced mode for the report
- name: minimap2
  role: produces the minimap2_splice BAM whose alignment metrics are reported
- name: genomecomb
  role: framework within which scywalker (and its reporting) is implemented
  repo: https://github.com/derijkp/genomecomb
- name: scsorter
  role: cell typing whose assignments are projected onto the UMAP panel (if a markerset
    is given)
- name: sctype
  role: cell typing fallback using its internal database for the given tissue; assignments
    feed the UMAP panel
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:f4fe129a6ac20f25cc05ad6591925f49c4c8d1377fe6b297b732f9ad0624f3b1
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# interactive-html-report-generation

## Summary

Assembles scywalker pipeline outputs — cramino alignment metrics, knee plot with emptydrops-approved cells, read-length and genes-per-cell histograms, alignment interval-overlap summary, and a cell-type UMAP — into a single interactive HTML report for long-read single-cell quality control and cell type identification review.

## When to use

Use when a scywalker run has produced the minimap2_splice BAM, emptydrops-filtered gene counts, and cell-type assignments, and an interactive QC report summarizing them is needed.

## When NOT to use

- No cell-type markerset was supplied (scsorter/sctype typing was not run), so the UMAP/typing panel cannot be populated meaningfully.
- The input is a downstream analysis of an already-produced report (e.g. re-plotting extracted tables) rather than summarizing scywalker pipeline outputs.
- The run used the organelle-specific algorithm branch on organellar data, where the isoquant-based code path may hang or crash and report completeness is not guaranteed.

## Inputs

- minimap2_splice BAM file of aligned full-length single-cell reads
- per-cell gene counts file (emptydrops-approved filtered set, sc_gene_counts_filtered)
- per-cell cell-type group assignments from the calling and typing modules (e.g. group_filtered)
- cell marker file used for cell typing (e.g. -cellmarkerfile markers tsv)

## Outputs

- single interactive report.html containing cramino-derived alignment metrics
- knee plot with emptydrops-approved cells marked
- read-length histograms (overall and versus number of exons per read)
- genes-per-cell histogram
- genomic interval overlap summary of alignments
- UMAP embedding colored by assigned cell type

## How to apply

Load the minimap2_splice BAM produced by the scywalker run and compute alignment metrics with cramino in spliced mode. Load the per-cell gene counts (the emptydrops-approved filtered set) and the per-cell cell-type group assignments from the calling and typing modules. Compute knee-plot thresholds and apply the emptydrops-based cell QC filters to mark approved cells on the plot. Build the QC panels: read-length histograms (overall and versus number of exons per read), a genes-per-cell histogram, and a genomic interval overlap summary of the alignments. Project the assigned cell types (from scsorter/sctype) onto a UMAP embedding over the per-cell count matrix for the cell type identification panel, then assemble all metrics and plots into one interactive HTML document.

## Related tools

- **scywalker** (generates the report module that assembles metrics and plots into the interactive HTML report) — https://github.com/derijkp/scywalker
- **cramino** (computes alignment metrics from the minimap2 BAM in spliced mode for the report)
- **minimap2** (produces the minimap2_splice BAM whose alignment metrics are reported)
- **genomecomb** (framework within which scywalker (and its reporting) is implemented) — https://github.com/derijkp/genomecomb
- **scsorter** (cell typing whose assignments are projected onto the UMAP panel (if a markerset is given))
- **sctype** (cell typing fallback using its internal database for the given tissue; assignments feed the UMAP panel)

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- report.html exists in the sample/project output directory
- report.html is parseable HTML
- report contains the cramino-derived alignment metrics
- report contains the knee plot with emptydrops-approved cells marked
- report contains read-length, genes-per-cell, interval-overlap, and UMAP panels

## Limitations

- Cell typing (and hence the report's UMAP panel) only runs when a cell-specific markerset is provided; the test dataset's markers are explicitly not good markers, just enough to let celltyping run.
- The isoquant-based code often hangs or crashes on organelle data, which can affect upstream outputs feeding the report.
- Proper cell-type determination requires adequate data; the limited chromosome-17 test dataset cannot be properly cell-typed.

## Evidence

- [results] report includes cramino metrics and panels: [evidence span withheld — anchor status: not_found]
- [abstract] scywalker includes QC, cell typing, interactive report: "Scywalker includes data quality control, cell type identification, and an interactive report."
- [results] filtered files contain emptydrops-approved cells: "In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells"."
- [readme] markersets required for cell typing in README: "If cell specific markersets are provided, it will also assign cell-types
and generate pseudobulk (per cell-type) counts for each sample"
- [readme] example scywalker run with cellmarkerfile: "scywalker -v 1 -d 8 \
	-dmaxmem 64G \
	-refdir g17 \
	-sc_expectedcells 183 \
	-cellmarkerfile markers_chr17.tsv \
	-threads 6 \
	test10x"
