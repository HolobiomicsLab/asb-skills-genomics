---
name: emptydrops-cell-filtering
description: Use when running scywalker on 10x-style droplet long-read single-cell
  data (ONT or PacBio) and you need to separate true cells from empty droplets in
  the gene count output.
when_to_use_negative:
- Input is bulk long-read cDNA or non-droplet single-cell data with no cell barcodes,
  where empty-droplet statistics are meaningless.
- You need counts for all detected barcodes (including likely empty droplets) — use
  the _raw count files instead of filtering.
- The genome/reference has organelle chromosomes unspecified in scywalker_makerefdir,
  which breaks the scywalker pipeline upstream of filtering (isoquant-based code hangs
  on organelle data).
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scywalker
  role: End-to-end workflow that embeds emptydrops cell filtering after barcode finding
    and gene quantification; -sc_expectedcells triggers the filtering
  repo: https://github.com/derijkp/scywalker
- name: genomecomb
  role: Framework in which scywalker (and its filtering/count-file handling) is implemented
  repo: https://github.com/derijkp/genomecomb
- name: scsorter
  role: Downstream cell typing applied to emptydrops-approved cells; group_filtered
    removes uncertain assignments
  repo: https://github.com/derijkp/scywalker
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:37113444404e06d692c4f77618f37a3d606b93897178b06353a23b10564a7445
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - genomecomb
  - scsorter
  - scywalker
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# emptydrops-cell-filtering

## Summary

Distinguish real cells from empty droplets in scywalker long-read single-cell (10x ONT or PacBio) analyses using the emptydrops algorithm, producing is_cell flags and _filtered count matrices restricted to approved cells.

## When to use

Use when running scywalker on 10x-style droplet long-read single-cell data (ONT or PacBio) and you need to separate true cells from empty droplets in the gene count output.

## When NOT to use

- Input is bulk long-read cDNA or non-droplet single-cell data with no cell barcodes, where empty-droplet statistics are meaningless.
- You need counts for all detected barcodes (including likely empty droplets) — use the _raw count files instead of filtering.
- The genome/reference has organelle chromosomes unspecified in scywalker_makerefdir, which breaks the scywalker pipeline upstream of filtering (isoquant-based code hangs on organelle data).

## Inputs

- scywalker gene-per-cell count files (_raw and _filtered) from a 10x droplet long-read project/sample directory
- Expected number of cells supplied via -sc_expectedcells (e.g. 183 for scmixology2)
- FASTQ-derived cell barcode assignments produced by the scywalker single-command workflow

## Outputs

- is_cell flag per barcode (1 = real cell, 0 = empty droplet)
- emptydrops-filtered gene count files (sc_gene_counts_filtered) containing only approved cells
- group_filtered column of cell-type assignments with uncertain calls removed (when celltyping is enabled)

## How to apply

Provide the expected number of cells to the scywalker run with '-sc_expectedcells' (e.g. 183 for the scmixology2 dataset); this information is required for filtering cells using emptydrops, and scywalker embeds emptydrops in its end-to-end workflow after cell barcode finding and gene quantification. In the generated cell count files, check the 'is_cell' column (1 = emptydrops categorized the cell as real, 0 = not). Use the *_sc_gene_counts_filtered files, which contain only emptydrops-approved cells, rather than the _raw files that provide counts for all detected 'cells'. When downstream cell typing (scsorter/sctype) is used, extract assignments via the 'group_filtered' column, which removes uncertain calls.

## Related tools

- **scywalker** (End-to-end workflow that embeds emptydrops cell filtering after barcode finding and gene quantification; -sc_expectedcells triggers the filtering) — https://github.com/derijkp/scywalker
- **genomecomb** (Framework in which scywalker (and its filtering/count-file handling) is implemented) — https://github.com/derijkp/genomecomb
- **scsorter** (Downstream cell typing applied to emptydrops-approved cells; group_filtered removes uncertain assignments) — https://github.com/derijkp/scywalker

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -threads 6 test10x
```

## Evaluation signals

- The is_cell column is present in cell count files and equals 1 for the expected number of real cells (~183 in the scmixology2 benchmark).
- sc_gene_counts_filtered files contain exactly the emptydrops-approved cells, while sc_gene_counts_raw files contain counts for all detected 'cells'.
- Cell-level Pearson correlation against Cell Ranger short-read counts on filtered cells reaches ~0.942 for scywalker on scmixology2 (vs 0.734 for wf-single-cell, 0.65 for BLAZE-FLAMES).
- group_filtered contains no uncertain cell-type calls for tools that support uncertainty filtering.

## Limitations

- The filtering depends on a user-supplied expected cell number (-sc_expectedcells), which must be known a priori.
- Only real-cell information is retained in _filtered files; raw barcode-level signal from empty droplets is only available in _raw files.
- The isoquant-based quantification feeding the count files often hangs or crashes on very different organelle data, so organelles must be declared at reference-build time or filtering never runs.

## Evidence

- [methods] This information is required for filtering cells using emptydrops: "This information is required for filtering cells using emptydrops"
- [results] is_cell flag marks real cells: "is_cell : 1 if emptydrops categorized the cell as real, 0 if not"
- [results] Filtered count files contain only approved cells: "In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells"."
- [results] group_filtered removes uncertain assignments: "group_filtered : assigned group/cell type filtering put uncertains (foor tools that support this)"
- [readme] Expected cells option in README example run: "-sc_expectedcells 183 \
	-cellmarkerfile markers_chr17.tsv"
