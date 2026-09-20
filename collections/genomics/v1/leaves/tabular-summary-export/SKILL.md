---
name: tabular-summary-export
description: Use when a long-read isoform catalog (e.g. PacBio lrCaptureSeq abundance
  tables processed with IsoPops) must be reduced to per-gene isoform counts, total
  distinct isoforms, and per-experiment read totals for export or comparison with
  public annotations.
when_to_use_negative:
- 'The input is short-read RNA-seq data only: typical RNA-seq read lengths (<200 bp)
  cannot resolve full-length sequences of multi-kilobase transcripts, so per-gene
  full-length isoform counts cannot be computed this way.'
- You need quantitative isoform expression time/place mapping via single-cell integration
  rather than a static catalog summary.
- The database still contains off-target or truncated transcripts and you have not
  yet run the IsoPots filtering steps (filter_db, filter_truncations) that precede
  a defensible summary.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: IsoPops
  role: R package used to compile_raw_db and process_db the transcript, abundance,
    GFF, and ORF files into a gene-attributed isoform database that the summary tabulates
  repo: github.com/kellycochran/IsoPops
- name: R
  role: runtime environment in which the abundance and gene ID-to-name tables are
    read (read.table with header = TRUE) and counts are tabulated and exported to
    CSV
- name: dplyr
  role: Imports dependency of IsoPops used for grouping transcripts by gene name and
    aggregating counts per gene and per experiment
  repo: github.com/kellycochran/IsoPops
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:3bef731cba5f5f8e332052da67a99d2c484d0307b3eb14001d3678035f918d3a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoPops
  - R
  - dplyr
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# tabular-summary-export

## Summary

Summarize the lrCaptureSeq isoform catalog produced by the IsoPops pipeline into a per-gene isoform-count table plus catalog-wide and per-experiment read totals, exported to a CSV (catalog_summary.csv). It quantifies how many mRNA isoforms the 30 targeted genes yield relative to public annotations.

## When to use

Use when a long-read isoform catalog (e.g. PacBio lrCaptureSeq abundance tables processed with IsoPops) must be reduced to per-gene isoform counts, total distinct isoforms, and per-experiment read totals for export or comparison with public annotations.

## When NOT to use

- The input is short-read RNA-seq data only: typical RNA-seq read lengths (<200 bp) cannot resolve full-length sequences of multi-kilobase transcripts, so per-gene full-length isoform counts cannot be computed this way.
- You need quantitative isoform expression time/place mapping via single-cell integration rather than a static catalog summary.
- The database still contains off-target or truncated transcripts and you have not yet run the IsoPots filtering steps (filter_db, filter_truncations) that precede a defensible summary.

## Inputs

- per-isoform full-length read-count (abundance) table from the deposited lrCaptureSeq package data
- two-column gene ID-to-name mapping table (e.g. ~/IsoPops_Data/gene_IDs_and_names.tsv, read with header = TRUE)
- transcript sequence, GFF annotation, and ORF files passed to compile_raw_db

## Outputs

- catalog_summary.csv containing the per-gene isoform count table, catalog-wide total (4116 isoforms), and per-experiment read totals (~132,000 reads each)
- per-gene isoform counts (e.g. Crb1 = 15, Megf11 = 234) attributing every transcript to its gene

## How to apply

In R, load the per-isoform full-length read-count (abundance) table and the two-column gene ID-to-name mapping with read.table(..., header = TRUE). Build the catalog with IsoPops: compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename) then process_db(rawDB, gene_ID_table) to attach gene names and finalize gene information. Group transcripts by gene name (e.g. with dplyr) and tabulate distinct isoforms per gene; sum read counts within each lrCaptureSeq experiment for per-experiment totals, and count total distinct isoforms catalog-wide. Write the combined per-gene count table, catalog-wide total, and per-experiment totals to catalog_summary.csv. The expected output for the mouse-retina condition is 4116 isoforms across 30 genes with ~132,000 full-length reads per experiment, including Crb1 = 15 and Megf11 = 234 isoforms.

## Related tools

- **IsoPops** (R package used to compile_raw_db and process_db the transcript, abundance, GFF, and ORF files into a gene-attributed isoform database that the summary tabulates) — github.com/kellycochran/IsoPops
- **R** (runtime environment in which the abundance and gene ID-to-name tables are read (read.table with header = TRUE) and counts are tabulated and exported to CSV)
- **dplyr** (Imports dependency of IsoPops used for grouping transcripts by gene name and aggregating counts per gene and per experiment) — github.com/kellycochran/IsoPops

## Examples

```
gene_ID_table <- read.table("~/IsoPops_Data/gene_IDs_and_names.tsv", header = T, stringsAsFactors = F); DB_all <- process_db(rawDB, gene_ID_table)
```

## Evaluation signals

- The catalog total equals 4116 distinct isoforms spanning exactly 30 genes.
- Per-experiment summed read counts fall within ~132,000 ± 10%.
- The per-gene table contains Crb1 = 15 isoforms and Megf11 = 234 isoforms under the mouse-retina condition.
- Every transcript in the summary is attributed to a gene via the gene ID-to-name mapping (no NA gene names).
- catalog_summary.csv contains all three components: per-gene counts, catalog-wide total, and per-experiment totals.

## Limitations

- Some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of the library preparation, so the catalog may lack certain short and/or rare transcripts.
- Absence of a transcript cannot be excluded: certain transcripts may be missing from the isoform catalogs.
- The 30 targeted genes likely have more isoforms than the average gene because they were selected for evidence of transcript diversity, so catalog sizes should not be generalized to other gene classes.

## Evidence

- [results] The lrCaptureSeq catalog contains 4116 isoforms of the 30 targeted genes, approximately one order of magnitude greater than the number currently annotated in public databases.: [evidence span withheld — anchor status: not_found]
- [methods] compile_raw_db invocation: "rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename)"
- [methods] gene mapping table read: "gene_ID_table <- read.table("~/IsoPops_Data/gene_IDs_and_names.tsv", header = T, stringsAsFactors = F)"
- [discussion] size-selection caveat: "some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of our library preparation protocol"
- [readme] README package description: "IsoPops R package for analysis of isoform-diverse transcript datasets"
