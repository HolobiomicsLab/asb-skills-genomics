---
name: gene-identifier-to-name-mapping
description: Use when an IsoPops lrCaptureSeq catalog carries gene IDs only and transcripts
  must be attributed to gene names before per-gene isoform counts or per-experiment
  read totals can be tabulated.
when_to_use_negative:
- The database already carries gene names because process_db(rawDB, gene_ID_table)
  has already been applied to this catalog
- Attempting per-gene isoform counts from short-read RNA-seq data, where typical read
  lengths <200 bp cannot resolve the full-length sequence of multi-kilobase transcripts
- Treating these 30 genes' per-gene counts as typical of the average gene — they were
  selected because they showed evidence of transcript diversity and probably have
  more isoforms than the average gene
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0621
tools:
- name: R
  role: Reads the two-column gene_IDs_and_names.tsv mapping table with read.table(header
    = T, stringsAsFactors = F) so each transcript can be attributed to its gene
- name: IsoPops
  role: process_db(rawDB, gene_ID_table) applies the ID-to-name mapping to finalize
    gene information in the isoform database (DB_all)
  repo: github.com/kellycochran/IsoPops
- name: dplyr
  role: Groups mapped transcripts by gene name to tabulate the number of distinct
    isoforms per gene
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:58be8acbf38a969e2f75f8accb49cf1d839018c635a0e48c5940be20df8595c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoPops
  - R
  - dplyr
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gene-identifier-to-name-mapping

## Summary

Attributes every lrCaptureSeq transcript in an IsoPops catalog to its gene by reading a two-column gene ID-to-name table (gene_IDs_and_names.tsv) into R and applying process_db, enabling per-gene isoform tabulation such as the 4116 isoforms across 30 targeted genes.

## When to use

Use when an IsoPops lrCaptureSeq catalog carries gene IDs only and transcripts must be attributed to gene names before per-gene isoform counts or per-experiment read totals can be tabulated.

## When NOT to use

- The database already carries gene names because process_db(rawDB, gene_ID_table) has already been applied to this catalog
- Attempting per-gene isoform counts from short-read RNA-seq data, where typical read lengths <200 bp cannot resolve the full-length sequence of multi-kilobase transcripts
- Treating these 30 genes' per-gene counts as typical of the average gene — they were selected because they showed evidence of transcript diversity and probably have more isoforms than the average gene

## Inputs

- Two-column gene ID-to-name mapping file (e.g., ~/IsoPops_Data/gene_IDs_and_names.tsv) with a header row
- IsoPops raw database (rawDB) compiled from transcript, abundance, GFF, and ORF input files

## Outputs

- Gene-annotated IsoPops database (DB_all) in which every transcript is attributed to its gene
- Per-gene distinct isoform count table (e.g., Crb1 = 15, Megf11 = 234 isoforms)
- catalog_summary.csv with the per-gene count table, catalog-wide total (4116 isoforms across 30 genes), and per-experiment read totals (~132,000)

## How to apply

Read the two-column gene ID-to-name mapping file with read.table('~/IsoPops_Data/gene_IDs_and_names.tsv', header = TRUE, stringsAsFactors = FALSE) so every transcript can be attributed to its gene. First compile the catalog into a database via compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename), then pass rawDB and gene_ID_table to process_db(rawDB, gene_ID_table) to finalize gene information (DB_all). Group transcripts by gene name (dplyr) to tabulate the number of distinct isoforms per gene, and sum abundance read counts within each experiment for per-experiment read totals. Validate that the mapped catalog spans exactly 30 genes and 4116 distinct isoforms, that per-gene counts include Crb1 = 15 and Megf11 = 234 isoforms under the mouse-retina condition, and that per-experiment read totals are ~132,000 (±10%). Write the per-gene count table, catalog-wide total, and per-experiment totals to catalog_summary.csv.

## Related tools

- **R** (Reads the two-column gene_IDs_and_names.tsv mapping table with read.table(header = T, stringsAsFactors = F) so each transcript can be attributed to its gene)
- **IsoPops** (process_db(rawDB, gene_ID_table) applies the ID-to-name mapping to finalize gene information in the isoform database (DB_all)) — github.com/kellycochran/IsoPops
- **dplyr** (Groups mapped transcripts by gene name to tabulate the number of distinct isoforms per gene)

## Examples

```
gene_ID_table <- read.table('~/IsoPops_Data/gene_IDs_and_names.tsv', header = T, stringsAsFactors = F); DB_all <- process_db(rawDB, gene_ID_table)
```

## Evaluation signals

- After process_db, every transcript in DB_all carries a gene name; no unmapped gene IDs remain
- The mapped catalog totals 4116 distinct isoforms spanning exactly 30 targeted genes
- Per-gene isoform counts reproduce the reference values Crb1 = 15 and Megf11 = 234 under the mouse-retina condition
- Per-experiment summed read counts fall within ~132,000 ± 10%
- catalog_summary.csv contains the per-gene count table, the catalog-wide total, and the per-experiment totals

## Limitations

- Transcripts whose gene IDs are absent from gene_IDs_and_names.tsv cannot be attributed to a gene and drop out of per-gene counts
- The underlying catalog may lack certain short and/or rare transcripts; absence from the mapped catalog cannot be excluded
- Some isoforms smaller than 4.5 kb may have evaded detection due to the library-preparation size-selection step, capping per-gene isoform counts

## Evidence

- [methods] The gene ID-to-name mapping file is read into R with header = TRUE and stringsAsFactors = FALSE: "gene_ID_table <- read.table("~/IsoPops_Data/gene_IDs_and_names.tsv", header = T, stringsAsFactors = F)"
- [methods] process_db applies the mapping table to finalize gene information in the database: "DB_all <- process_db(rawDB, gene_ID_table)"
- [methods] The raw database that receives the mapping is compiled from transcript, abundance, GFF, and ORF files: "rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename)"
- [readme] The mapping skill operates within the IsoPots package for isoform-diverse transcript datasets: "R package for analysis of isoform-diverse transcript datasets"
