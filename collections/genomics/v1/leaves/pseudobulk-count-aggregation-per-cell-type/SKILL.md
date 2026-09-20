---
name: pseudobulk-count-aggregation-per-cell-type
description: Use when long-read single-cell (10x nanopore or PacBio) counts must be
  converted into per-cell-type pseudobulk gene and isoform count tables for cross-sample
  comparison, given per-cell counts and cell-type annotations.
when_to_use_negative:
- 'No cell-specific markersets are available (-cellmarkerfile not given): scsorter
  will not be used to determine celltypes and pseudobulk files will not be made'
- Input is already bulk or pseudobulk count data — no per-cell barcode-level counts
  exist to aggregate
- Cells have not passed emptydrops filtering (is_cell flag) — only emptydrops-approved
  cells should enter the pseudobulk tables
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scywalker
  role: End-to-end workflow that assigns cell types and generates pseudobulk (per
    cell-type) counts for each sample, plus cross-sample comparison count files
  repo: https://github.com/derijkp/scywalker
- name: genomecomb
  role: Framework within which scywalker (including pseudobulk aggregation and compar
    count files) is largely implemented
  repo: https://github.com/derijkp/genomecomb
- name: scsorter
  role: Assigns cell types from the -cellmarkerfile marker tsv; required for making
    the pseudobulk files
- name: sctype
  role: Assigns cell types to Seurat clusters using its internal database for the
    given tissue
- name: emptydrops
  role: Filters cells (is_cell flags via -sc_expectedcells) so only approved cells
    enter the filtered counts used for pseudobulk aggregation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:cba3fa2af83dc5db91fd56a7c8f211c3a9c355300d3a008647ce2340e4039752
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - emptydrops
  - genomecomb
  - scsorter
  - sctype
  - scywalker
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pseudobulk-count-aggregation-per-cell-type

## Summary

Aggregates scywalker per-cell gene and isoform counts into pseudobulk (per cell-type) gene and isoform count tables for each sample, enabling cell-type-specific and cross-sample comparison of isoform expression, including novel isoforms.

## When to use

Use when long-read single-cell (10x nanopore or PacBio) counts must be converted into per-cell-type pseudobulk gene and isoform count tables for cross-sample comparison, given per-cell counts and cell-type annotations.

## When NOT to use

- No cell-specific markersets are available (-cellmarkerfile not given): scsorter will not be used to determine celltypes and pseudobulk files will not be made
- Input is already bulk or pseudobulk count data — no per-cell barcode-level counts exist to aggregate
- Cells have not passed emptydrops filtering (is_cell flag) — only emptydrops-approved cells should enter the pseudobulk tables

## Inputs

- Per-sample per-cell gene count files from a scywalker run
- Per-sample per-cell isoform count files from a scywalker run
- Cell-type annotations in group_filtered (from ScType/scSorter after Seurat clustering)
- Marker tsv via -cellmarkerfile (fields: marker and celltype obligatory; tissue, markertype, weight optional)
- emptydrops is_cell flags / -sc_expectedcells value defining approved cells

## Outputs

- Pseudobulk gene count table per cell type for each sample (cell types in separate columns)
- Pseudobulk isoform count tables with weighed, unique, and strict (>=90% transcript coverage) counts
- Cross-sample pseudobulk comparison count files in the compar subdirectory, with novel isoforms merged across samples by matching splice junctions

## How to apply

Run scywalker with a cell marker file (-cellmarkerfile) so it assigns cell types via Seurat clustering (resolution 0.5) plus ScType (internal database for the given tissue) and scSorter; record uncertain assignments in group_filtered. Then aggregate the emptydrops-filtered per-sample gene and isoform count files into pseudobulk tables per cell type for each sample. For isoforms, compute three count flavors: weighed (multimapping reads give each target 1/N), unique, and strict (only unique reads covering >=90% of the transcript). Merge novel isoforms with matching splice junctions across samples and write the cross-sample pseudobulk comparison count files into the compar subdirectory. If no marker file is given, scsorter is not used to determine celltypes or make pseudobulk files.

## Related tools

- **scywalker** (End-to-end workflow that assigns cell types and generates pseudobulk (per cell-type) counts for each sample, plus cross-sample comparison count files) — https://github.com/derijkp/scywalker
- **genomecomb** (Framework within which scywalker (including pseudobulk aggregation and compar count files) is largely implemented) — https://github.com/derijkp/genomecomb
- **scsorter** (Assigns cell types from the -cellmarkerfile marker tsv; required for making the pseudobulk files)
- **sctype** (Assigns cell types to Seurat clusters using its internal database for the given tissue)
- **emptydrops** (Filters cells (is_cell flags via -sc_expectedcells) so only approved cells enter the filtered counts used for pseudobulk aggregation)

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- Filtered count files contain only emptydrops-approved cells (is_cell = 1); the _raw files retain counts for all detected 'cells'
- Every retained cell carries a cell-type assignment in group_filtered (uncertain assignments removed for tools that support this)
- Per-sample cluster/cell-type recovery matches expected composition, e.g. the five expected cell-line clusters in the scmixology2 benchmark
- Pseudobulk tables are complete for all samples and cell types, with weighed/unique/strict isoform count columns present
- Cross-sample comparison files in the compar subdirectory exist and novel isoforms with matching splice junctions are merged across samples

## Limitations

- The isoquant-based code often hangs or crashes on very different organelle data, affecting downstream counts if organelles are not specified at reference build
- Cell typing is unreliable on limited/low-quality data — in the test dataset the marker genes were not good markers, only enough to let celltyping run
- Without a marker file, scsorter cell typing and pseudobulk file generation are skipped entirely

## Evidence

- [intro] If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts for each sample: "If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts for each sample"
- [results] multimapping reads are weighed: "multimapping reads are weighed (if maps to N genes -> each gets 1/N count)"
- [results] strict counts require 90% transcript coverage: "counts_strict: only unique reads that cover >= 90% of the transcript"
- [results] In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells".: "In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells"."
- [methods] If not given, scsorter will not be used to determine celltypes and making pseudobulk files: "If not given, scsorter will not
    be used to determine celltypes and making pseudobulk files"
