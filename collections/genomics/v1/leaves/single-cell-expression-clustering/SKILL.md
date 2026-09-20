---
name: single-cell-expression-clustering
description: Use when analyzing scywalker long-read single-cell (10x Oxford nanopore
  or PacBio) data and you need expression-based cell groups, cell-type assignments,
  and per-cell-type pseudobulk gene/isoform count tables per sample.
when_to_use_negative:
- No cell-specific markersets are available and scSorter-based typing/pseudobulk is
  required without falling back to ScType only
- Data are not single-cell long-read cDNA (e.g. bulk RNA-seq or short-read-only data);
  scywalker targets 10x Oxford nanopore (or PacBio with -pacbioindex) data
- Only empty droplets/background barcodes are of interest — use the _raw counts instead
  of filtered cell-type pseudobulk outputs
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scywalker
  role: End-to-end workflow that orchestrates EmptyDrops filtering, Seurat clustering,
    ScType/scSorter cell typing, and pseudobulk generation
  repo: https://github.com/derijkp/scywalker
- name: genomecomb
  role: Framework in which scywalker is largely implemented, providing core algorithms
    and tsv query utilities
  repo: https://github.com/derijkp/genomecomb
- name: emptydrops
  role: Classifies barcodes as real cells (is_cell=1) using -sc_expectedcells to filter
    the count matrices
- name: scsorter
  role: Assigns cell types to expression clusters from the -cellmarkerfile marker
    tsv; without it, cell typing/pseudobulk via scSorter is skipped
- name: sctype
  role: Assigns cell types to clusters using its internal database for the given tissue
- name: IsoQuant
  role: Underlying isoform reconstruction/quantification whose per-cell gene and isoform
    counts feed the clustering and pseudobulk steps
  repo: https://github.com/ablab/IsoQuant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:25defa857fe89260357594d19c86dd6038451ff09a4a7be41e84ea46f156393f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - IsoQuant
  - emptydrops
  - genomecomb
  - scsorter
  - sctype
  - scywalker
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# single-cell-expression-clustering

## Summary

In scywalker, single-cell expression clustering retains only EmptyDrops-approved cells, groups them with Seurat clustering (resolution 0.5), assigns cell types via ScType (internal tissue database) and scSorter (marker tsv), then aggregates filtered per-cell gene/isoform counts into per-cell-type pseudobulk tables.

## When to use

Use when analyzing scywalker long-read single-cell (10x Oxford nanopore or PacBio) data and you need expression-based cell groups, cell-type assignments, and per-cell-type pseudobulk gene/isoform count tables per sample.

## When NOT to use

- No cell-specific markersets are available and scSorter-based typing/pseudobulk is required without falling back to ScType only
- Data are not single-cell long-read cDNA (e.g. bulk RNA-seq or short-read-only data); scywalker targets 10x Oxford nanopore (or PacBio with -pacbioindex) data
- Only empty droplets/background barcodes are of interest — use the _raw counts instead of filtered cell-type pseudobulk outputs

## Inputs

- scywalker per-sample gene count files (_raw and _filtered tsv)
- scywalker per-sample isoform count files
- sample -sc_expectedcells value for EmptyDrops
- cellmarkerfile marker tsv (-cellmarkerfile; columns: marker, celltype obligatory; tissue, markertype, weight optional)
- tissue designation for ScType's internal database

## Outputs

- filtered gene/isoform count files containing only EmptyDrops-approved cells (is_cell flag)
- per-cell cell-type assignments with uncertain calls removed in group_filtered
- pseudobulk gene count tables per cell type for each sample
- pseudobulk isoform count tables (weighed, unique, strict) per cell type for each sample
- cross-sample pseudobulk comparison count files in the compar subdirectory

## How to apply

Read the per-sample gene and isoform count files and apply EmptyDrops filtering using the sample's -sc_expectedcells value, keeping only cells flagged is_cell=1 (the _filtered files; _raw files keep all detected cells). Cluster retained cells with Seurat at resolution 0.5 to define expression-based groups. Assign cell types two ways: ScType using its internal database for the given tissue, and scSorter driven by the -cellmarkerfile marker tsv (marker and celltype columns obligatory; tissue, markertype, weight optional), recording uncertain assignments in group_filtered. Aggregate filtered cells into pseudobulk gene and isoform count tables per cell type for each sample, computing weighed (multimapping reads contribute 1/N), unique, and strict (unique reads covering >=90% of the transcript) isoform counts. Merge novel isoforms with matching splice junctions across samples and write cross-sample pseudobulk comparison files into the compar subdirectory.

## Related tools

- **scywalker** (End-to-end workflow that orchestrates EmptyDrops filtering, Seurat clustering, ScType/scSorter cell typing, and pseudobulk generation) — https://github.com/derijkp/scywalker
- **genomecomb** (Framework in which scywalker is largely implemented, providing core algorithms and tsv query utilities) — https://github.com/derijkp/genomecomb
- **emptydrops** (Classifies barcodes as real cells (is_cell=1) using -sc_expectedcells to filter the count matrices)
- **scsorter** (Assigns cell types to expression clusters from the -cellmarkerfile marker tsv; without it, cell typing/pseudobulk via scSorter is skipped)
- **sctype** (Assigns cell types to clusters using its internal database for the given tissue)
- **IsoQuant** (Underlying isoform reconstruction/quantification whose per-cell gene and isoform counts feed the clustering and pseudobulk steps) — https://github.com/ablab/IsoQuant

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- Filtered count files contain only EmptyDrops-approved cells (every retained barcode has is_cell=1), while _raw files still contain all detected cells
- Every retained cell carries a cell-type assignment in group_filtered, with uncertain calls removed
- Per-sample cluster/cell-type recovery matches expected composition (e.g. the five expected cell-line clusters in the scmixology2 benchmark)
- Pseudobulk tables are complete for all samples and cell types, with weighed/unique/strict isoform count columns populated
- Cross-sample pseudobulk comparison count files exist in the compar subdirectory

## Limitations

- scSorter is only run when a -cellmarkerfile is given; otherwise scsorter will not determine celltypes or make pseudobulk files
- Uncertain cell-type assignments must be filtered (group_filtered) for tools that support this; raw group labels may contain uncertain calls
- The isoquant-based code can hang or crash on very different organelle data, so organelles must be declared via scywalker_makerefdir -organelles
- Strict isoform counts require >=90% transcript coverage, so partially covered isoforms are excluded from strict counts

## Evidence

- [methods] Cluster the retained cells (Seurat, resolution 0.5) to define expression-based cell groups.: [evidence span withheld — anchor status: not_found]
- [results] is_cell : 1 if emptydrops categorized the cell as real, 0 if not: "is_cell : 1 if emptydrops categorized the cell as real, 0 if not"
- [results] counts_strict: only unique reads that cover >= 90% of the transcript: "counts_strict: only unique reads that cover >= 90% of the transcript"
- [methods] however sctype will use its internal database with the given tissue: "however sctype will use its internal database 
    with the given tissue"
- [methods] If not given, scsorter will not be used to determine celltypes and making pseudobulk files: "If not given, scsorter will not
    be used to determine celltypes and making pseudobulk files"
- [readme] If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts for each sample: "If cell specific markersets are provided, it will also assign cell-types
and generate pseudobulk (per cell-type) counts for each sample"
