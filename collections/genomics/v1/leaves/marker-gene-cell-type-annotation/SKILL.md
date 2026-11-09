---
name: marker-gene-cell-type-annotation
description: Use when single-cell count matrices (per-cell gene/isoform counts from
  scywalker) need cluster-based cell-type labels from marker gene sets, and per-cell-type
  pseudobulk counts are required per sample.
when_to_use_negative:
- 'No cell-specific markersets are provided: scsorter will not be used to determine
  celltypes and make pseudobulk files'
- The test/reference dataset is too limited for meaningful typing (e.g. the chromosome-17-only
  scywalker test data whose markers_chr17.tsv are not real markers)
- Cells have not yet passed EmptyDrops filtering — annotation should only run on is_cell=1
  cells
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scywalker
  role: End-to-end workflow that clusters retained cells (Seurat resolution 0.5),
    invokes ScType/scSorter for typing, and generates the pseudobulk tables
  repo: https://github.com/derijkp/scywalker
- name: scsorter
  role: Marker-file-driven cell-type assignment using the -cellmarkerfile tsv; also
    gates pseudobulk file generation
  repo: https://github.com/derijkp/scywalker
- name: sctype
  role: Cluster cell-type assignment using its internal marker database for the given
    tissue
  repo: https://github.com/derijkp/scywalker
- name: genomecomb
  role: Framework in which scywalker and the cell-typing/pseudobulk steps are implemented
  repo: https://github.com/derijkp/genomecomb
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:d3a06c46ce3c34bea200a5747768115a4480ff99e6a093ffc6da588273cc09ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - genomecomb
  - scsorter
  - sctype
  - scywalker
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# marker-gene-cell-type-annotation

## Summary

Assigns cell types to single-cell clusters in scywalker's long-read (Oxford nanopore 10x) single-cell workflow using ScType with its internal tissue database and scSorter driven by a user-supplied marker tsv, then propagates the annotations into per-cell-type pseudobulk gene and isoform count tables.

## When to use

Use when single-cell count matrices (per-cell gene/isoform counts from scywalker) need cluster-based cell-type labels from marker gene sets, and per-cell-type pseudobulk counts are required per sample.

## When NOT to use

- No cell-specific markersets are provided: scsorter will not be used to determine celltypes and make pseudobulk files
- The test/reference dataset is too limited for meaningful typing (e.g. the chromosome-17-only scywalker test data whose markers_chr17.tsv are not real markers)
- Cells have not yet passed EmptyDrops filtering — annotation should only run on is_cell=1 cells

## Inputs

- Per-sample per-cell gene and isoform count files produced by a scywalker run (emptydrops-filtered cells only)
- Marker gene tsv file passed via -cellmarkerfile (obligatory fields: marker, celltype; optional: tissue, markertype, weight)
- Tissue specification for ScType's internal marker database
- Sample parameter -sc_expectedcells used upstream for EmptyDrops cell calling

## Outputs

- Per-cell cell-type assignments including group and group_filtered (uncertains removed) columns
- Pseudobulk per-cell-type gene count tables for each sample
- Pseudobulk per-cell-type isoform count tables (weighed, unique, and strict counts)
- Cross-sample pseudobulk comparison count files written to the compar subdirectory

## How to apply

After EmptyDrops filtering using the sample's -sc_expectedcells value (keeping only cells with is_cell=1), cluster the retained cells with Seurat at resolution 0.5 to define expression-based groups. Assign cell types to the clusters with ScType, which uses its internal database for the given tissue, and in parallel with scSorter driven by the -cellmarkerfile marker tsv (fields: marker and celltype obligatory; tissue, markertype, weight optional). Record uncertain assignments in the group_filtered column, which removes uncertains for tools that support this. Only proceed to pseudobulk if cell types were assigned: scsorter will not be used for pseudobulk file generation if no marker file is given, since cell-to-cell-type results are combined with the per-cell gene and isoform files to generate pseudobulk tables with one column per cell type.

## Related tools

- **scywalker** (End-to-end workflow that clusters retained cells (Seurat resolution 0.5), invokes ScType/scSorter for typing, and generates the pseudobulk tables) — https://github.com/derijkp/scywalker
- **scsorter** (Marker-file-driven cell-type assignment using the -cellmarkerfile tsv; also gates pseudobulk file generation) — https://github.com/derijkp/scywalker
- **sctype** (Cluster cell-type assignment using its internal marker database for the given tissue) — https://github.com/derijkp/scywalker
- **genomecomb** (Framework in which scywalker and the cell-typing/pseudobulk steps are implemented) — https://github.com/derijkp/genomecomb

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- Every EmptyDrops-approved cell carries a cell-type assignment in the group_filtered column
- Per-sample cluster/cell-type recovery matches expected composition (e.g. the five expected cell-line clusters in the scmixology2 benchmark)
- Pseudobulk gene and isoform tables are complete for all samples and all cell types
- Pseudobulk isoform tables include all three count flavors: weighed (multimapping reads give 1/N), unique, and strict (>=90% transcript coverage)
- Cross-sample comparison count files exist in the compar subdirectory and novel isoforms with matching splice junctions are merged across samples

## Limitations

- scSorter typing and pseudobulk generation are skipped entirely when no -cellmarkerfile is given
- ScType depends on its internal database matching the given tissue; typing quality is bounded by that database
- The isoquant-based code often hangs or crashes on very different organelle data, affecting upstream counts that feed typing
- group_filtered removes uncertain assignments only for tools that support this; other tools' uncertainties remain unflagged

## Evidence

- [results] Cluster the retained cells (Seurat, resolution 0.5) to define expression-based cell groups.: [evidence span withheld — anchor status: not_found]
- [methods] scsorter will not be used to determine celltypes and making pseudobulk files: "If not given, scsorter will not
    be used to determine celltypes and making pseudobulk files"
- [methods] sctype will use its internal database with the given tissue: "however sctype will use its internal database 
    with the given tissue"
- [results] group_filtered : assigned group/cell type filtering put uncertains (foor tools that support this): "group_filtered : assigned group/cell type filtering put uncertains (foor tools that support this)"
- [full_text] The cell-to-cell-type results are used together with the per-cell gene and isoform files to generate pseudobulk files: [evidence span withheld — anchor status: not_found]
