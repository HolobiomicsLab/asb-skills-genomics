---
name: in-silico-protease-digestion-of-orf-proteome
description: Use when a long-read isoform catalog yields predicted ORF proteins and
  you need to enumerate which tryptic peptides are novel relative to a reference proteome
  (e.g., UniProt mouse) before targeted mass spectrometry.
when_to_use_negative:
- Input is an experimentally acquired mass-spectrometry peak list rather than predicted
  protein sequences — use peptide-spectrum matching instead
- The isoform catalog has not been filtered (no filter_db with exon_min_count = 4,
  abund_cutoff = 1, no filter_truncations), so ORFs are unvalidated and the peptide
  library is noisy
- The protease is not trypsin or the proline rule should not apply; the diff logic
  and enrichment-ratio expectations are calibrated for tryptic digestion
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: IsoPops
  role: R package used to compile, process, and filter the lrCaptureSeq isoform database
    whose predicted ORF proteins supply the peptide FASTA to digest
  repo: github.com/kellycochran/IsoPops
- name: R
  role: Implementation environment for the in-silico digestion, UniProt peptide-set
    diff, and unannotated_peptides.tsv export
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17009-7
    title: Comprehensive identification of mRNA isoforms reveals the diversity of
      neural cell-surface molecules with roles in retinal development and disease
  source_content_hash: sha256:dfe3aaecd2a3a33162de47c4a3ddc3c61eec19ad8f6f7bfed46a16371ff48730
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoPops
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# In-silico protease digestion of ORF proteome

## Summary

Digests predicted ORF protein sequences (e.g., from the lrCaptureSeq IsoPops isoform catalog) in silico with trypsin plus the proline rule and diffs the resulting peptide set against a trypsin-digested UniProt Mouse Reference proteome to define unannotated peptides for MS-based validation of newly-discovered proteins.

## When to use

Use when a long-read isoform catalog yields predicted ORF proteins and you need to enumerate which tryptic peptides are novel relative to a reference proteome (e.g., UniProt mouse) before targeted mass spectrometry.

## When NOT to use

- Input is an experimentally acquired mass-spectrometry peak list rather than predicted protein sequences — use peptide-spectrum matching instead
- The isoform catalog has not been filtered (no filter_db with exon_min_count = 4, abund_cutoff = 1, no filter_truncations), so ORFs are unvalidated and the peptide library is noisy
- The protease is not trypsin or the proline rule should not apply; the diff logic and enrichment-ratio expectations are calibrated for tryptic digestion

## Inputs

- Predicted ORF peptide FASTA from the filtered IsoPops database (task_005 output)
- UniProt Mouse Reference proteome (protein FASTA)
- Supplementary Data 3 list of 35 MS-identified unannotated peptides

## Outputs

- Predicted tryptic peptide set from the ORF proteome (proline rule applied)
- Unannotated-peptide set (predicted peptides absent from UniProt peptides)
- unannotated_peptides.tsv with peptide sequences and gene/ORF provenance
- Library-enrichment ratio (predicted library size relative to UniProt peptide set)

## How to apply

Load the predicted ORF peptide FASTA produced from the filtered IsoPops database (built with compile_raw_db, process_db, filter_db with exon_min_count = 4 and abund_cutoff = 1, and filter_truncations). Perform in-silico trypsin digestion of every predicted ORF protein applying the proline rule (no cleavage when Lys/Arg is followed by Pro) to produce the predicted peptide set. Digest the UniProt Mouse Reference proteome identically and subtract its peptide set from the predicted set to obtain the unannotated-peptide set. Validate by confirming that all 35 MS-identified unannotated peptides from Supplementary Data 3 are contained in the derived set, and compute the library-enrichment ratio, which should fall in the ~1.20–1.30x range (~25% enrichment over UniProt peptides).

## Related tools

- **IsoPops** (R package used to compile, process, and filter the lrCaptureSeq isoform database whose predicted ORF proteins supply the peptide FASTA to digest) — github.com/kellycochran/IsoPops
- **R** (Implementation environment for the in-silico digestion, UniProt peptide-set diff, and unannotated_peptides.tsv export)

## Examples

```
Rscript -e 'library(IsoPops); rawDB <- compile_raw_db(transcript_filename, abundance_filename, gff_filename, ORF_filename); DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)'
```

## Evaluation signals

- All 35 peptide sequences from Supplementary Data 3 are contained in the derived unannotated-peptide set
- Library-enrichment ratio (predicted peptide library size / UniProt peptide set) falls in the 1.20–1.30x range (~25% enrichment)
- Every peptide in the unannotated set is absent from the identically digested (trypsin + proline rule) UniProt Mouse Reference peptide set
- unannotated_peptides.tsv carries gene/ORF provenance for each peptide sequence
- Digestion respects the proline rule: no predicted cleavage at Lys/Arg immediately followed by Pro

## Limitations

- Catalogs may lack certain short and/or rare transcripts — the article notes 'we cannot exclude the possibility that certain transcripts are missing from the catalogs' and that some isoforms smaller than 4.5 kb may have evaded detection due to library size selection, so the derived peptide library is not exhaustive
- The 30 targeted genes were selected for evidence of transcript diversity, so peptide enrichment may not generalize to average genes
- Identical digestion parameters must be applied to both predicted and UniProt proteomes or the peptide diff is invalid

## Evidence

- [abstract] MS-identified unannotated peptides absent from UniProt: "By mass spectrometry we confirm expression of newly-discovered proteins on the cell surface in vivo."
- [methods] IsoPops filtering thresholds precede ORF peptide export: "DB_filter_4ex <- filter_db(DB_all, exon_min_count = 4, abund_cutoff = 1)"
- [readme] IsoPops package role: "R package for analysis of isoform-diverse transcript datasets"
- [abstract] Thousands of unannotated isoforms motivate the novel-peptide search: "Applying this approach to neural cell-surface molecules, we identify thousands of unannotated isoforms expressed in retina and brain."
