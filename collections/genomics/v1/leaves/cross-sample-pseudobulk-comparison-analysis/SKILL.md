---
name: cross-sample-pseudobulk-comparison-analysis
description: Use when scywalker long-read single-cell runs (10x Oxford nanopore or
  PacBio) have per-sample per-cell gene/isoform count files plus cell-type assignments
  and you need per-cell-type pseudobulk counts comparable across samples.
when_to_use_negative:
- 'No cell-specific markersets are provided: scsorter will not be used to determine
  celltypes and making pseudobulk files'
- Input is short-read 10x data or a pre-aggregated feature-barcode matrix rather than
  scywalker long-read per-cell count output
- The reference lacks proper markers (e.g. the test markers_chr17.tsv contains placeholder
  markers 'just made to allow celltyping to at least run'), so cell-type pseudobulk
  labels are not biologically meaningful
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scywalker
  role: End-to-end pipeline that runs cell barcode assignment, isoform reconstruction/quantification,
    cell typing, pseudobulk generation, and cross-sample comparison count files in
    one command
  repo: https://github.com/derijkp/scywalker
- name: genomecomb
  role: Framework in which scywalker is implemented; provides cg/sw tooling for querying
    tsv count tables and compar outputs
  repo: https://github.com/derijkp/genomecomb
- name: IsoQuant
  role: Underlying isoform reconstruction and quantification engine whose per-cell
    gene/isoform counts feed the pseudobulk aggregation
  repo: https://github.com/ablab/IsoQuant
- name: scSorter
  role: Assigns cell types to Seurat clusters from the -cellmarkerfile markers, which
    define the pseudobulk grouping; skipped when no marker file is given
- name: sctype
  role: Cell-type assignment using its internal database for the given tissue, complementing
    scSorter for cluster labeling
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:68422ce0f8ce5aa20335e4691a8362db4760788454d10f7f9eb22548034fd045
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - IsoQuant
  - genomecomb
  - scSorter
  - sctype
  - scywalker
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cross-sample-pseudobulk-comparison-analysis

## Summary

Aggregates emptydrops-filtered per-cell gene and isoform counts from scywalker into per-cell-type pseudobulk tables per sample, and produces cross-sample pseudobulk comparison count files (in a compar subdirectory) enabling cell-type-specific comparison of gene and isoform expression, including novel isoforms, between samples.

## When to use

Use when scywalker long-read single-cell runs (10x Oxford nanopore or PacBio) have per-sample per-cell gene/isoform count files plus cell-type assignments and you need per-cell-type pseudobulk counts comparable across samples.

## When NOT to use

- No cell-specific markersets are provided: scsorter will not be used to determine celltypes and making pseudobulk files
- Input is short-read 10x data or a pre-aggregated feature-barcode matrix rather than scywalker long-read per-cell count output
- The reference lacks proper markers (e.g. the test markers_chr17.tsv contains placeholder markers 'just made to allow celltyping to at least run'), so cell-type pseudobulk labels are not biologically meaningful

## Inputs

- Per-sample scywalker gene count files (raw and filtered) with emptydrops is_cell flags
- Per-sample per-cell isoform count files with weighed, unique, and strict count columns
- Cell-type marker tsv file (-cellmarkerfile) with obligatory marker and celltype columns, optional tissue, markertype, weight
- Sample -sc_expectedcells value for EmptyDrops cell filtering
- Reference directory (scywalker_makerefdir output with fasta, gtf, and minimap2 indexes)

## Outputs

- Per-sample pseudobulk gene count tables per cell type
- Per-sample pseudobulk isoform count tables per cell type with weighed (1/N multimapping), unique, and strict (>=90% transcript coverage) counts
- Cross-sample pseudobulk comparison count files written to the compar subdirectory
- Per-cell group_filtered annotations with uncertain assignments removed

## How to apply

First read the per-sample gene and isoform count files produced by the scywalker run and apply EmptyDrops filtering using the sample's -sc_expectedcells value so only cells flagged is_cell=1 (real cells) are retained. Cluster the retained cells with Seurat (resolution 0.5) and assign cell types to clusters using ScType (internal database for the given tissue) and scSorter driven by the -cellmarkerfile marker tsv (marker and celltype fields obligatory; tissue, markertype, weight optional), recording uncertain assignments separately in group_filtered. Aggregate the filtered cells into pseudobulk (per cell-type) gene and isoform count tables for each sample, computing weighed isoform counts (multimapping reads give 1/N), unique counts, and strict counts (only unique reads covering >= 90% of the transcript). Merge novel isoforms with matching splice junctions across samples and write the cross-sample pseudobulk comparison count files into the compar subdirectory, enabling direct identification of cell-type-specific expression of novel isoforms. Validation: filtered count files must contain only emptydrops-approved cells, every cell must carry a cell-type assignment in group_filtered, and pseudobulk tables must be complete for all samples and cell types (e.g. the five expected cell-line clusters in the scmixology2 benchmark).

## Related tools

- **scywalker** (End-to-end pipeline that runs cell barcode assignment, isoform reconstruction/quantification, cell typing, pseudobulk generation, and cross-sample comparison count files in one command) — https://github.com/derijkp/scywalker
- **genomecomb** (Framework in which scywalker is implemented; provides cg/sw tooling for querying tsv count tables and compar outputs) — https://github.com/derijkp/genomecomb
- **IsoQuant** (Underlying isoform reconstruction and quantification engine whose per-cell gene/isoform counts feed the pseudobulk aggregation) — https://github.com/ablab/IsoQuant
- **scSorter** (Assigns cell types to Seurat clusters from the -cellmarkerfile markers, which define the pseudobulk grouping; skipped when no marker file is given)
- **sctype** (Cell-type assignment using its internal database for the given tissue, complementing scSorter for cluster labeling)

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- Filtered count files contain only emptydrops-approved cells (is_cell=1); _raw files provide counts for all detected cells
- Every retained cell carries a cell-type assignment in group_filtered, with uncertains removed for tools that support this
- Per-sample cluster/cell-type recovery matches expected composition (e.g. the five expected cell-line clusters in the scmixology2 benchmark)
- Pseudobulk tables are complete for all samples and cell types, and compar subdirectory files contain merged isoforms with matching splice junctions across samples
- Isoform strict counts only include unique reads covering >= 90% of the transcript, and multimapping reads are weighed (mapping to N genes gives each 1/N)

## Limitations

- The isoquant based code often hangs or crashes on very different organelle data; organelle chromosomes must be specified via scywalker_makerefdir -organelles
- Proper cell typing requires good markerset files; the test markers_chr17.tsv markers are not real markers and cannot yield meaningful cell types
- If no -cellmarkerfile is given, scsorter cell typing and pseudobulk file generation are skipped entirely
- Pseudobulk accuracy at isoform level depends on the strict-count definition (unique reads with >=90% transcript coverage), which excludes partially covered reads

## Evidence

- [intro] If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts for each sample: "If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts for each sample, and make count files allowing comparison between samples"
- [results] In this file (_filtered) only information on the emptydrops approved cells is given: "In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells"."
- [results] multimapping reads are weighed (if maps to N genes -> each gets 1/N count): "multimapping reads are weighed (if maps to N genes -> each gets 1/N count)"
- [results] counts_strict: only unique reads that cover >= 90% of the transcript: "counts_strict: only unique reads that cover >= 90% of the transcript"
- [methods] This information is required for filtering cells using emptydrops: "This information is required for filtering cells using emptydrops"
