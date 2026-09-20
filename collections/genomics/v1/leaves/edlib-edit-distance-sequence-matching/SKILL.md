---
name: edlib-edit-distance-sequence-matching
description: Use when assigning cell barcodes to ONT/PacBio single-cell reads from
  10x Genomics 5'-anchored cDNA libraries, where sequencing error and variable barcode/UMI
  placement demand error-tolerant motif search before whitelist alignment.
when_to_use_negative:
- Data are short-read (Illumina) 10x libraries where barcode and UMI sit in a fixed
  R1 position — direct whitelist matching without softclip motif search suffices
- The goal is downstream UMI deduplication rather than barcode detection — for that,
  nanoranger.R collapses UMIs with starcode at Levenshtein distance 3 since UMIs have
  no whitelist
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: nanoranger
  role: Pipeline embedding the edlib motif search; locates barcode-UMI motifs in 5'
    softclips and assigns barcodes/UMIs
  repo: github.com/mehdiborji/nanoranger
- name: edlib
  role: Levenshtein-distance approximate string matching used to find barcode-UMI
    motifs at maximum edit distance 5 within a 200 nt window
- name: STAR
  role: End-to-end alignment of candidate barcode-UMI sequences against the N-padded
    10x whitelist (≤1 mismatch/indel to accept)
  repo: github.com/alexdobin/STAR
- name: 10x Genomics
  role: Source chemistry whose 5' cell-barcode whitelist and barcode-UMI structure
    define the motif being searched
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:f5298296d67c37f5d84e1a41d61d0b5dfb9869f82ec894526b0e7e7a4cbdbf90
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Genomics
  - Pacific Biosciences
  - STAR
  - edlib
  - nanoranger
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# edlib-edit-distance-sequence-matching

## Summary

Use edlib's Levenshtein-distance approximate matching to locate 10x Genomics barcode-UMI motifs in long-read 5' softclips, tolerating the elevated error rate of ONT/PacBio reads. It is the motif-detection front end of the nanoranger cell-barcode assignment pipeline.

## When to use

Use when assigning cell barcodes to ONT/PacBio single-cell reads from 10x Genomics 5'-anchored cDNA libraries, where sequencing error and variable barcode/UMI placement demand error-tolerant motif search before whitelist alignment.

## When NOT to use

- Data are short-read (Illumina) 10x libraries where barcode and UMI sit in a fixed R1 position — direct whitelist matching without softclip motif search suffices
- The goal is downstream UMI deduplication rather than barcode detection — for that, nanoranger.R collapses UMIs with starcode at Levenshtein distance 3 since UMIs have no whitelist

## Inputs

- Long reads (FASTQ/BAM with cDNA alignments) from 10x Genomics 5'-anchored whole-transcriptome cDNA libraries sequenced on ONT or PacBio devices
- 5' softclip sequences from cDNA-aligned reads
- 10x Genomics 5' cell-barcode whitelist

## Outputs

- Candidate barcode-UMI motif regions per read
- Per-read barcode and UMI assignments written to barcode_umi_assignments.tsv

## How to apply

Start from long reads sequenced from 10x Genomics 5'-anchored whole-transcriptome cDNA libraries that have been cDNA-aligned, retaining reads with 5' softclips. Run an edlib Levenshtein-distance motif search over each 5' softclip with a maximum edit distance of 5, restricted to a 200 nt window, to locate candidate barcode-UMI motif regions. This edit-distance tolerance is needed because ONT error is far above the <1/1000 rate of Illumina reads and barcode/UMI position varies per transcript. Pass the extracted candidate barcode-UMI sequences to STAR for end-to-end alignment against an N-padded version of the 10x whitelist (padding prevents adapter/UMI bases from being penalized), accepting barcodes with at most 1 mismatch or indel, and extract the 10 nt following the barcode as the UMI.

## Related tools

- **nanoranger** (Pipeline embedding the edlib motif search; locates barcode-UMI motifs in 5' softclips and assigns barcodes/UMIs) — github.com/mehdiborji/nanoranger
- **edlib** (Levenshtein-distance approximate string matching used to find barcode-UMI motifs at maximum edit distance 5 within a 200 nt window)
- **STAR** (End-to-end alignment of candidate barcode-UMI sequences against the N-padded 10x whitelist (≤1 mismatch/indel to accept)) — github.com/alexdobin/STAR
- **10x Genomics** (Source chemistry whose 5' cell-barcode whitelist and barcode-UMI structure define the motif being searched)

## Evaluation signals

- Accepted barcode assignments show end-to-end alignment against the N-padded whitelist with at most 1 mismatch or indel
- UMI extracted as exactly the 10 nt immediately following the assigned barcode
- barcode_umi_assignments.tsv contains one barcode/UMI pair per processed candidate read
- Motif search restricted to 5' softclips of cDNA-aligned reads, within a 200 nt window and edit distance ≤5
- False barcode assignments drop when comparing ONT error profiles against the known whitelist size (≈737k 10x 5' barcodes)

## Limitations

- Maximum motif edit distance of 5 may miss barcode motifs in reads with clustered errors beyond that tolerance
- Motif search is restricted to 5' softclips of 5'-anchored 10x libraries; reads lacking softclips or from non-5'-anchored chemistries are out of scope
- Barcodes are whitelist-corrected, but UMIs have no whitelist and require separate error collapsing (starcode, Levenshtein distance 3 in nanoranger.R)

## Evidence

- [methods] edlib motif search parameters: [evidence span withheld — anchor status: not_found]
- [methods] N-padded whitelist rationale: [evidence span withheld — anchor status: not_found]
- [readme] UMI error collapsing downstream: [evidence span withheld — anchor status: not_found]
- [methods] Barcode acceptance criterion: [evidence span withheld — anchor status: not_found]
