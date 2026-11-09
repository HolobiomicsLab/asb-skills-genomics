---
name: r-package-data-processing
description: Use when working with PacBio lrCaptureSeq-derived full-length isoform
  catalogs (transcript, abundance, GFF, ORF files) that need compilation into a structured
  R database, filtering, or downstream isoform-level analysis in R.
when_to_use_negative:
- Input data is short-read RNA-seq reads (<200 bp) rather than full-length isoform
  sequences — such reads cannot resolve full-length multi-kilobase transcripts
- The goal is de novo isoform discovery without targeted CaptureSeq pull-down data;
  IsoPops processes curated isoform catalogs rather than raw PacBio subreads
- A simple one-off script would suffice and no isoform-specific database filtering,
  k-mer analysis, or Lorenz-style visualization is needed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: IsoPops
  role: Core R package providing compile_raw_db, process_db, filter_db, filter_truncations,
    filter_db_by_ID, get_kmer_counts, kmer_PCA, kmer_tSNE, cluster_isoforms, jellyfish_plot,
    and write_gff_data for isoform database processing
  repo: github.com/kellycochran/IsoPops
- name: R
  role: Execution environment and scripting language for the IsoPops companion package
    and all downstream analysis
  repo: github.com/kellycochran/IsoPops
- name: dplyr
  role: Imports for tabular manipulation of isoform/gene tables within IsoPops
  repo: github.com/kellycochran/IsoPops
- name: magrittr
  role: Imports for pipe-based chaining of IsoPops processing steps
  repo: github.com/kellycochran/IsoPops
- name: stringr
  role: Imports for string handling of transcript identifiers and GFF fields
  repo: github.com/kellycochran/IsoPops
- name: ggplot2
  role: Imports for plotting isoform abundance and diversity visualizations (e.g.
    jellyfish_plot)
  repo: github.com/kellycochran/IsoPops
- name: vegan
  role: Used for computing Shannon diversity indices from isoform relative abundances
    in downstream ranking analyses
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:d70b40c19f62adc2984721d7b69929b0602b0b3be46e9cfbec67b3a35d095688
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoPops
  - R
  - dplyr
  - ggplot2
  - magrittr
  - stringr
  - vegan
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9583333333333334
n_claims: 24
---

# r-package-data-processing

## Summary

Process and analyze long-read mRNA isoform datasets using the IsoPops R package: compile raw isoform databases from transcript, abundance, GFF, and ORF files; filter by exon count, abundance, and truncation; and support downstream visualization/analysis with tidyverse packages.

## When to use

Use when working with PacBio lrCaptureSeq-derived full-length isoform catalogs (transcript, abundance, GFF, ORF files) that need compilation into a structured R database, filtering, or downstream isoform-level analysis in R.

## When NOT to use

- Input data is short-read RNA-seq reads (<200 bp) rather than full-length isoform sequences — such reads cannot resolve full-length multi-kilobase transcripts
- The goal is de novo isoform discovery without targeted CaptureSeq pull-down data; IsoPops processes curated isoform catalogs rather than raw PacBio subreads
- A simple one-off script would suffice and no isoform-specific database filtering, k-mer analysis, or Lorenz-style visualization is needed

## Inputs

- Transcript sequence file (e.g. PacBio lrCaptureSeq isoform sequences)
- Isoform abundance file (per-isoform read counts)
- GFF annotation file (filter_top_gff output can be exported with write_gff_data)
- ORF prediction file
- Gene ID/name table (TSV, e.g. gene_IDs_and_names.tsv)

## Outputs

- IsoPops database objects (rawDB, DB_all, filtered DBs such as DB_filter_4ex or DB_filter_4ex_95perc)
- RDS files saved via saveRDS for reuse
- GFF files exported via write_gff_data (e.g. fullfilter_top10.gff)
- K-mer count matrices, PCA/t-SNE embeddings, isoform clusters, and per-gene abundance/diversity tables (e.g. shannon_abundance_ranking.csv)

## How to apply

Compile input files into a raw database with compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename), then add gene metadata with process_db(rawDB, gene_ID_table), where gene_ID_table is loaded via read.table from a TSV such as gene_IDs_and_names.tsv. Apply filters per analysis goals: filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1) for minimum exon counts and abundance, filter_truncations() to drop truncated transcripts, and an optional 95% abundance cutoff (e.g. producing DB_filter_4ex_95perc) — set recalc_abundances appropriately (e.g. recalc_abundances = F) when downstream rankings must preserve original abundances. Persist databases with saveRDS/readRDS so filtered states can be reused across analyses (k-mer counting, PCA, t-SNE, clustering, diversity ranking). Use dplyr, magrittr, stringr, and ggplot2 for table manipulation and Lorenz-style (jellyfish_plot) visualizations of per-gene isoform abundance portfolios.

## Related tools

- **IsoPops** (Core R package providing compile_raw_db, process_db, filter_db, filter_truncations, filter_db_by_ID, get_kmer_counts, kmer_PCA, kmer_tSNE, cluster_isoforms, jellyfish_plot, and write_gff_data for isoform database processing) — github.com/kellycochran/IsoPops
- **R** (Execution environment and scripting language for the IsoPops companion package and all downstream analysis) — github.com/kellycochran/IsoPops
- **dplyr** (Imports for tabular manipulation of isoform/gene tables within IsoPops) — github.com/kellycochran/IsoPops
- **magrittr** (Imports for pipe-based chaining of IsoPops processing steps) — github.com/kellycochran/IsoPops
- **stringr** (Imports for string handling of transcript identifiers and GFF fields) — github.com/kellycochran/IsoPops
- **ggplot2** (Imports for plotting isoform abundance and diversity visualizations (e.g. jellyfish_plot)) — github.com/kellycochran/IsoPops
- **vegan** (Used for computing Shannon diversity indices from isoform relative abundances in downstream ranking analyses)

## Examples

```
DB_all <- process_db(compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename), gene_ID_table); DB_f <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)
```

## Evaluation signals

- The compiled database (rawDB) loads without error and process_db attaches gene metadata from gene_IDs_and_names.tsv with correct stringAsFactors = F handling
- After filter_db(exon_min_count = 4, abund_cutoff = 1) and filter_truncations(), no truncated transcripts or low-abundance/low-exon-count isoforms remain in the filtered database
- readRDS restores the saved DB (e.g. DB_filter_4ex_95perc) with identical structure and abundances to the pre-save object when recalc_abundances = F was used
- Downstream outputs are reproducible: e.g. Shannon ranking identifies Nrxn3 as the most isoform-diverse gene (~900 isoforms) and Crb1-B as the most abundant isoform in the filtered dataset
- write_gff_data(filter_top_gff(DB, 10)) emits a well-formed GFF containing the top 10 isoforms per gene

## Limitations

- Short and/or rare transcripts may be missing from the catalogs; some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of library preparation
- The 30 targeted genes probably have more isoforms than the average gene, since they were selected for evidence of transcript diversity, so diversity metrics may not generalize
- Isoform abundance estimates depend on the filter parameters chosen (exon_min_count, abund_cutoff, 95% abundance cutoff); changing recalc_abundances alters downstream rankings

## Evidence

- [methods] rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename): "rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename)"
- [methods] DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1): "DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)"
- [methods] gene_ID_table <- read.table("~/IsoPops_Data/gene_IDs_and_names.tsv", header = T, stringsAsFactors = F): "gene_ID_table <- read.table("~/IsoPops_Data/gene_IDs_and_names.tsv", header = T, stringsAsFactors = F)"
- [methods] write_gff_data(filter_top_gff(DB_filter_4ex_95perc, 10), "~/IsoPops_Data/fullfilter_top10.gff"): "write_gff_data(filter_top_gff(DB_filter_4ex_95perc, 10), "~/IsoPops_Data/fullfilter_top10.gff")"
- [discussion] some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of our library preparation protocol: "some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of our library preparation protocol"
- [readme] R package for analysis of isoform-diverse transcript datasets: "R package for analysis of isoform-diverse transcript datasets"
