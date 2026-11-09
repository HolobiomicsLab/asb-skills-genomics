---
name: cell-type-annotation-visualization
description: Reference for annotating scywalker long-read single-nuclei per-cell gene
  counts with cell types from a markerset and visualizing the assignments on UMAP
  in the QC report.
when_to_use_negative:
- 'No cell-specific markerset is provided: scsorter will not be used to determine
  cell types and pseudobulk files are not made'
- Analysis is limited to a chromosome-subset test dataset (e.g., chromosome 17 only)
  where marker genes cannot properly determine cell types — scywalker's own test data
  notes celltyping can at least run but results are not meaningful
- Input lacks per-cell counts for emptydrops-approved cells (e.g., only raw counts
  over all detected 'cells' are available), so annotation and the UMAP panel cannot
  be restricted to real cells
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0099
tools:
- name: scywalker
  role: Orchestrates cell-type assignment from markersets, pseudobulk generation,
    and the UMAP panel in the interactive HTML report
  repo: https://github.com/derijkp/scywalker
- name: scsorter
  role: Determines cell types and makes pseudobulk files from the markerset; skipped
    if no markerset is given
- name: sctype
  role: Alternative cell-typing tool that can use its internal database with a given
    tissue
- name: genomecomb
  role: Framework in which scywalker (and hence celltyping/reporting) is implemented;
    provided in full within the scywalker distribution
  repo: https://github.com/derijkp/genomecomb
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:a9b65d69bc69bfafd77c3cd6d7729000b30a2ff1e2b6d0351ca672c5a2349039
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - cramino
  - genomecomb
  - scsorter
  - sctype
  - scywalker
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cell-type-annotation-visualization

## Summary

Assigns cell types to emptydrops-approved single-nuclei using marker gene sets via scsorter or sctype within scywalker, generates per-cell-type pseudobulk counts, and projects the assigned cell types onto a UMAP embedding for the interactive report panel.

## When to use

Reference for annotating scywalker long-read single-nuclei per-cell gene counts with cell types from a markerset and visualizing the assignments on UMAP in the QC report.

## When NOT to use

- No cell-specific markerset is provided: scsorter will not be used to determine cell types and pseudobulk files are not made
- Analysis is limited to a chromosome-subset test dataset (e.g., chromosome 17 only) where marker genes cannot properly determine cell types — scywalker's own test data notes celltyping can at least run but results are not meaningful
- Input lacks per-cell counts for emptydrops-approved cells (e.g., only raw counts over all detected 'cells' are available), so annotation and the UMAP panel cannot be restricted to real cells

## Inputs

- Per-cell gene counts (emptydrops-approved filtered set) from the scywalker calling module (sc_gene_counts_filtered)
- Cell-type markerset file (e.g., -cellmarkerfile markers_chr17.tsv) mapping genes to cell types
- Tissue specification (for sctype internal database lookup)
- is_cell flags from emptydrops distinguishing real cells from detected 'cells'

## Outputs

- Per-cell cell-type group assignments (group and group_filtered columns)
- Pseudobulk per-cell-type counts per sample
- Cross-sample pseudobulk comparison count files
- UMAP embedding over the per-cell count matrix with assigned cell types, shown in the report.html cell-type identification panel

## How to apply

Run scywalker with -cellmarkerfile pointing to a cell-type-specific markerset (e.g., markers_chr17.tsv in the test dataset); if a markerset is provided, scsorter and/or sctype are used to determine cell types, with sctype optionally using its internal database for a given tissue. Annotation is applied only to the emptydrops-approved cells from the sc_gene_counts_filtered per-cell count matrix (the _raw file covers all detected 'cells'). Produce per-cell-type pseudobulk count files per sample plus count files for cross-sample pseudobulk comparison. Store group assignments with a group_filtered variant that removes uncertain assignments for tools that support this. For visualization, project the assigned cell types onto a UMAP embedding computed over the per-cell count matrix to build the cell-type identification panel of the interactive HTML report.

## Related tools

- **scywalker** (Orchestrates cell-type assignment from markersets, pseudobulk generation, and the UMAP panel in the interactive HTML report) — https://github.com/derijkp/scywalker
- **scsorter** (Determines cell types and makes pseudobulk files from the markerset; skipped if no markerset is given)
- **sctype** (Alternative cell-typing tool that can use its internal database with a given tissue)
- **genomecomb** (Framework in which scywalker (and hence celltyping/reporting) is implemented; provided in full within the scywalker distribution) — https://github.com/derijkp/genomecomb

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- report.html exists, parses as HTML, and contains the UMAP cell-type identification panel
- group_filtered column contains no uncertain assignments for the cell-typing tools that support this filtering
- Per-cell-type pseudobulk counts per sample and cross-sample comparison count files are present alongside the per-cell group assignments
- Only emptydrops-approved cells (is_cell = 1) carry cell-type assignments in the filtered count files
- Pseudobulk per-cell-type totals are consistent with the summed per-cell counts over the assigned groups

## Limitations

- Cell typing requires a cell-specific markerset; without one, scsorter is not used and no pseudobulk files are generated
- On limited test data (chromosome 17 only), the included markers are not good markers and celltyping merely runs as a smoke test
- Cell-type identification for novel isoforms depends on the quality of assignments: sctype falls back to its internal database for the given tissue, which may not fit all species/conditions
- The isoquant-based code may hang or crash on organelle data, so organelles must be specified at refdir creation before annotation-downstream steps

## Evidence

- [intro] If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts for each sample: "If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts"
- [methods] scsorter celltyping skipped without markerset: "If not given, scsorter will not
    be used to determine celltypes and making pseudobulk files"
- [methods] sctype internal database with tissue: "however sctype will use its internal database 
    with the given tissue"
- [results] group_filtered removes uncertains: "group_filtered : assigned group/cell type filtering put uncertains (foor tools that support this)"
- [abstract] Report includes cell type identification UMAP: "Scywalker includes data quality control, cell type identification, and an interactive report."
- [readme] Test dataset markers are not real markers: "The "marker" genes in the included markers_chr17.tsv are not good
# markers, they are just made to allow celltyping to at least run"
