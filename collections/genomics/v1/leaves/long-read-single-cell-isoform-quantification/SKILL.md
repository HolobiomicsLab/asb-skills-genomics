---
name: long-read-single-cell-isoform-quantification
description: Use when starting from long-read (10x ONT or PacBio) single-cell FASTQs
  and you need cell barcode demultiplexing, isoform reconstruction/quantification,
  and per-cell or pseudobulk isoform count matrices without matching short-read data.
when_to_use_negative:
- Only matched short-read quantification is needed and a Cell Ranger short-read pipeline
  output already exists
- Input is already an isoform/gene count matrix — the FASTQ-to-count stage of scywalker
  does not apply
- Reads are not full-length single-cell long-read cDNA (e.g. bulk long-read data without
  cell barcodes)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scywalker
  role: 'Single-command end-to-end pipeline: cell barcode finding/assignment, alignment,
    isoform reconstruction and quantification, cell typing, and pseudobulk count generation'
  repo: https://github.com/derijkp/scywalker
- name: genomecomb
  role: Framework in which scywalker is implemented; the cg/sw executables provide
    extra tools for querying tsv count files and downstream analysis
  repo: https://github.com/derijkp/genomecomb
- name: IsoQuant
  role: Underlying engine for isoform and gene reconstruction and quantification within
    scywalker
  repo: https://github.com/ablab/IsoQuant
- name: minimap2
  role: Default read aligner used with the splice preset (minimap2_splice); reference
    directory needs the matching minimap2 index
- name: scsorter
  role: Assigns cell types from the -cellmarkerfile markersets, triggering pseudobulk
    (per cell-type) count generation
- name: sctype
  role: Alternative cell-typing tool that uses its internal database with the given
    tissue
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:ffb3a170d9bd1f4d6c9679524f56d9417aa7689adaf61eb7e93929bbea526518
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - DRIMSeq
  - IsoQuant
  - genomecomb
  - minimap2
  - scsorter
  - sctype
  - scywalker
  - stageR
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# long-read-single-cell-isoform-quantification

## Summary

End-to-end scywalker workflow that converts Oxford Nanopore (or PacBio) full-length single-cell cDNA FASTQs into demultiplexed bulk and per-cell isoform and gene counts in a single command, including isoform reconstruction via IsoQuant and pseudobulk per-cell-type counts when markersets are supplied.

## When to use

Use when starting from long-read (10x ONT or PacBio) single-cell FASTQs and you need cell barcode demultiplexing, isoform reconstruction/quantification, and per-cell or pseudobulk isoform count matrices without matching short-read data.

## When NOT to use

- Only matched short-read quantification is needed and a Cell Ranger short-read pipeline output already exists
- Input is already an isoform/gene count matrix — the FASTQ-to-count stage of scywalker does not apply
- Reads are not full-length single-cell long-read cDNA (e.g. bulk long-read data without cell barcodes)

## Inputs

- Oxford Nanopore long-read single-cell FASTQ files (10x chemistry) in a sample/project fastq subdirectory
- Reference genome fasta and transcript gtf (or genomecomb gene tsv) for scywalker_makerefdir
- Cell-specific markersets file passed via -cellmarkerfile
- 10x barcode whitelist (e.g. whitelists/737K-august-2016.txt.gz)
- Parameters such as -sc_expectedcells and resource settings (-d cores, -dmaxmem)

## Outputs

- Bulk and per-cell isoform and gene count matrices
- Pseudobulk (per cell-type) isoform counts per sample for cross-sample comparison
- sc_gene_counts_filtered (emptydrops-approved cells) and sc_gene_counts_raw (all detected cells) files
- Weighed, unique, and strict isoform count variants (strict requires >= 90% transcript coverage)
- Cell type assignments and quality-control interactive report

## How to apply

First build a reference directory with scywalker_makerefdir from a genome fasta and gtf, specifying organelle chromosomes (e.g. -organelles 'chrM chrPt') so the organelle-specific code path is used, and add a minimap2 index (-pacbioindex 1 for PacBio). Organize data as a sample directory (with a fastq subdirectory) or a project directory (optionally created from a -samplesheet), then run the single scywalker command with the ONT default minimap2_splice aligner and distribution options such as -d for cores and -dmaxmem for memory. Provide -sc_expectedcells for cell barcode calling and a -cellmarkerfile with cell-specific markersets so scsorter assigns cell types and pseudobulk (per cell-type) isoform counts are generated for each sample. Filter to real cells via emptydrops: use the _filtered gene count files (emptydrops-approved cells only) for downstream analyses, while the _raw files retain all detected cells. Isoform counts are emitted as weighed, unique, and strict variants, with strict counts limited to unique reads covering >= 90% of the transcript.

## Related tools

- **scywalker** (Single-command end-to-end pipeline: cell barcode finding/assignment, alignment, isoform reconstruction and quantification, cell typing, and pseudobulk count generation) — https://github.com/derijkp/scywalker
- **genomecomb** (Framework in which scywalker is implemented; the cg/sw executables provide extra tools for querying tsv count files and downstream analysis) — https://github.com/derijkp/genomecomb
- **IsoQuant** (Underlying engine for isoform and gene reconstruction and quantification within scywalker) — https://github.com/ablab/IsoQuant
- **minimap2** (Default read aligner used with the splice preset (minimap2_splice); reference directory needs the matching minimap2 index)
- **scsorter** (Assigns cell types from the -cellmarkerfile markersets, triggering pseudobulk (per cell-type) count generation)
- **sctype** (Alternative cell-typing tool that uses its internal database with the given tissue)

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- Correlation of scywalker per-cell and gene-per-cell UMI counts against short-read (Cell Ranger) quantification — reported cell-level R = 0.942 and gene-per-cell R = 0.89 on the scmixology2 benchmark
- sc_gene_counts_filtered contains only emptydrops-approved cells (is_cell = 1) while _raw contains all detected cells
- Strict isoform counts include only unique reads covering >= 90% of the transcript
- Pseudobulk count files are produced per sample only when a -cellmarkerfile is supplied so scsorter celltyping ran
- Multimapping gene reads are weighed so a read mapping to N genes contributes 1/N to each gene

## Limitations

- The isoquant-based code often hangs or crashes on very different organelle data, so organelle chromosomes must be specified in scywalker_makerefdir to use the organelle-specific algorithm
- The executable must stay inside the portable application directory to find its dependencies
- Default chromosome grouping assumes alt/unplaced chromosome names contain '_' (e.g. chr1_KI270706v1_random); use -groupchromosomes for genomes where this pattern fails
- Cell typing is not meaningful on limited test data — marker genes must be appropriate for the tissue

## Evidence

- [abstract] End-to-end analysis in one command: "Scywalker streamlines the entire analysis process, from sequenced fragments in FASTQ format to demultiplexed pseudobulk isoform counts, into a single command suitable for execution on either server"
- [intro] End-to-end pipeline scope: "Starting from fastqs, it will find and assign cellbarcodes, align reads, and reconstruct (based on [IsoQuant](https://github.com/ablab/IsoQuant)) and quantify isoforms and genes, producing both bulk"
- [readme] Pseudobulk generation requires markersets: "If cell specific markersets are provided, it will also assign cell-types and generate pseudobulk (per cell-type) counts for each sample, and make count files allowing comparison between samples for"
- [readme] Organelle handling caveat: "If there are organelles in the genome sequence, it is important to specify them so the organelle specific algorithm can be used: The isoquant based code often hangs or crashes on the very different"
- [results] Filtered vs raw count files: "In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells"."
- [results] Strict count threshold: "counts_strict: only unique reads that cover >= 90% of the transcript"
