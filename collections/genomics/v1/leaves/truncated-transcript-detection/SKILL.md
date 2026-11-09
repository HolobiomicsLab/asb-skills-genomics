---
name: truncated-transcript-detection
description: Use when a PacBio Iso-Seq/lrCaptureSeq isoform database (IsoPops Database
  compiled from transcript FASTA, abundance table, GFF, and ORF FASTA) may contain
  truncated transcripts that should be removed before downstream isoform counting
  or GFF3 export.
when_to_use_negative:
- Input is raw PacBio subreads or unclustered consensus reads that have not yet been
  compiled into an IsoPops Database with compile_raw_db/process_db.
- The goal is de novo transcript discovery or genome annotation rather than curating
  an existing isoform catalog from targeted lrCaptureSeq data.
- Short-read RNA-seq data alone is the input — typical read lengths <200 bp cannot
  resolve full-length multi-kilobase isoforms, so truncation filtering of full-length
  transcripts is not applicable.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: IsoPops
  role: R package whose filter_truncations function removes truncated transcripts
    from contiguous exon-boundary subsets of the filtered database
  repo: https://github.com/kellycochran/IsoPops
- name: R
  role: Runtime environment in which the IsoPops filtering pipeline is executed
- name: dplyr
  role: Imported dependency of IsoPops used for database table manipulation during
    filtering
- name: stringr
  role: Imported dependency of IsoPops used for string processing of transcript identifiers/sequences
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:a7a4ae6aafbacc3eb6df3fa00f9ea8aa756afce38fc8b2dd223e3c581d0852ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kellycochran/IsoPops
  tools:
  - IsoPops
  - R
  - dplyr
  - stringr
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# truncated-transcript-detection

## Summary

IsoPops-based detection and removal of truncated transcript artifacts from lrCaptureSeq isoform catalogs, applied after exon-count and abundance floors and before the bottom-5% abundance filter, to yield the final curated catalog (≈4116 isoforms from 8287 pre-filter isoforms across 30 targeted genes).

## When to use

Use when a PacBio Iso-Seq/lrCaptureSeq isoform database (IsoPops Database compiled from transcript FASTA, abundance table, GFF, and ORF FASTA) may contain truncated transcripts that should be removed before downstream isoform counting or GFF3 export.

## When NOT to use

- Input is raw PacBio subreads or unclustered consensus reads that have not yet been compiled into an IsoPops Database with compile_raw_db/process_db.
- The goal is de novo transcript discovery or genome annotation rather than curating an existing isoform catalog from targeted lrCaptureSeq data.
- Short-read RNA-seq data alone is the input — typical read lengths <200 bp cannot resolve full-length multi-kilobase isoforms, so truncation filtering of full-length transcripts is not applicable.

## Inputs

- IsoPops Database compiled from transcript FASTA, abundance table, GFF, and ORF FASTA (rawDB via compile_raw_db)
- Gene ID/name table (e.g., gene_IDs_and_names.tsv)
- Partially filtered database after filter_db(exon_min_count = 4, abund_cutoff = 1)

## Outputs

- Database with truncated transcripts removed (e.g., DB_filter_4ex_notrunc)
- Final curated isoform catalog (~4116 isoforms across 30 genes) written as GFF3 via write_gff_data

## How to apply

First compile the pre-filter Iso-Seq annotation into an IsoPops Database with compile_raw_db and process it against the gene ID/name table with process_db. Apply an exon-count floor and abundance cutoff with filter_db(exon_min_count = 4, abund_cutoff = 1), so that only isoforms with at least 4 non-contiguous genomic segments and abundance ≥ 1 remain. Then run filter_truncations on this filtered database: it removes truncated transcripts from contiguous exon-boundary subsets, i.e., isoforms whose exon boundaries are a contiguous subset of a longer isoform and are flagged as truncation artifacts. Finally, remove the least abundant 5% of isoforms per gene with filter_db(DB, 4, 0.95, recalc_abundances = F) and export with write_gff_data. Rationale: PacBio consensus transcripts can include truncated fragments of longer isoforms; removing them after the exon/abundance floors prevents fragment inflation of isoform counts before the per-gene bottom-5% abundance pruning.

## Related tools

- **IsoPops** (R package whose filter_truncations function removes truncated transcripts from contiguous exon-boundary subsets of the filtered database) — https://github.com/kellycochran/IsoPops
- **R** (Runtime environment in which the IsoPops filtering pipeline is executed)
- **dplyr** (Imported dependency of IsoPops used for database table manipulation during filtering)
- **stringr** (Imported dependency of IsoPops used for string processing of transcript identifiers/sequences)

## Examples

```
rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename); DB_all <- process_db(rawDB, gene_ID_table); DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1); DB_notrunc <- filter_truncations(DB_filter_4ex)
```

## Evaluation signals

- The pipeline script runs without error end-to-end from compile_raw_db through write_gff_data.
- The output GFF3 file exists and parses as valid GFF3 format.
- The output GFF3 isoform row count is approximately 4116, down from 8287 pre-filter isoforms.
- No retained isoform in the final catalog has fewer than 4 non-contiguous genomic segments (exon_min_count = 4 floor held).
- Isoforms removed by filter_truncations are exactly those with contiguous exon-boundary subsets of longer isoforms; spot-check a gene's isoform set before/after.

## Limitations

- Catalogs may lack certain short and/or rare transcripts, and some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of library preparation, so truncation filtering cannot recover missing full-length isoforms.
- recalc_abundances is set to FALSE in the final filter step, so abundances are not recomputed after removal; ordering of filter steps affects final counts.
- The 30 targeted genes likely have more isoforms than the average gene because they were selected for evidence of transcript diversity, so thresholds tuned here may not transfer directly to other gene sets.

## Evidence

- [methods] filter_truncations on the exon/abundance-filtered database: "DB_filter_4ex_notrunc <- filter_truncations(DB_filter_4ex)"
- [methods] abundance floor filter: "DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)"
- [methods] bottom-5% abundance removal filter: "DB_filter_4ex_95perc"
- [discussion] size-selection detection limitation: "some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of our library preparation protocol"
- [readme] package description in repository README: "R package for analysis of isoform-diverse transcript datasets"
