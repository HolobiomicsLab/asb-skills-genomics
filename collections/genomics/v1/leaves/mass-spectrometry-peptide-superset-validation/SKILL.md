---
name: mass-spectrometry-peptide-superset-validation
description: Use when validating that peptides identified by mass spectrometry against
  a novel isoform-derived ORF catalog are absent from a standard reference proteome
  (e.g., UniProt mouse) and confirming a published count of unannotated peptides.
when_to_use_negative:
- The goal is de novo peptide identification from raw MS spectra rather than validating
  a predicted-peptide superset against a reference database.
- The isoform catalog has not yet been filtered to predicted ORFs (no compile_raw_db/process_db/filter_db
  output), since the predicted peptide library must be built from the filtered ORF
  sequences.
- The target proteome is not one for which a standard reference (e.g., UniProt mouse)
  exists to diff against, making the 'unannotated' classification undefined.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: IsoPops
  role: R package used to compile, process, and filter the lrCaptureSeq isoform database
    from which the predicted ORF peptide FASTA is derived
  repo: https://github.com/kellycochran/IsoPops
- name: R
  role: Implementation environment for the IsoPops filtering workflow and the in-silico
    trypsin digestion / peptide-set diffing steps
- name: dplyr
  role: Imports of IsoPops used for data manipulation in the filtering pipeline feeding
    the peptide library
- name: stringr
  role: Imports of IsoPops used for string/sequence handling in database processing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:f74d67d2bf9bcea6341ed675abb7e29dfa61f8f2a4655da66a1d7b778470e67a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kellycochran/IsoPops
  tools:
  - IsoPops
  - R
  - dplyr
  - stringr
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# mass-spectrometry-peptide-superset-validation

## Summary

Reproduces the lrCaptureSeq tryptic peptide reference library by in-silico trypsin digestion (with the proline rule) of the IsoPops predicted-ORF proteome, diffs it against an identically digested UniProt mouse reference proteome to recover the unannotated-peptide set, and validates that all 35 MS-identified unannotated peptides (Supplementary Data 3) are contained in the derived set.

## When to use

Use when validating that peptides identified by mass spectrometry against a novel isoform-derived ORF catalog are absent from a standard reference proteome (e.g., UniProt mouse) and confirming a published count of unannotated peptides.

## When NOT to use

- The goal is de novo peptide identification from raw MS spectra rather than validating a predicted-peptide superset against a reference database.
- The isoform catalog has not yet been filtered to predicted ORFs (no compile_raw_db/process_db/filter_db output), since the predicted peptide library must be built from the filtered ORF sequences.
- The target proteome is not one for which a standard reference (e.g., UniProt mouse) exists to diff against, making the 'unannotated' classification undefined.

## Inputs

- Predicted ORF peptide FASTA from the filtered IsoPops lrCaptureSeq database (task_005 output)
- UniProt Mouse Reference proteome (FASTA or tabular download)
- Supplementary Data 3 list of 35 MS-identified unannotated peptides

## Outputs

- unannotated_peptides.tsv — unannotated peptide sequences with gene/ORF provenance
- Predicted tryptic peptide set (with proline rule)
- UniProt tryptic peptide set (with proline rule)
- Library-enrichment ratio (predicted peptide library size relative to UniProt peptide set)

## How to apply

Load the predicted ORF peptide FASTA produced by the IsoPops filtering pipeline (compile_raw_db, process_db, filter_db with exon_min_count=4, filter_truncations), which contains the protein sequences of the filtered lrCaptureSeq isoform catalog. Perform in-silico trypsin digestion on each predicted ORF protein applying the proline rule (no cleavage when Lys/Arg is followed by Pro) to build the predicted peptide set. Digest the UniProt mouse reference proteome identically and diff the two peptide sets to obtain the unannotated-peptide set (predicted peptides absent from UniProt). Check the unannotated set against the Supplementary Data 3 list of 35 MS-identified unannotated peptides, then compute the library-enrichment ratio of predicted-library size to UniProt peptide-set size and verify it falls in the 1.20–1.30x range (~25% enrichment).

## Related tools

- **IsoPops** (R package used to compile, process, and filter the lrCaptureSeq isoform database from which the predicted ORF peptide FASTA is derived) — https://github.com/kellycochran/IsoPops
- **R** (Implementation environment for the IsoPops filtering workflow and the in-silico trypsin digestion / peptide-set diffing steps)
- **dplyr** (Imports of IsoPops used for data manipulation in the filtering pipeline feeding the peptide library)
- **stringr** (Imports of IsoPops used for string/sequence handling in database processing)

## Examples

```
rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename)
```

## Evaluation signals

- All 35 sequences from Supplementary Data 3 are contained in the derived unannotated-peptide set.
- The library-enrichment ratio (predicted peptide library size / UniProt peptide set size) falls in the 1.20–1.30x range, consistent with ~25% enrichment.
- The unannotated_peptides.tsv rows each carry gene/ORF provenance and a peptide sequence absent from the identically digested UniProt mouse set.
- Digestion applies the proline rule consistently to both the predicted-ORF proteome and the UniProt reference so the diff is apples-to-apples.

## Limitations

- Typical short-read RNA-seq read lengths (<200 bp) cannot resolve full-length sequences of multi-kilobase transcripts, so isoform catalogs (and hence derived peptide libraries) built only from RNA-seq would be incomplete.
- Some isoforms smaller than 4.5 kb may have evaded detection due to the size selection step of the library preparation, meaning the derived peptide library may lack corresponding peptides.
- We cannot exclude the possibility that certain transcripts are missing from the catalogs, so the unannotated-peptide set is a lower bound on truly novel peptides.

## Evidence

- [abstract] By mass spectrometry we confirm expression of newly-discovered proteins on the cell surface in vivo: "By mass spectrometry we confirm expression of newly-discovered proteins on the cell surface in vivo."
- [methods] Compile a raw database from input files: "rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename)"
- [methods] Filter database by exon count, abundance, or truncated transcripts: "DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)"
- [discussion] Companion R software package: "we provide a companion R software package for analyzing and visualizing isoform catalogs"
- [readme] IsoPops README description: "R package for analysis of isoform-diverse transcript datasets"
