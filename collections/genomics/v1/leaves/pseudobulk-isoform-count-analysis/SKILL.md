---
name: pseudobulk-isoform-count-analysis
description: Use when long-read (Oxford Nanopore or PacBio) 10x single-cell FASTQs
  must be converted into per-cell-type pseudobulk isoform counts, e.g. to compare
  neuronal vs glial groups or feed DRIMSeq/stageR DTU testing.
when_to_use_negative:
- No cell-specific markerset is available (scsorter will not run, so no cell types
  or pseudobulk files are generated) — only per-cell/bulk counts are produced
- Data are short-read single-cell rather than full-length long-read cDNA (scywalker
  targets 10x Oxford Nanopore and PacBio long-read data)
- Only empty-drop rejected barcodes/noise cells are of interest — the _filtered files
  exclude cells not approved by emptydrops
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scywalker
  role: 'End-to-end pipeline: demultiplexes barcodes, aligns, reconstructs and quantifies
    isoforms, assigns cell types and aggregates per-cell-type pseudobulk isoform counts'
  repo: https://github.com/derijkp/scywalker
- name: scsorter
  role: Cell typing from the supplied markerset (-cellmarkerfile); required for pseudobulk
    file generation
  repo: https://github.com/derijkp/scywalker
- name: sctype
  role: Alternative cell-typing method using its internal database for the given tissue
  repo: https://github.com/derijkp/scywalker
- name: IsoQuant
  role: Underlying isoform reconstruction and quantification engine for long-read
    alignment-based isoform calling
  repo: https://github.com/ablab/IsoQuant
- name: DRIMSeq
  role: Downstream Dirichlet-multinomial screening of the pseudobulk isoform counts
    for gene-level differential transcript usage (gDTU)
- name: stageR
  role: Hierarchical two-stage correction producing gene-level gFDR and per-transcript
    tFDR after DRIMSeq testing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:455b57258bc97c4b1268b2c154d88e52d31818bd81c107fc2e16ee62716e1b2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - DRIMSeq
  - IsoQuant
  - scsorter
  - sctype
  - scywalker
  - stageR
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pseudobulk-isoform-count-analysis

## Summary

Generate per-cell-type (pseudobulk) isoform and gene count matrices from long-read single-cell data with scywalker, by supplying cell-specific markersets so scsorter/sctype assign cell types, then aggregate per-cell isoform counts (weighed, unique, or strict) into per-group counts for cross-sample comparison and downstream DTU testing.

## When to use

Use when long-read (Oxford Nanopore or PacBio) 10x single-cell FASTQs must be converted into per-cell-type pseudobulk isoform counts, e.g. to compare neuronal vs glial groups or feed DRIMSeq/stageR DTU testing.

## When NOT to use

- No cell-specific markerset is available (scsorter will not run, so no cell types or pseudobulk files are generated) — only per-cell/bulk counts are produced
- Data are short-read single-cell rather than full-length long-read cDNA (scywalker targets 10x Oxford Nanopore and PacBio long-read data)
- Only empty-drop rejected barcodes/noise cells are of interest — the _filtered files exclude cells not approved by emptydrops

## Inputs

- FASTQ files of 10x (or PacBio with -pacbioindex 1) long-read single-cell data in a sample/project directory fastq/ subdirectory
- scywalker reference directory built by scywalker_makerefdir from a genome fasta and transcripts.gtf (or a genomecomb gene tsv)
- Cell-specific markerset file passed via -cellmarkerfile (e.g. markers_chr17.tsv)
- 10x barcode whitelist (e.g. whitelists/737K-august-2016.txt.gz)
- Optional samplesheet for multi-sample project setup

## Outputs

- Per-cell isoform and gene count matrices (_filtered and _raw variants)
- Pseudobulk (per cell-type) isoform/gene counts for each sample
- Cross-sample comparison count files for pseudobulk counts
- Cell-type/group assignments (group, group_filtered) with emptydrops is_cell flags
- Downstream dtu_significant_genes.tsv with gFDR and per-transcript tFDR when combined with DRIMSeq/stageR

## How to apply

Run scywalker on a sample or project directory containing a fastq/ subdirectory with a reference directory built by scywalker_makerefdir (plus minimap2 indexes, PacBio index via -pacbioindex 1 if needed). Supply a markerset via -cellmarkerfile so the celltyping module (scsorter, or sctype with its internal tissue database) assigns cell types; without this file scsorter is not run and pseudobulk files are not produced. Cells are filtered with emptydrops (is_cell flag), and the _filtered count files contain only emptydrops-approved cells while _raw contains all detected cells. Isoform counts distinguish weighed counts (multimapping reads get 1/N per gene), unique counts, and counts_strict (unique reads covering >= 90% of the transcript); use group_filtered to drop uncertain cell-type assignments. Use the resulting per-cell-type pseudobulk count tables and the cross-sample comparison count files as the count matrix and group assignments for two-group analyses such as DRIMSeq Dirichlet-multinomial gDTU screening followed by stageR transcript-level confirmation, retaining genes at FDR < 0.05 with per-transcript tFDR values.

## Related tools

- **scywalker** (End-to-end pipeline: demultiplexes barcodes, aligns, reconstructs and quantifies isoforms, assigns cell types and aggregates per-cell-type pseudobulk isoform counts) — https://github.com/derijkp/scywalker
- **scsorter** (Cell typing from the supplied markerset (-cellmarkerfile); required for pseudobulk file generation) — https://github.com/derijkp/scywalker
- **sctype** (Alternative cell-typing method using its internal database for the given tissue) — https://github.com/derijkp/scywalker
- **IsoQuant** (Underlying isoform reconstruction and quantification engine for long-read alignment-based isoform calling) — https://github.com/ablab/IsoQuant
- **DRIMSeq** (Downstream Dirichlet-multinomial screening of the pseudobulk isoform counts for gene-level differential transcript usage (gDTU))
- **stageR** (Hierarchical two-stage correction producing gene-level gFDR and per-transcript tFDR after DRIMSeq testing)

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- Pseudobulk files exist only when -cellmarkerfile was supplied; absence of cell-type assignments/pseudobulk outputs means scsorter never ran
- _filtered count files should contain only emptydrops-approved cells (is_cell = 1), while _raw files include all detected cells
- counts_strict values must be a subset of unique counts, restricted to reads covering >= 90% of the transcript
- Multimapping gene-level counts must be fractional and consistent with 1/N weighting per gene
- DTU results written to dtu_significant_genes.tsv should all pass FDR < 0.05 on both gFDR and tFDR, with scale comparable to the paper's 301 gDTU genes on brain samples

## Limitations

- The isoquant-based code often hangs or crashes on very different organelle data, so organelle chromosomes must be specified via -organelles in scywalker_makerefdir
- Cell-type assignment quality depends on marker quality — in the test dataset the marker genes are explicitly not good markers, only enabling celltyping to run
- counts_strict discards non-unique reads and reads with < 90% transcript coverage, so strict counts can severely undercount multimapping or partially covered isoforms
- group_filtered is only available for celltyping tools that support uncertainty filtering

## Evidence

- [abstract] Scywalker streamlines the entire analysis process, from sequenced fragments in FASTQ format to demultiplexed pseudobulk isoform counts, into a single command: "Scywalker streamlines the entire analysis process, from sequenced fragments in FASTQ format to demultiplexed pseudobulk isoform counts, into a single command"
- [methods] If not given, scsorter will not be used to determine celltypes and making pseudobulk files: "If not given, scsorter will not
    be used to determine celltypes and making pseudobulk files"
- [results] counts_strict: only unique reads that cover >= 90% of the transcript: "counts_strict: only unique reads that cover >= 90% of the transcript"
- [results] In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells".: "In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells"."
- [readme] If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts for each sample: "If cell specific markersets are provided, it will also assign cell-types
and generate pseudobulk (per cell-type) counts for each sample"
