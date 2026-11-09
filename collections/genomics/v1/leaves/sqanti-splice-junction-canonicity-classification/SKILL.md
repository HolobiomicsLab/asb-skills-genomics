---
name: sqanti-splice-junction-canonicity-classification
description: Use when a filtered long-read isoform catalog (e.g., IsoPops 4+ exon
  / no-truncation database) must be independently validated by short-read RNA-seq
  junction support and each junction's donor-acceptor motif labeled canonical vs non-canonical.
when_to_use_negative:
- The goal is de novo transcript discovery rather than validating an existing long-read
  isoform catalog with short reads.
- Input short-read data are single-cell libraries; the article notes 3' bias means
  this integration works best for isoforms that differ at their 3' ends, so junction-level
  validation across internal exons is inappropriate.
- No independent short-read RNA-seq data from a matched tissue (retina/brain) are
  available to provide junction support.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: IsoPops
  role: R package used to compile, filter, and export the isoform catalog annotation
    as GFF (write_gff_data) that defines the junction set to validate
  repo: github.com/kellycochran/IsoPops
- name: STAR
  role: Builds a genome index from the isoform GFF (--sjdbGTFfile) and aligns short
    reads in two-pass mode to count junction-spanning reads
- name: SQANTI
  role: Classifies each aligned junction by splice-site motif as canonical vs non-canonical
- name: R
  role: Companion language for running IsoPops filtering and GFF export steps
  repo: github.com/kellycochran/IsoPops
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:c9b77e3fe886b4a95a7038937451913c6944bcf918a1ec99c59847b98401fe7b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoPops
  - R
  - SQANTI
  - STAR
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# sqanti-splice-junction-canonicity-classification

## Summary

A validation-and-classification skill for long-read isoform catalogs: splice junctions predicted from lrCaptureSeq isoforms (exported via IsoPops GFF) are cross-checked against independent short-read RNA-seq alignments (STAR) and labeled canonical vs non-canonical by splice-site motif using SQANTI. In the source paper this showed 98.1% short-read support (n = 79,020 junctions) and ~98.9% canonical motif fraction.

## When to use

Use when a filtered long-read isoform catalog (e.g., IsoPops 4+ exon / no-truncation database) must be independently validated by short-read RNA-seq junction support and each junction's donor-acceptor motif labeled canonical vs non-canonical.

## When NOT to use

- The goal is de novo transcript discovery rather than validating an existing long-read isoform catalog with short reads.
- Input short-read data are single-cell libraries; the article notes 3' bias means this integration works best for isoforms that differ at their 3' ends, so junction-level validation across internal exons is inappropriate.
- No independent short-read RNA-seq data from a matched tissue (retina/brain) are available to provide junction support.

## Inputs

- Filtered isoform database from IsoPops (e.g., DB_filter_4ex_notrunc, 4+ exon / no-truncation)
- GFF splice-junction annotation exported via IsoPops write_gff_data
- Short-read RNA-seq FASTQ files from retina and brain (GEO GSE101986 and GSE79416)
- Reference genome (for STAR index construction)

## Outputs

- junction_classification.tsv with per-junction coordinates, short-read support status, and canonical/non-canonical motif label
- Supported-junction fraction statistic (expected range 0.96-0.99)
- Canonical-junction fraction statistic (expected ~0.989)

## How to apply

First export the filtered isoform database as GFF with IsoPops (write_gff_data after filter_db/filter_truncations, e.g., exon_min_count = 4, abund_cutoff = 1). Build a STAR genome index supplying this GFF via --sjdbGTFfile so alignments are guided to the long-read-defined donor-acceptor pairs. Align short-read RNA-seq FASTQ data (in the paper, GEO accessions GSE101986 and GSE79416 from retina and brain) in STAR two-pass mode and tally reads spanning each predicted junction. Compute the supported-junction fraction (supported / total), expecting ~0.96-0.99 (paper: 0.981 across 79,020 junctions). Then run SQANTI on the alignment to classify each junction by splice-site motif, expecting ~98.9% canonical, and write per-junction records (coordinates, support, canonical/non-canonical label) to junction_classification.tsv.

## Related tools

- **IsoPops** (R package used to compile, filter, and export the isoform catalog annotation as GFF (write_gff_data) that defines the junction set to validate) — github.com/kellycochran/IsoPops
- **STAR** (Builds a genome index from the isoform GFF (--sjdbGTFfile) and aligns short reads in two-pass mode to count junction-spanning reads)
- **SQANTI** (Classifies each aligned junction by splice-site motif as canonical vs non-canonical)
- **R** (Companion language for running IsoPops filtering and GFF export steps) — github.com/kellycochran/IsoPops

## Examples

```
STAR --runMode genomeGenerate --genomeDir idx --genomeFastaFiles genome.fa --sjdbGTFfile isoforms.gff --sjdbOverhang 100 && STAR --genomeDir idx --readFilesIn reads_1.fq reads_2.fq --twopassMode Basic
```

## Evaluation signals

- Supported-junction fraction (short-read-supported / total isoform junctions) falls in [0.96, 0.99]; the paper reports 0.981 across n = 79,020 junctions.
- Canonical-junction fraction from SQANTI is approximately 0.989.
- junction_classification.tsv contains one row per isoform-predicted junction with coordinates, support flag, and canonical/non-canonical label.
- Total junction count matches the number of donor-acceptor pairs in the exported GFF (paper scale: ~79,020 after 4+ exon / no-truncation filtering).
- STAR index was built with --sjdbGTFfile pointing to the long-read GFF, so novel long-read-defined junctions are alignable.

## Limitations

- Short-read RNA-seq read lengths (<200 bp) cannot resolve full-length sequences of multi-kilobase transcripts, so this skill validates junctions only, not full-length isoform structures.
- Size selection in the lrCaptureSeq library prep means some isoforms smaller than 4.5 kb may have evaded detection, biasing the junction set.
- It cannot be excluded that certain transcripts (and thus junctions) are missing from the catalogs; absence of a junction cannot be excluded.

## Evidence

- [discussion] lrCaptureSeq integration with short reads for validation and expression mapping: "We show that lrCaptureSeq data can be integrated with existing short-read RNA-seq datasets, including single-cell data, to reveal the time and place of isoform expression."
- [methods] IsoPops raw database compilation entry point: "rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename)"
- [methods] IsoPops filtering thresholds used before junction export: "DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)"
- [readme] IsoPops purpose as R package: "IsoPops
R package for analysis of isoform-diverse transcript datasets"
