---
name: orf-prediction-with-genomic-correction
description: Use when you have full-length mRNA isoforms (FASTA plus GFF from PacBio
  lrCaptureSeq-style catalogs) and need to count or extract predicted ORFs/protein
  sequences per isoform and gene.
when_to_use_negative:
- 'Input is raw short-read RNA-seq: typical read lengths <200 bp cannot resolve full-length
  multi-kilobase transcripts, so isoform counts and full-length ORFs cannot be determined'
- The isoform catalog has not yet been filtered (off-target transcripts, truncated
  transcripts, and low-abundance isoforms will corrupt ORF counts)
- The target genome is not mouse mm10 and no appropriate reference is available for
  genomic correction
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0257
- http://edamontology.org/topic_3308
tools:
- name: IsoPops
  role: R package used to compile the raw isoform database, attach gene information,
    and filter to full-length high-confidence isoforms before ORF prediction
  repo: github.com/kellycochran/IsoPops
- name: SQANTI
  role: SQANTI v1.2 performs the ORF prediction step with genomic correction against
    the mm10 reference genome
- name: R
  role: Execution environment for the IsoPops pipeline (dplyr, magrittr, stringr,
    ggplot2 as dependencies)
- name: Pacific Biosciences (PacBio) long-read sequencing
  role: Source technology that generated the full-length lrCaptureSeq isoform catalog
    the ORFs are predicted from
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:4b2492cf56f88460d6c785603ea2ae1a55d5ed72cab78c90d04866b5578c3d6b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoPops
  - Pacific Biosciences (PacBio) long-read sequencing
  - R
  - SQANTI
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0257
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# orf-prediction-with-genomic-correction

## Summary

Predicts open reading frames (ORFs) from a filtered long-read isoform catalog (lrCaptureSeq of neural cell-surface molecules) using SQANTI v1.2 ORF prediction with genomic correction against the mm10 reference genome, to define predicted coding sequences and protein diversity.

## When to use

Use when you have full-length mRNA isoforms (FASTA plus GFF from PacBio lrCaptureSeq-style catalogs) and need to count or extract predicted ORFs/protein sequences per isoform and gene.

## When NOT to use

- Input is raw short-read RNA-seq: typical read lengths <200 bp cannot resolve full-length multi-kilobase transcripts, so isoform counts and full-length ORFs cannot be determined
- The isoform catalog has not yet been filtered (off-target transcripts, truncated transcripts, and low-abundance isoforms will corrupt ORF counts)
- The target genome is not mouse mm10 and no appropriate reference is available for genomic correction

## Inputs

- Transcript FASTA of full-length isoform sequences
- Isoform abundance table
- GFF annotation of isoform structures
- Candidate ORF FASTA
- Two-column gene ID/name table (gene_IDs_and_names.tsv)
- mm10 reference genome for genomic correction

## Outputs

- Set of unique predicted ORFs for the isoform catalog (2247 unique ORFs from 4116 isoforms in the reference run)
- Valid predicted protein FASTA file
- Processed IsoPops database saved with saveRDS

## How to apply

First prepare the isoform catalog in R with IsoPops: compile_raw_db from the transcript FASTA, abundance table, GFF, and ORF FASTA; then process_db with a 2-column gene ID/name table to attach gene identity. Restrict to full-length high-confidence isoforms via filter_db (exon_min_count=4, abund_cutoff=1), filter_truncations, and a second filter_db pass at a 0.95 abundance cutoff with recalc_abundances=FALSE. Then run SQANTI v1.2 ORF prediction over the filtered catalog, applying genomic correction against the mm10 reference genome so spurious internal ORFs are discarded in favor of coding sequences consistent with the genome. Save the processed database with saveRDS and write the predicted protein sequences to a FASTA file. Rationale: isoform number scales with ORF number across the 30 targeted genes, so reliable ORF calls are what turn an isoform catalog into protein-diversity statements.

## Related tools

- **IsoPops** (R package used to compile the raw isoform database, attach gene information, and filter to full-length high-confidence isoforms before ORF prediction) — github.com/kellycochran/IsoPops
- **SQANTI** (SQANTI v1.2 performs the ORF prediction step with genomic correction against the mm10 reference genome)
- **R** (Execution environment for the IsoPops pipeline (dplyr, magrittr, stringr, ggplot2 as dependencies))
- **Pacific Biosciences (PacBio) long-read sequencing** (Source technology that generated the full-length lrCaptureSeq isoform catalog the ORFs are predicted from)

## Examples

```
rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename); DB_all <- process_db(rawDB, gene_ID_table)
```

## Evaluation signals

- Output must contain 2247 unique ORFs from the 4116 isoform catalog, within a ±2% tolerance
- The predicted protein FASTA file must exist and parse as valid FASTA format
- The processed database round-trips via saveRDS/readRDS without loss of isoform or gene fields
- Isoform number typically scales with ORF number across the 30 targeted genes — check this monotone relationship per gene
- Filters must have been applied in order (exon_min_count=4, abund_cutoff=1, filter_truncations, then 0.95 abundance cutoff with recalc_abundances=FALSE) before prediction

## Limitations

- Some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of the library preparation protocol
- The catalogs may lack certain short and/or rare transcripts; absence cannot be excluded
- The 30 targeted genes likely have more isoforms than the average gene because they were selected for evidence of transcript diversity, so ORF counts are not representative of average genes

## Evidence

- [abstract] Applying this approach to neural cell-surface molecules, we identify thousands of unannotated isoforms expressed in retina and brain.: "Applying this approach to neural cell-surface molecules, we identify thousands of unannotated isoforms expressed in retina and brain."
- [methods] rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename): "rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename)"
- [methods] DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1): "DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)"
- [intro] Because typical RNA-seq read lengths are <200 bp, the method is not able to resolve the full-length sequence of multi-kilobase transcripts: "because typical RNA-seq read lengths are <200 bp, the method is not able to resolve the full-length sequence of multi-kilobase transcripts"
- [discussion] some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of our library preparation protocol: "some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of our library preparation protocol"
- [readme] To learn more about IsoPops and how it is used, check out: https://kellycochran.github.io/IsoPops/index.html: "To learn more about IsoPops and how it is used, check out: https://kellycochran.github.io/IsoPops/index.html"
