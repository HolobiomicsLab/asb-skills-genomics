---
name: orf-protein-sequence-processing
description: Use when you have predicted ORF peptide/protein FASTA from an isoform
  catalog and need in-silico trypsin digestion, reference-proteome diffing, or extraction
  of peptides absent from UniProt for targeted mass spectrometry.
when_to_use_negative:
- Input is raw transcript/GFF/abundance files rather than processed ORF protein sequences
  — use IsoPops database compilation and filtering (compile_raw_db/process_db/filter_db)
  first
- The goal is peptide identification from raw mass-spectrometry spectra (mzML/RAW)
  rather than building a predicted peptide reference from ORF sequences
- Protein sequences already come from a fully annotated reference proteome, so a UniProt-absence
  diff would be trivially empty
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0080
tools:
- name: IsoPops
  role: R package used to compile, process, and filter the isoform/ORF database from
    which the predicted ORF peptide sequences are taken
  repo: github.com/kellycochran/IsoPops
- name: R
  role: Implementation language for IsoPops database handling and the in-silico digestion
    / UniProt peptide diff workflow
  repo: github.com/kellycochran/IsoPops
- name: Pacific Biosciences (PacBio) long-read sequencing
  role: lrCaptureSeq platform that generated the full-length isoform catalog underlying
    the predicted ORF proteome
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:12e25be54cc0b9fd2504a1081d85aad4dd6b83277c8b2083c44a6830895d6f64
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoPops
  - Pacific Biosciences (PacBio) long-read sequencing
  - R
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# orf-protein-sequence-processing

## Summary

Process predicted ORF protein sequences from an lrCaptureSeq/PacBio isoform catalog (managed with the IsoPops R package) into an in-silico tryptic peptide reference, and diff it against a reference proteome such as UniProt Mouse to extract unannotated peptides for MS validation.

## When to use

Use when you have predicted ORF peptide/protein FASTA from an isoform catalog and need in-silico trypsin digestion, reference-proteome diffing, or extraction of peptides absent from UniProt for targeted mass spectrometry.

## When NOT to use

- Input is raw transcript/GFF/abundance files rather than processed ORF protein sequences — use IsoPops database compilation and filtering (compile_raw_db/process_db/filter_db) first
- The goal is peptide identification from raw mass-spectrometry spectra (mzML/RAW) rather than building a predicted peptide reference from ORF sequences
- Protein sequences already come from a fully annotated reference proteome, so a UniProt-absence diff would be trivially empty

## Inputs

- Predicted ORF peptide/protein FASTA from the filtered IsoPops database (output of ORF-based isoform catalog processing)
- UniProt Mouse Reference proteome (protein sequence database)
- MS-identified unannotated peptide list (Supplementary Data 3, 35 peptides)

## Outputs

- Predicted tryptic peptide set (proline rule) from the lrCaptureSeq ORF proteome
- Unannotated-peptide set (predicted peptides absent from UniProt mouse reference)
- Library-enrichment ratio (predicted library size / UniProt peptide set size, expected ~1.20–1.30x)
- unannotated_peptides.tsv with peptide sequences and gene/ORF provenance

## How to apply

Load the predicted ORF peptide FASTA produced from the filtered IsoPops database (IsoPops workflow: compile_raw_db → process_db → filter_db → filter_truncations). Digest each predicted ORF protein in silico with trypsin applying the proline rule (no cleavage when Lys/Arg is followed by Pro), and digest the UniProt Mouse Reference proteome identically. Diff the two peptide sets to obtain the unannotated-peptide set, then compare against the MS-identified list (Supplementary Data 3; 35 unannotated peptides) to confirm all MS hits are contained in the derived set. Compute the library-enrichment ratio of the predicted peptide library relative to the UniProt peptide set and verify it falls in the ~1.20–1.30x range (~25% enrichment). Finally write unannotated peptide sequences with gene/ORF provenance to a TSV (e.g., unannotated_peptides.tsv).

## Related tools

- **IsoPops** (R package used to compile, process, and filter the isoform/ORF database from which the predicted ORF peptide sequences are taken) — github.com/kellycochran/IsoPops
- **R** (Implementation language for IsoPops database handling and the in-silico digestion / UniProt peptide diff workflow) — github.com/kellycochran/IsoPops
- **Pacific Biosciences (PacBio) long-read sequencing** (lrCaptureSeq platform that generated the full-length isoform catalog underlying the predicted ORF proteome)

## Examples

```
DB_filter_4ex_notrunc <- filter_truncations(filter_db(process_db(rawDB, gene_ID_table), exon_min_count = 4, abund_cutoff = 1))
```

## Evaluation signals

- All 35 MS-identified unannotated peptides from Supplementary Data 3 are contained in the derived predicted-peptide minus UniProt diff set
- Library-enrichment ratio (predicted peptide library size / UniProt peptide set size) falls in the 1.20–1.30x range (~25% enrichment)
- Trypsin digestion applied the proline rule: no cleavage products where Lys/Arg is immediately followed by Pro
- unannotated_peptides.tsv contains one row per unannotated peptide with non-empty gene/ORF provenance columns and no UniProt-matching sequences
- Identical digestion parameters (trypsin + proline rule) applied to both the predicted ORF set and the UniProt mouse reference set

## Limitations

- The isoform catalog may lack certain short and/or rare transcripts; absence of a transcript or peptide cannot be excluded
- Some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of the library preparation protocol
- Unannotated-peptide validation is limited to the 30 targeted genes, which likely have more isoforms than the average gene since they were selected for evidence of transcript diversity

## Evidence

- [methods] rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename): "rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename)"
- [abstract] By mass spectrometry we confirm expression of newly-discovered proteins on the cell surface in vivo.: "By mass spectrometry we confirm expression of newly-discovered proteins on the cell surface in vivo."
- [discussion] we provide a companion R software package for analyzing and visualizing isoform catalogs: "we provide a companion R software package for analyzing and visualizing isoform catalogs"
- [readme] R package for analysis of isoform-diverse transcript datasets: "R package for analysis of isoform-diverse transcript datasets"
