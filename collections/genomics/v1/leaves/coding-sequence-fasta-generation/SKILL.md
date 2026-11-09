---
name: coding-sequence-fasta-generation
description: Use when you have a filtered full-length isoform catalog (e.g. PacBio
  lrCaptureSeq transcripts curated in IsoPops) and need to enumerate and export the
  unique predicted open reading frames as protein sequences.
when_to_use_negative:
- Input transcripts are short-read RNA-seq assemblies rather than full-length long-read
  (PacBio) isoforms, since typical RNA-seq read lengths are <200 bp and cannot resolve
  full-length multi-kilobase transcripts
- The isoform catalog has not been filtered for full-length, high-confidence isoforms
  (exon count, abundance, truncation status), because ORF prediction over off-target
  or truncated transcripts will inflate the unique ORF count
- The goal is transcript-level isoform diversity (exon structure, GFF output) rather
  than protein-level coding-sequence enumeration, where IsoPops GFF-export functions
  like filter_top_gff and write_gff_data are more appropriate
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SQANTI
  role: Predicts ORFs / coding sequences over the filtered isoform catalog, applying
    genomic correction against mm10 to define predicted coding sequences
- name: IsoPops
  role: R package used to compile the raw database from transcript FASTA, abundance
    table, GFF, and ORF FASTA inputs and to filter the catalog to full-length, high-confidence
    isoforms before ORF prediction
  repo: github.com/kellycochran/IsoPops
- name: R
  role: Environment in which the IsoPops pipeline runs (compile_raw_db, process_db,
    filter_db, filter_truncations, saveRDS)
  repo: github.com/kellycochran/IsoPops
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:24d89154afdec59fbbeaa1f6bf11b38180961d26af99677682eb70498041399c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoPops
  - R
  - SQANTI
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# coding-sequence-fasta-generation

## Summary

Predicts protein-coding sequences for a curated long-read isoform catalog by running SQANTI v1.2 ORF prediction with genomic correction against the mm10 reference genome, yielding a set of unique predicted ORFs and a protein FASTA file for downstream proteomic and isoform-diversity analyses.

## When to use

Use when you have a filtered full-length isoform catalog (e.g. PacBio lrCaptureSeq transcripts curated in IsoPops) and need to enumerate and export the unique predicted open reading frames as protein sequences.

## When NOT to use

- Input transcripts are short-read RNA-seq assemblies rather than full-length long-read (PacBio) isoforms, since typical RNA-seq read lengths are <200 bp and cannot resolve full-length multi-kilobase transcripts
- The isoform catalog has not been filtered for full-length, high-confidence isoforms (exon count, abundance, truncation status), because ORF prediction over off-target or truncated transcripts will inflate the unique ORF count
- The goal is transcript-level isoform diversity (exon structure, GFF output) rather than protein-level coding-sequence enumeration, where IsoPops GFF-export functions like filter_top_gff and write_gff_data are more appropriate

## Inputs

- Transcript FASTA of full-length isoform sequences
- Isoform abundance table
- Gene annotation GFF file
- ORF FASTA file from the isoform catalog
- Two-column gene ID/name table (e.g. gene_IDs_and_names.tsv)
- mm10 mouse reference genome for SQANTI genomic correction

## Outputs

- Set of unique predicted ORFs (2247 unique ORFs for the 4116-isoform lrCaptureSeq catalog)
- Predicted protein sequences in valid FASTA format
- Processed IsoPops database (RDS) with ORF annotations attached

## How to apply

First curate the isoform set in R with the IsoPops package: compile_raw_db from the transcript FASTA, abundance table, GFF, and ORF FASTA files; process_db with the 2-column gene ID/name table; then filter_db with exon_min_count=4 and abund_cutoff=1, filter_truncations, and a final filter_db pass at a 0.95 abundance cutoff with recalc_abundances=FALSE so only full-length, high-confidence isoforms remain. Run SQANTI v1.2 ORF prediction over this filtered catalog, applying genomic correction against the mm10 reference genome so that predicted coding sequences are consistent with the mouse genome annotation. Count distinct ORFs across the surviving isoforms (the lrCaptureSeq catalog of 4116 isoforms from 30 targeted neural cell-surface genes yields 2247 unique ORFs) and write the predicted protein sequences to a FASTA file. Save the processed database with saveRDS so the ORF-annotated catalog can be reloaded without recomputation.

## Related tools

- **SQANTI** (Predicts ORFs / coding sequences over the filtered isoform catalog, applying genomic correction against mm10 to define predicted coding sequences)
- **IsoPops** (R package used to compile the raw database from transcript FASTA, abundance table, GFF, and ORF FASTA inputs and to filter the catalog to full-length, high-confidence isoforms before ORF prediction) — github.com/kellycochran/IsoPops
- **R** (Environment in which the IsoPops pipeline runs (compile_raw_db, process_db, filter_db, filter_truncations, saveRDS)) — github.com/kellycochran/IsoPops

## Examples

```
rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename); DB_all <- process_db(rawDB, gene_ID_table); DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)
```

## Evaluation signals

- The unique-ORF count matches 2247 within a ±2% tolerance when run on the 4116-isoform lrCaptureSeq catalog
- The predicted protein output file exists and parses as valid FASTA format
- Every isoform passing the final 0.95 abundance-cutoff filter with recalc_abundances=FALSE has an assigned predicted ORF
- ORF-to-isoform scaling is coherent: isoform number typically scales with ORF number across the 30 targeted genes
- The processed database round-trips via saveRDS/readRDS and retains ORF annotations

## Limitations

- The catalogs may lack certain short and/or rare transcripts, and some isoforms smaller than 4.5 kb may have evaded detection due to the size-selection step of library preparation, so the ORF set is not guaranteed complete
- Absence of a transcript cannot be excluded: certain transcripts may be missing from the isoform catalogs
- The 30 targeted genes likely have more isoforms than the average gene because they were selected for evidence of transcript diversity, so ORF counts from this catalog are not representative of typical genes

## Evidence

- [methods] Compile a raw database from input files: "rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename)"
- [methods] Filter database by exon count, abundance, or truncated transcripts: "DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)"
- [methods] Filter truncated transcripts: "DB_filter_4ex_notrunc <- filter_truncations(DB_filter_4ex)"
- [readme] R package for isoform analysis: "IsoPops
R package for analysis of isoform-diverse transcript datasets"
- [discussion] Catalog completeness caveat: "some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of our library preparation protocol"
