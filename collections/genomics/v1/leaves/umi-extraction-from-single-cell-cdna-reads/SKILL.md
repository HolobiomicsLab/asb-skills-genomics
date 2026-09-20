---
name: umi-extraction-from-single-cell-cdna-reads
description: Use when processing long reads from 10x Genomics 5'-anchored whole-transcriptome
  cDNA libraries and per-read cell-barcode and UMI tags are needed before downstream
  mutation or isoform calling.
when_to_use_negative:
- Input is short-read 10x Read 1 data where barcodes/UMIs are provided directly in
  the read header rather than embedded in long cDNA softclips
- The library is not 5'-anchored 10x Genomics chemistry, since the barcode-UMI motif
  search assumes the barcode-UMI cassette lies in the 5' softclip
- UMIs have a known whitelist available — in that case whitelist-based correction,
  not starcode distance-3 clustering, is appropriate
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: nanoranger
  role: Runs the edlib barcode-UMI motif search, N-padded whitelist construction,
    STAR barcode assignment, and UMI extraction from the 10 nt following the barcode
  repo: github.com/mehdiborji/nanoranger
- name: STAR
  role: Aligns candidate barcode-UMI sequences end-to-end against the N-padded 10x
    barcode whitelist
  repo: github.com/alexdobin/STAR
- name: starcode
  role: Collapses sequencing-error-derived UMIs into groups within a Levenshtein distance
    of 3, since 10x UMIs have no correction whitelist
  repo: https://github.com/gui11aume/starcode
- name: edlib
  role: Levenshtein-distance motif search over 5' softclips to locate candidate barcode-UMI
    regions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:3c79674ff7509059836eb596f5ee99c7cf7f81eac679cb7d34cd033a827b30e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Genomics
  - Pacific Biosciences
  - STAR
  - edlib
  - nanoranger
  - starcode
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# umi-extraction-from-single-cell-cdna-reads

## Summary

Assigns 10x Genomics cell barcodes and extracts 10 nt UMIs from long-read single-cell cDNA sequences (ONT/PacBio) via N-padded-whitelist STAR alignment, then corrects UMI sequencing error downstream by starcode clustering.

## When to use

Use when processing long reads from 10x Genomics 5'-anchored whole-transcriptome cDNA libraries and per-read cell-barcode and UMI tags are needed before downstream mutation or isoform calling.

## When NOT to use

- Input is short-read 10x Read 1 data where barcodes/UMIs are provided directly in the read header rather than embedded in long cDNA softclips
- The library is not 5'-anchored 10x Genomics chemistry, since the barcode-UMI motif search assumes the barcode-UMI cassette lies in the 5' softclip
- UMIs have a known whitelist available — in that case whitelist-based correction, not starcode distance-3 clustering, is appropriate

## Inputs

- Long reads (FASTQ/BAM) sequenced from 10x Genomics 5'-anchored whole-transcriptome cDNA libraries (ONT or PacBio)
- cDNA alignments with 5' softclips retained
- 10x Genomics 5' cell-barcode whitelist

## Outputs

- barcode_umi_assignments.tsv with per-read cell-barcode and UMI assignments
- UMI clusters (starcode groups) collapsing sequencing-error-derived UMIs within Levenshtein distance 3

## How to apply

Retain long reads with 5' softclips from cDNA alignments, then run an edlib Levenshtein-distance motif search over each 5' softclip (maximum edit distance 5, restricted to a 200 nt window) to locate candidate barcode-UMI regions. Build an N-padded version of the 10x Genomics 5' cell-barcode whitelist so adapter and UMI sequences kept in the candidate reads are not penalized during end-to-end alignment against reference barcodes with STAR. Accept an assigned barcode when the end-to-end alignment contains at most 1 mismatch or indel, and extract the 10 nt immediately following the barcode as the UMI; write assignments to barcode_umi_assignments.tsv. Because 10x UMIs, unlike cell barcodes, have no whitelist for error correction, collapse UMI copies arising from sequencing error with starcode into groups within a Levenshtein distance of 3.

## Related tools

- **nanoranger** (Runs the edlib barcode-UMI motif search, N-padded whitelist construction, STAR barcode assignment, and UMI extraction from the 10 nt following the barcode) — github.com/mehdiborji/nanoranger
- **STAR** (Aligns candidate barcode-UMI sequences end-to-end against the N-padded 10x barcode whitelist) — github.com/alexdobin/STAR
- **starcode** (Collapses sequencing-error-derived UMIs into groups within a Levenshtein distance of 3, since 10x UMIs have no correction whitelist) — https://github.com/gui11aume/starcode
- **edlib** (Levenshtein-distance motif search over 5' softclips to locate candidate barcode-UMI regions)

## Evaluation signals

- Every accepted barcode assignment has an end-to-end STAR alignment with at most 1 mismatch or indel
- Each extracted UMI is exactly 10 nt and sits immediately after the assigned barcode in the read
- barcode_umi_assignments.tsv contains one assignment row per processed read with both CB and UMI fields populated
- Post-starcode, UMI groups converge within Levenshtein distance 3, removing singleton UMIs attributable to ONT/PacBio sequencing error
- Barcode candidate reads with adapter/UMI flanking sequence should still align fully because the whitelist is N-padded (no penalty at flanks)

## Limitations

- UMIs in 10x Genomics libraries lack a whitelist, so sequencing-error UMIs cannot be directly corrected and require approximate clustering (starcode distance 3), which may merge or split true UMIs at high error rates
- Barcode assignment tolerates only at most 1 mismatch or indel, so reads with more heavily corrupted barcodes are discarded
- The motif search is restricted to a 200 nt window within the 5' softclip and a maximum edit distance of 5; reads whose barcode-UMI cassette falls outside these limits are missed

## Evidence

- [methods] Accept an assigned barcode when the end-to-end alignment contains at most 1 mismatch or indel, and extract the 10 nt immediately following the barcode as the UMI.: [evidence span withheld — anchor status: not_found]
- [methods] nanoranger pads the whitelist of known barcodes with unknown nucleotides so that the adapter and UMI sequences kept in barcode candidate reads are not penalized during alignment against the reference barcodes.: [evidence span withheld — anchor status: not_found]
- [readme] UMIs that arise due to sequencing error are collapsed in groups within a Levenshtein distance of 3.: "By using [starcode](https://github.com/gui11aume/starcode), UMIs that arise due to sequencing error are collapsed
groups within a Levenshtein distance of 3"
- [other] Run an edlib Levenshtein-distance motif search over each 5' softclip (maximum edit distance 5, restricted to a 200 nt window) to locate candidate barcode-UMI motif regions.: [evidence span withheld — anchor status: not_found]
