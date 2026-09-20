---
name: exon-count-threshold-filtering
description: Use when a raw PacBio Iso-Seq/lrCaptureSeq isoform annotation (transcript
  FASTA, abundance table, GFF, ORF FASTA) must be converted into a curated isoform
  catalog by removing isoforms with fewer than 4 exon segments, truncated transcripts,
  or the bottom 5% least-abundant isoforms per gene.
when_to_use_negative:
- Input is short-read RNA-seq data, which cannot resolve full-length sequences of
  multi-kilobase transcripts and therefore lacks the isoform-level annotation these
  filters require
- The goal is off-target transcript removal or database saving/loading (process_db,
  saveRDS/readRDS steps), which are separate workflow stages rather than exon-count/abundance
  filtering
- Isoforms smaller than ~4.5 kb or very rare transcripts are the target of interest,
  since the size-selection protocol and filtering intentionally remove short and rare
  isoforms
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: IsoPops
  role: R package providing compile_raw_db, process_db, filter_db, filter_truncations,
    filter_top_gff, and write_gff_data used to build and filter the isoform database
  repo: github.com/kellycochran/IsoPops
- name: R
  role: Environment in which the IsoPops filtering pipeline is scripted and run
- name: dplyr
  role: IsoPops Imports dependency used for data manipulation during database filtering
- name: stringr
  role: IsoPops Imports dependency used for string processing of transcript/GFF fields
    during filtering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:89c919b19c934653227a56d61a4e5ab282d99d56add4275b7823b09a65868968
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoPops
  - R
  - dplyr
  - stringr
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# exon-count-threshold-filtering

## Summary

An IsoPops-based filtering procedure for lrCaptureSeq isoform catalogs that drops low-quality and low-confidence isoforms using a minimum non-contiguous exon-segment count (exon_min_count = 4), a per-isoform abundance floor, and truncation removal, reducing 8287 pre-filter isoforms to a curated catalog of ~4116 isoforms across 30 targeted genes.

## When to use

Use when a raw PacBio Iso-Seq/lrCaptureSeq isoform annotation (transcript FASTA, abundance table, GFF, ORF FASTA) must be converted into a curated isoform catalog by removing isoforms with fewer than 4 exon segments, truncated transcripts, or the bottom 5% least-abundant isoforms per gene.

## When NOT to use

- Input is short-read RNA-seq data, which cannot resolve full-length sequences of multi-kilobase transcripts and therefore lacks the isoform-level annotation these filters require
- The goal is off-target transcript removal or database saving/loading (process_db, saveRDS/readRDS steps), which are separate workflow stages rather than exon-count/abundance filtering
- Isoforms smaller than ~4.5 kb or very rare transcripts are the target of interest, since the size-selection protocol and filtering intentionally remove short and rare isoforms

## Inputs

- Transcript FASTA from PacBio Iso-Seq / lrCaptureSeq processing
- Isoform abundance table
- Gene annotation GFF
- ORF FASTA
- Gene ID/name table (e.g. gene_IDs_and_names.tsv)

## Outputs

- Filtered IsoPops Database containing ~4116 isoforms across 30 targeted genes
- GFF3 file of fully filtered isoforms (e.g. via write_gff_data, optionally top-10 per gene via filter_top_gff)

## How to apply

Load the pre-filter Iso-Seq annotation into an IsoPops Database with compile_raw_db (transcript FASTA, abundance table, GFF, ORF FASTA) and process_db against a gene ID/name TSV. Apply filter_db(DB, exon_min_count = 4, abund_cutoff = 1) to drop isoforms with fewer than 4 non-contiguous genomic segments, then filter_truncations to remove truncated transcripts from contiguous exon-boundary subsets. Finally apply filter_db(DB, 4, 0.95, recalc_abundances = F) to remove the least abundant 5% of isoforms per gene, and write the result with write_gff_data (optionally subset with filter_top_gff). The rationale is that isoforms with fewer than 4 exon segments are likely artifacts or fragments, and truncation/low-abundance removal enriches for genuine full-length isoforms, yielding ~4116 isoforms from 8287 pre-filter isoforms over the 30 targeted genes.

## Related tools

- **IsoPops** (R package providing compile_raw_db, process_db, filter_db, filter_truncations, filter_top_gff, and write_gff_data used to build and filter the isoform database) — github.com/kellycochran/IsoPops
- **R** (Environment in which the IsoPops filtering pipeline is scripted and run)
- **dplyr** (IsoPops Imports dependency used for data manipulation during database filtering)
- **stringr** (IsoPops Imports dependency used for string processing of transcript/GFF fields during filtering)

## Examples

```
rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename); DB_all <- process_db(rawDB, gene_ID_table); DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1); DB_filter_4ex_notrunc <- filter_truncations(DB_filter_4ex); DB_filter_4ex_95perc <- filter_db(DB_filter_4ex_notrunc, 4, 0.95, recalc_abundances = F)
```

## Evaluation signals

- The R script runs without error end-to-end through compile_raw_db, process_db, both filter_db calls, and filter_truncations
- The output GFF3 file exists and is valid GFF3 format
- The filtered isoform row count is approximately 4116, down from 8287 pre-filter isoforms across the 30 targeted genes
- No retained isoform has fewer than 4 non-contiguous genomic segments (exon_min_count = 4 was enforced with abund_cutoff = 1)
- Per gene, the least abundant 5% of isoforms were removed (filter_db with 0.95 quantile, recalc_abundances = F)

## Limitations

- Some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of library preparation, so the catalog may lack short isoforms
- The catalogs may lack certain short and/or rare transcripts, and absence of a transcript cannot be excluded
- The 30 targeted genes likely have more isoforms than the average gene because they were selected for evidence of transcript diversity, so filtering thresholds may not transfer directly to other gene sets

## Evidence

- [methods] exon-count and abundance floor with filter_db: "DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)"
- [methods] truncation filtering step: "DB_filter_4ex_notrunc <- filter_truncations(DB_filter_4ex)"
- [methods] bottom-5% abundance removal and GFF export: "write_gff_data(filter_top_gff(DB_filter_4ex_95perc, 10), "~/IsoPops_Data/fullfilter_top10.gff")"
- [methods] database compilation inputs: "rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename)"
- [discussion] size-selection limitation on small isoforms: "some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of our library preparation protocol"
- [readme] package description: "IsoPops R package for analysis of isoform-diverse transcript datasets"
