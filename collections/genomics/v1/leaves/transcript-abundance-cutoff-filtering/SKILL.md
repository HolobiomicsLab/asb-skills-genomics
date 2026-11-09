---
name: transcript-abundance-cutoff-filtering
description: Use when a raw Iso-Seq/capture-seq isoform database compiled with IsoPops
  contains spurious or low-support isoforms and you need to enforce exon-count, truncation,
  and per-gene abundance thresholds to yield a final catalog.
when_to_use_negative:
- Input is raw PacBio subreads or uncompiled Iso-Seq cluster output that has not yet
  been assembled into an IsoPops database via compile_raw_db/process_db
- Goal is isoform discovery/detection rather than curation — filtering imposes the
  paper's abundance and exon-count thresholds and will discard rare transcripts that
  a discovery run should retain
- Input is short-read RNA-seq data, which cannot resolve full-length multi-kilobase
  transcript sequences needed for exon-boundary-based filtering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: IsoPops
  role: R package providing compile_raw_db, process_db, filter_db, filter_truncations,
    filter_top_gff, and write_gff_data used to compile, filter, and export the isoform
    catalog
  repo: github.com/kellycochran/IsoPops
- name: R
  role: runtime environment in which the IsoPops filtering functions and supporting
    tidyverse packages are executed
- name: dplyr
  role: data-manipulation dependency of IsoPops used in database filtering operations
- name: stringr
  role: string-processing dependency of IsoPops used in transcript/GFF handling
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:83d2b58c37d48f81dee40ac5711f8e90cf96f30c78db2510a19ae78d8bb91487
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoPops
  - R
  - dplyr
  - stringr
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# transcript-abundance-cutoff-filtering

## Summary

A three-step isoform filtering routine (exon-count/abundance floor, truncation removal, bottom-5% abundance trimming) implemented with the IsoPops R package to convert a pre-filter lrCaptureSeq Iso-Seq annotation into a curated isoform catalog (8287 pre-filter isoforms reduced to ~4116 across 30 targeted genes).

## When to use

Use when a raw Iso-Seq/capture-seq isoform database compiled with IsoPops contains spurious or low-support isoforms and you need to enforce exon-count, truncation, and per-gene abundance thresholds to yield a final catalog.

## When NOT to use

- Input is raw PacBio subreads or uncompiled Iso-Seq cluster output that has not yet been assembled into an IsoPops database via compile_raw_db/process_db
- Goal is isoform discovery/detection rather than curation — filtering imposes the paper's abundance and exon-count thresholds and will discard rare transcripts that a discovery run should retain
- Input is short-read RNA-seq data, which cannot resolve full-length multi-kilobase transcript sequences needed for exon-boundary-based filtering

## Inputs

- transcript FASTA file from PacBio Iso-Seq processing
- isoform abundance table
- GFF annotation file
- ORF FASTA file
- gene ID/name table (e.g. gene_IDs_and_names.tsv)

## Outputs

- filtered IsoPops Database retaining full-length, minimally abundant isoforms
- GFF3 file of the filtered isoform catalog (~4116 isoforms across 30 genes)

## How to apply

Load the pre-filter Iso-Seq annotation into an IsoPops Database by compiling transcript FASTA, abundance table, GFF, and ORF FASTA with compile_raw_db and processing against the gene ID/name table with process_db. Apply filter_db(exon_min_count = 4, abund_cutoff = 1) to drop isoforms with fewer than 4 non-contiguous genomic segments or abundance below 1. Apply filter_truncations to remove truncated transcripts from contiguous exon-boundary subsets. Apply filter_db(DB, 4, 0.95, recalc_abundances = F) to remove the least abundant 5% of isoforms per gene, then write the fully filtered database to GFF3 with write_gff_data (optionally via filter_top_gff to cap per-gene output).

## Related tools

- **IsoPops** (R package providing compile_raw_db, process_db, filter_db, filter_truncations, filter_top_gff, and write_gff_data used to compile, filter, and export the isoform catalog) — github.com/kellycochran/IsoPops
- **R** (runtime environment in which the IsoPops filtering functions and supporting tidyverse packages are executed)
- **dplyr** (data-manipulation dependency of IsoPops used in database filtering operations)
- **stringr** (string-processing dependency of IsoPops used in transcript/GFF handling)

## Examples

```
DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)
```

## Evaluation signals

- The R script runs without error through all filter_db, filter_truncations, and write_gff_data calls
- Output GFF3 file exists and parses as valid GFF3
- Isoform row count in the output GFF3 is approximately 4116, reduced from 8287 pre-filter isoforms
- No retained isoform has fewer than 4 non-contiguous genomic segments or abundance below the abund_cutoff of 1
- Per-gene isoform counts reflect removal of the bottom 5% abundance tier when recalc_abundances = F

## Limitations

- The catalogs may lack certain short and/or rare transcripts; absence of an isoform after filtering cannot be taken as evidence it is not expressed
- Some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of the library preparation protocol, biasing the filtered catalog
- Certain transcripts may be missing from the isoform catalogs entirely, so filtering cannot recover them

## Evidence

- [methods] filter_db call with exon and abundance thresholds: "DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)"
- [methods] truncation filtering step: "DB_filter_4ex_notrunc <- filter_truncations(DB_filter_4ex)"
- [methods] writing filtered data to GFF3: "write_gff_data(filter_top_gff(DB_filter_4ex_95perc, 10), "~/IsoPops_Data/fullfilter_top10.gff")"
- [methods] compiling the raw database from input files: "rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename)"
- [readme] IsoPops package purpose: "R package for analysis of isoform-diverse transcript datasets"
