---
name: sctype-celltype-annotation
description: Use when running scywalker on 10x Oxford Nanopore or PacBio single-cell
  FASTQ data and cell-type assignments plus per-cell-type pseudobulk counts are needed
  from a marker/cell-type file.
when_to_use_negative:
- No cell-specific markerset is provided — scsorter will not be used to determine
  celltypes and make pseudobulk files, and the scywalker test dataset markers (markers_chr17.tsv)
  are explicitly not good markers, only made so celltyping can run on that limited
  dataset
- Input is short-read single-cell data — scywalker targets full-length single-cell
  cDNA from Oxford Nanopore or PacBio long-read sequencing
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scywalker
  role: End-to-end workflow that runs cell typing and pseudobulk generation in a single
    command when a cellmarkerfile is given
  repo: https://github.com/derijkp/scywalker
- name: genomecomb
  role: Framework in which scywalker (and thus celltyping) is largely implemented,
    bundled in the scywalker distribution
  repo: https://github.com/derijkp/genomecomb
- name: scsorter
  role: Marker-based cell type assignment used by scywalker when a cellmarkerfile
    is supplied; its group assignments are reported with group_filtered removing uncertains
- name: sctype
  role: Alternative cell typing that uses its internal database with the given tissue
- name: minimap2
  role: Aligner whose PacBio-specific index (built with -pacbioindex 1) must be present
    to run the workflow on PacBio data
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:762e51063932569dbcf7974a6a778926608a43b8a8c5894fdb5294d138c76397
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - genomecomb
  - minimap2
  - scsorter
  - sctype
  - scywalker
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sctype-celltype-annotation

## Summary

Cell-type assignment for scywalker single-cell long-read data: when a cell-specific markerset is supplied via -cellmarkerfile, scywalker runs scsorter/sctype to assign cell types and generate pseudobulk (per cell-type) counts, with sctype able to use its internal database for a given tissue.

## When to use

Use when running scywalker on 10x Oxford Nanopore or PacBio single-cell FASTQ data and cell-type assignments plus per-cell-type pseudobulk counts are needed from a marker/cell-type file.

## When NOT to use

- No cell-specific markerset is provided — scsorter will not be used to determine celltypes and make pseudobulk files, and the scywalker test dataset markers (markers_chr17.tsv) are explicitly not good markers, only made so celltyping can run on that limited dataset
- Input is short-read single-cell data — scywalker targets full-length single-cell cDNA from Oxford Nanopore or PacBio long-read sequencing

## Inputs

- scywalker project/sample directory with FASTQ files (fastq subdirectory per sample, or -samplesheet tsv with 'sample' and 'seqfiles' fields)
- Reference directory (e.g. genomecomb refdb_hg38) built via scywalker_makerefdir with PacBio minimap2 index when needed
- Cell marker file with 'marker' and 'celltype' columns (and tissue where applicable)
- -sc_expectedcells value required for emptydrops cell filtering

## Outputs

- Per-cell cell-type assignments (group_filtered column; uncertain calls removed)
- Pseudobulk (per cell-type) count tables per sample
- Cross-sample pseudobulk comparison count files
- Filtered cell count files with is_cell flag and emptydrops-approved cells

## How to apply

Run the single scywalker command on the project/sample directory with a -cellmarkerfile containing 'marker' and 'celltype' columns (plus tissue where applicable) so scsorter and sctype can determine cell types and make pseudobulk files; omitting the marker file disables celltyping. For PacBio data, build the reference with scywalker_makerefdir using '-pacbioindex 1' and run scywalker with '-preset pacbio', a required -sc_expectedcells for emptydrops cell filtering, and low thread counts (~4-8). After emptydrops filtering, take per-cell scsorter cell-type assignments from the group_filtered column (where uncertain calls are removed) to build a cell-type assignment table and UMAP colored by assigned cell type.

## Related tools

- **scywalker** (End-to-end workflow that runs cell typing and pseudobulk generation in a single command when a cellmarkerfile is given) — https://github.com/derijkp/scywalker
- **genomecomb** (Framework in which scywalker (and thus celltyping) is largely implemented, bundled in the scywalker distribution) — https://github.com/derijkp/genomecomb
- **scsorter** (Marker-based cell type assignment used by scywalker when a cellmarkerfile is supplied; its group assignments are reported with group_filtered removing uncertains)
- **sctype** (Alternative cell typing that uses its internal database with the given tissue)
- **minimap2** (Aligner whose PacBio-specific index (built with -pacbioindex 1) must be present to run the workflow on PacBio data)

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- is_cell flag equals 1 for cells emptydrops categorized as real and 0 for others; the _filtered count files contain only emptydrops-approved cells while _raw files cover all detected cells
- group_filtered column removes uncertain cell-type assignments (for tools that support this), so no uncertain calls remain in the final assignment table
- Recovered populations match canonical PBMC cell types (T cells, B cells, monocytes, NK cells) as reported in Supplementary Fig. S16 for the PacBio PBMC demonstration
- Number of filtered cells is consistent with the expected number of cells supplied via -sc_expectedcells
- Without a marker file, no cell types or pseudobulk files are produced, confirming the cellmarkerfile is what triggers scsorter/sctype

## Limitations

- The isoquant based code often hangs or crashes on the very different organelle data, so organelles must be specified at reference construction
- Celltyping quality depends on the marker file; the test dataset's markers_chr17.tsv genes are not good markers, just made so celltyping can at least run
- Cell typing and pseudobulk generation are skipped entirely when no cellmarkerfile is given

## Evidence

- [intro] If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts for each sample: "If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts for each sample"
- [methods] If not given, scsorter will not be used to determine celltypes and making pseudobulk files: "If not given, scsorter will not
    be used to determine celltypes and making pseudobulk files"
- [methods] however sctype will use its internal database with the given tissue: "however sctype will use its internal database 
    with the given tissue"
- [results] group_filtered : assigned group/cell type filtering put uncertains (foor tools that support this): "group_filtered : assigned group/cell type filtering put uncertains (foor tools that support this)"
- [readme] The "marker" genes in the included markers_chr17.tsv are not good markers, they are just made to allow celltyping to at least run: "The "marker" genes in the included markers_chr17.tsv are not good
# markers, they are just made to allow celltyping to at least run on this specific limited dataset"
