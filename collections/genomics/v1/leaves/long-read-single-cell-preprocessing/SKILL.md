---
name: long-read-single-cell-preprocessing
description: Use when input is 10x single-cell long-read FASTQ data (Oxford Nanopore
  or PacBio) and the goal is cell barcode demultiplexing, alignment, isoform/gene
  quantification, and emptydrops cell filtering without matched short-read data.
when_to_use_negative:
- Input is already aligned BAMs or a processed count/feature matrix — scywalker starts
  from sequenced fragments in FASTQ format
- Short-read single-cell data that standard short-read pipelines handle natively (scywalker
  is designed for long-read data without the need for matching short reads)
- Reference genomes containing organelle sequences when they cannot be explicitly
  declared via -organelles, since the IsoQuant-based code can hang or crash on very
  different organelle data
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scywalker
  role: 'End-to-end workflow driver: cell barcode demultiplexing, alignment, isoform/gene
    reconstruction and quantification, cell typing and pseudobulk counting from long-read
    single-cell FASTQ in a single command'
  repo: https://github.com/derijkp/scywalker
- name: scywalker_makerefdir
  role: Builds the scywalker reference directory (indexes and supporting files) from
    a genome FASTA and GTF, with -pacbioindex 1 for PacBio data and -organelles for
    organelle chromosomes
  repo: https://github.com/derijkp/scywalker
- name: minimap2
  role: Default read aligner used via the minimap2_splice preset; ONT and PacBio indexes
    are prebuilt in the reference directory
- name: IsoQuant
  role: Underlying engine used by scywalker to reconstruct and quantify isoforms and
    genes from aligned long reads
  repo: https://github.com/ablab/IsoQuant
- name: genomecomb
  role: Framework within which scywalker is implemented; also provides reference directories
    (refdb_hg38) and cg/sw utilities for querying result tsv files
  repo: https://github.com/derijkp/genomecomb
- name: scsorter
  role: Optional cell-type assignment from a provided marker file; also used to make
    pseudobulk files
- name: sctype
  role: Alternative cell-typing tool that uses its internal database with the given
    tissue when markers are not supplied
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:ef95657813a8e6a01b51eb21c5b9e3fb67a6049f92d6a59772069998e403ec55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - IsoQuant
  - genomecomb
  - minimap2
  - scsorter
  - sctype
  - scywalker
  - scywalker_makerefdir
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# long-read-single-cell-preprocessing

## Summary

End-to-end preprocessing of long-read single-cell (10x) transcriptome data — Oxford Nanopore or PacBio — using scywalker: from FASTQ through cell barcode assignment, minimap2_splice alignment, and IsoQuant-based isoform/gene reconstruction and quantification, producing bulk and per-cell count files.

## When to use

Use when input is 10x single-cell long-read FASTQ data (Oxford Nanopore or PacBio) and the goal is cell barcode demultiplexing, alignment, isoform/gene quantification, and emptydrops cell filtering without matched short-read data.

## When NOT to use

- Input is already aligned BAMs or a processed count/feature matrix — scywalker starts from sequenced fragments in FASTQ format
- Short-read single-cell data that standard short-read pipelines handle natively (scywalker is designed for long-read data without the need for matching short reads)
- Reference genomes containing organelle sequences when they cannot be explicitly declared via -organelles, since the IsoQuant-based code can hang or crash on very different organelle data

## Inputs

- Long-read single-cell FASTQ files from 10x libraries (Oxford Nanopore or PacBio), placed in a fastq subdirectory of a sample directory or project directory, or listed in a samplesheet tsv with 'sample' and 'seqfiles' columns
- Reference genome FASTA and transcript GTF (or a genomecomb gene tsv / pre-built genomecomb reference directory such as refdb_hg38)
- 10x barcode whitelist (e.g. whitelists/737K-august-2016.txt.gz included with scywalker)
- Expected number of cells per sample for emptydrops filtering (-sc_expectedcells)
- Optional cell marker file with 'marker' and 'celltype' columns (plus tissue if applicable) for cell typing

## Outputs

- Bulk and per-cell gene count matrices, with _filtered versions containing only emptydrops-approved cells (is_cell = 1) and _raw versions for all detected 'cells'
- Isoform counts distinguishing weighed, unique, and strict counts (strict requiring >= 90% transcript coverage)
- Per-sample cell barcode assignments and demultiplexed counts
- Per-cell-type pseudobulk counts and cross-sample pseudobulk comparison count files (when a cellmarkerfile is given)
- Interactive report including data quality control

## How to apply

Install the scywalker binary distribution (e.g. v0.112.0, self-contained, includes genomecomb) and build a reference directory with scywalker_makerefdir from a genome FASTA and GTF, specifying organelle chromosomes explicitly so the IsoQuant-based code does not hang or crash on organelle data; add a PacBio minimap2 index with '-pacbioindex 1' when processing PacBio (rather than ONT) reads. Organize FASTQs into a sample directory (fastq subdirectory) or project directory, optionally built from a -samplesheet tsv with 'sample' and 'seqfiles' fields. Run the single scywalker command with the project/sample directory, -refdir, and -sc_expectedcells (required for emptydrops cell filtering), setting distribution options such as -d (cores/cluster) and -dmaxmem, keeping -threads low (~4-8). scywalker then performs cell barcode demultiplexing, minimap2_splice alignment, IsoQuant-based isoform and gene reconstruction, and writes bulk and per-cell counts; if a -cellmarkerfile with marker/celltype columns is provided, it also assigns cell types via scsorter (or sctype with an internal tissue database) and produces per-cell-type pseudobulk counts. Correct application is judged by checking that _filtered count files contain only emptydrops-approved cells (is_cell = 1) and that multimapping reads are weighed 1/N across N genes in gene counts.

## Related tools

- **scywalker** (End-to-end workflow driver: cell barcode demultiplexing, alignment, isoform/gene reconstruction and quantification, cell typing and pseudobulk counting from long-read single-cell FASTQ in a single command) — https://github.com/derijkp/scywalker
- **scywalker_makerefdir** (Builds the scywalker reference directory (indexes and supporting files) from a genome FASTA and GTF, with -pacbioindex 1 for PacBio data and -organelles for organelle chromosomes) — https://github.com/derijkp/scywalker
- **minimap2** (Default read aligner used via the minimap2_splice preset; ONT and PacBio indexes are prebuilt in the reference directory)
- **IsoQuant** (Underlying engine used by scywalker to reconstruct and quantify isoforms and genes from aligned long reads) — https://github.com/ablab/IsoQuant
- **genomecomb** (Framework within which scywalker is implemented; also provides reference directories (refdb_hg38) and cg/sw utilities for querying result tsv files) — https://github.com/derijkp/genomecomb
- **scsorter** (Optional cell-type assignment from a provided marker file; also used to make pseudobulk files)
- **sctype** (Alternative cell-typing tool that uses its internal database with the given tissue when markers are not supplied)

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- The _filtered gene count files contain only emptydrops-approved cells, while _raw files provide counts for all detected 'cells'; is_cell = 1 marks cells categorized as real by emptydrops
- Gene multimapping reads are weighed such that a read mapping to N genes gives each gene a 1/N count
- Isoform counts separate counts_weighed, counts_unique, and counts_strict, with strict counting only unique reads covering >= 90% of the transcript
- Recovered cell counts per sample are consistent with the -sc_expectedcells value supplied for emptydrops filtering
- Per-cell total counts and per-cell gene counts correlate highly with matched short-read quantification on benchmark data (e.g. R = 0.986 and R = 0.886 on PacBio HG002)

## Limitations

- The IsoQuant-based code often hangs or crashes on very different organelle data, so organelle chromosomes must be specified with -organelles in scywalker_makerefdir
- By default scywalker_makerefdir only creates an ONT minimap2 index; a PacBio index must be added explicitly with '-pacbioindex 1'
- Default chromosome grouping assumes alt/unplaced chromosomes contain '_' in their names; use -groupchromosomes with regular expressions for genomes where this pattern does not hold
- Cell typing requires a markerset; on limited test data (e.g. chromosome-17-only) proper cell typing is not possible and marker files are placeholders
- Cell typing can only recover populations as good as the supplied marker genes; group_filtered removes uncertain assignments only for tools that support this

## Evidence

- [abstract] Scywalker streamlines the entire analysis process, from sequenced fragments in FASTQ format to demultiplexed pseudobulk isoform counts, into a single command suitable for execution on either server: "Scywalker streamlines the entire analysis process, from sequenced fragments in FASTQ format to demultiplexed pseudobulk isoform counts, into a single command suitable for execution on either server"
- [readme] organelles must be declared: "If there are organelles in the genome sequence, it is important to specify them so the organelle specific algorithm can be used: The isoquant based code often hangs or crashes on the very different"
- [results] filtered vs raw counts: "In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells"."
- [methods]  PacBio index option: "the appropriate index can be added by giving the option '-pacbioindex 1' to scywalker_makerefdir"
- [results] strict isoform counts: "counts_strict: only unique reads that cover >= 90% of the transcript"
