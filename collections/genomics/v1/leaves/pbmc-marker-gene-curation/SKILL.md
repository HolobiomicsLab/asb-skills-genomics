---
name: pbmc-marker-gene-curation
description: Use when processing long-read single-cell (10x) FASTQ data, e.g. PacBio
  or Oxford Nanopore PBMC runs, and you need marker-based cell-type assignments plus
  pseudobulk per-cell-type counts from scywalker.
when_to_use_negative:
- Input is already a cell-by-gene or cell-by-isoform count matrix from another pipeline
  — scywalker's typing runs end-to-end from FASTQ within its own workflow
- No cell-type marker file is available and sctype's internal tissue database does
  not cover the sample's tissue — scsorter/sctype celltyping is skipped and only counts
  are produced
- Limited test data (e.g. chromosome-17-restricted scywalker_test dataset) — the included
  markers_chr17.tsv are explicitly 'not good markers, they are just made to allow
  celltyping to at least run'
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0621
tools:
- name: scywalker
  role: End-to-end pipeline run with -cellmarkerfile, -sc_expectedcells and -preset
    pacbio to perform emptydrops filtering and scsorter/sctype cell typing
  repo: https://github.com/derijkp/scywalker
- name: scsorter
  role: Determines cell types and makes pseudobulk files from the provided markerset;
    group_filtered column removes uncertain assignments
- name: sctype
  role: Alternative cell-typing backend that uses its internal database with the given
    tissue when marker-file-based typing is not used
- name: genomecomb
  role: Implements scywalker; provides refdb_hg38 reference directories and cg/sw
    tooling for querying tsv count files
  repo: https://github.com/derijkp/genomecomb/
- name: minimap2
  role: Aligner whose PacBio index must be generated via scywalker_makerefdir '-pacbioindex
    1' when analyzing PacBio data
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:244cfb06d3ecc593095d8491b9c3c0225adabfb2a84b3297d3d85d64a5776979
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# pbmc-marker-gene-curation

## Summary

Curate a PBMC cell-type marker file (with 'marker' and 'celltype' columns) and use it with scywalker's cell filtering and typing module (scsorter, optionally sctype) to assign canonical PBMC populations — T cells, B cells, monocytes, NK cells — from long-read single-cell data such as the PacBio PBMC dataset shown in Supplementary Fig. S16.

## When to use

Use when processing long-read single-cell (10x) FASTQ data, e.g. PacBio or Oxford Nanopore PBMC runs, and you need marker-based cell-type assignments plus pseudobulk per-cell-type counts from scywalker.

## When NOT to use

- Input is already a cell-by-gene or cell-by-isoform count matrix from another pipeline — scywalker's typing runs end-to-end from FASTQ within its own workflow
- No cell-type marker file is available and sctype's internal tissue database does not cover the sample's tissue — scsorter/sctype celltyping is skipped and only counts are produced
- Limited test data (e.g. chromosome-17-restricted scywalker_test dataset) — the included markers_chr17.tsv are explicitly 'not good markers, they are just made to allow celltyping to at least run'

## Inputs

- PBMC single-cell FASTQ files (PacBio or Oxford Nanopore 10x full-length cDNA)
- Reference genome fasta and gtf transcript file (e.g. hg38) or a genomecomb refdb_hg38 reference directory with matching minimap2 indexes
- Cell marker file (tsv) with 'marker' and 'celltype' columns, plus tissue when using sctype's internal database
- Expected number of cells (value for -sc_expectedcells)
- 10x barcode whitelist (e.g. whitelists/737K-august-2016.txt.gz shipped with scywalker)

## Outputs

- Cell count files with emptydrops is_cell flag (is_cell 1 = real cell, 0 = empty droplet)
- Filtered count files (_filtered, emptydrops-approved cells only) alongside _raw counts for all detected cells
- Per-cell cell-type assignment table from scsorter (group_filtered column with uncertain calls removed)
- Pseudobulk (per cell-type) counts per sample and cross-sample comparison count files
- UMAP colored by assigned cell type (as rendered for Supplementary Fig. S16)

## How to apply

Build a reference directory with scywalker_makerefdir from hg38 fasta and gtf, adding '-pacbioindex 1' for PacBio data so the minimap2 PacBio index is generated. Organize FASTQs into a project/sample directory (fastq subdirectory per sample, or a '-samplesheet' tsv with 'sample' and 'seqfiles' fields). Run the single scywalker command with '-preset pacbio', the '-refdir', '-sc_expectedcells' (required for emptydrops cell filtering), and '-cellmarkerfile' pointing at a markers file with 'marker' and 'celltype' columns (plus tissue if sctype will use its internal database for a given tissue). Extract scsorter cell-type assignments per cell from the group_filtered column, which removes uncertain calls, and use the is_cell flag from emptydrops to distinguish real cells from empty droplets. Validate that recovered populations match the canonical PBMC cell types (T cells, B cells, monocytes, NK cells) reported in Supplementary Fig. S16 and that filtered cell counts are consistent with the expected cell number supplied.

## Related tools

- **scywalker** (End-to-end pipeline run with -cellmarkerfile, -sc_expectedcells and -preset pacbio to perform emptydrops filtering and scsorter/sctype cell typing) — https://github.com/derijkp/scywalker
- **scsorter** (Determines cell types and makes pseudobulk files from the provided markerset; group_filtered column removes uncertain assignments)
- **sctype** (Alternative cell-typing backend that uses its internal database with the given tissue when marker-file-based typing is not used)
- **genomecomb** (Implements scywalker; provides refdb_hg38 reference directories and cg/sw tooling for querying tsv count files) — https://github.com/derijkp/genomecomb/
- **minimap2** (Aligner whose PacBio index must be generated via scywalker_makerefdir '-pacbioindex 1' when analyzing PacBio data)

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -preset pacbio -refdir refdb_hg38 -sc_expectedcells 183 -cellmarkerfile pbmc_markers.tsv -threads 6 pacbio_pbmc_project
```

## Evaluation signals

- Recovered cell-type populations match the canonical PBMC types (T cells, B cells, monocytes, NK cells) shown in Supplementary Fig. S16
- Number of cells with is_cell = 1 in the cell count files is consistent with the -sc_expectedcells value supplied
- _filtered count files contain only emptydrops-approved cells while _raw files cover all detected cells
- group_filtered column contains assigned cell types with uncertain calls removed, distinct from raw group assignments
- Pseudobulk per-cell-type count files are produced per sample and allow cross-sample comparison

## Limitations

- Organelle chromosomes can crash the isoquant-based code; organelles must be specified in scywalker_makerefdir so the organelle-specific algorithm is used
- On limited test data, cell typing cannot be properly determined — markers must be real, biologically meaningful markers for valid PBMC typing
- If no cellmarkerfile is given, scsorter will not be used to determine cell types or make pseudobulk files; sctype typing depends on tissue coverage of its internal database
- -sc_expectedcells is required for emptydrops cell filtering, so a reasonable prior expectation of cell numbers is needed

## Evidence

- [methods] scsorter will not be used to determine celltypes and making pseudobulk files: "If not given, scsorter will not
    be used to determine celltypes and making pseudobulk files"
- [methods] sctype uses internal database with given tissue: "however sctype will use its internal database 
    with the given tissue"
- [results] group_filtered removes uncertain assignments: "group_filtered : assigned group/cell type filtering put uncertains (foor tools that support this)"
- [results] is_cell flag marks real cells by emptydrops: "is_cell : 1 if emptydrops categorized the cell as real, 0 if not"
- [readme] PacBio index added via -pacbioindex 1: "the appropriate index can be added by giving the 
option '-pacbioindex 1' to scywalker_makerefdir"
- [abstract] Scywalker includes cell type identification: "Scywalker includes data quality control, cell type identification, and an interactive report."
