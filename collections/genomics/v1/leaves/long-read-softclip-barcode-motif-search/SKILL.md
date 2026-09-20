---
name: long-read-softclip-barcode-motif-search
description: Use when assigning 10x Genomics cell barcodes and UMIs to long ONT/PacBio
  reads from 5'-anchored whole-transcriptome cDNA libraries whose cDNA alignments
  retain 5' softclips, as in nanoranger genotyping.
when_to_use_negative:
- The library is not 5'-anchored 10x Genomics chemistry, so barcodes/UMIs are not
  at the 5' end and no 5' softclip is available for motif search
- Short-read scRNA-seq data with matched cell barcodes already corrected against the
  whitelist — this skill targets long reads with sequencing-error-prone barcodes
- Reads have no 5' softclips in their cDNA alignments (fully end-to-end aligned),
  leaving no candidate barcode-UMI region to search
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: nanoranger
  role: Implements the edlib softclip motif search, N-padded whitelist construction,
    STAR barcode alignment, and barcode_umi_assignments.tsv output
  repo: github.com/mehdiborji/nanoranger
- name: edlib
  role: Levenshtein-distance motif search over each 5' softclip to locate candidate
    barcode-UMI regions
- name: STAR
  role: End-to-end alignment of candidate barcode-UMI sequences against the N-padded
    10x barcode whitelist
  repo: https://github.com/alexdobin/STAR
- name: starcode
  role: Downstream collapse of UMIs within a Levenshtein distance of 3, since UMIs
    (unlike cell barcodes) lack a whitelist for error correction
  repo: https://github.com/gui11aume/starcode
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:07f26d9a9b8eae756b8a13f00c0a8b992edba747aed1ae323bd3028a885996a0
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# long-read-softclip-barcode-motif-search

## Summary

A nanoranger skill that locates 10x Genomics barcode-UMI motifs in the 5' softclips of long ONT/PacBio reads via edlib Levenshtein-distance search, then assigns barcodes by STAR end-to-end alignment against an N-padded whitelist.

## When to use

Use when assigning 10x Genomics cell barcodes and UMIs to long ONT/PacBio reads from 5'-anchored whole-transcriptome cDNA libraries whose cDNA alignments retain 5' softclips, as in nanoranger genotyping.

## When NOT to use

- The library is not 5'-anchored 10x Genomics chemistry, so barcodes/UMIs are not at the 5' end and no 5' softclip is available for motif search
- Short-read scRNA-seq data with matched cell barcodes already corrected against the whitelist — this skill targets long reads with sequencing-error-prone barcodes
- Reads have no 5' softclips in their cDNA alignments (fully end-to-end aligned), leaving no candidate barcode-UMI region to search

## Inputs

- Long reads (FASTQ/BAM) from ONT or Pacific Biosciences sequencing of 10x Genomics 5'-anchored whole-transcriptome cDNA libraries or targeted amplicons
- cDNA alignments (BAM) retaining 5' softclips
- 10x Genomics 5' cell-barcode whitelist (e.g. 737K-august-2016.txt)

## Outputs

- barcode_umi_assignments.tsv with per-read barcode and UMI assignments

## How to apply

Load long reads from 10x Genomics 5'-anchored whole-transcriptome cDNA libraries together with their cDNA alignments, retaining reads with 5' softclips. Run an edlib Levenshtein-distance motif search over each 5' softclip (maximum edit distance 5, restricted to a 200 nt window) to locate candidate barcode-UMI motif regions. Build an N-padded version of the 10x Genomics 5' cell-barcode whitelist — padding with unknown nucleotides prevents adapter and UMI sequences retained in the candidates from being penalized — and align candidates end-to-end against it with STAR. Accept a barcode assignment only when the end-to-end alignment contains at most 1 mismatch or indel, then extract the 10 nt immediately following the barcode as the UMI and write per-read assignments to barcode_umi_assignments.tsv.

## Related tools

- **nanoranger** (Implements the edlib softclip motif search, N-padded whitelist construction, STAR barcode alignment, and barcode_umi_assignments.tsv output) — github.com/mehdiborji/nanoranger
- **edlib** (Levenshtein-distance motif search over each 5' softclip to locate candidate barcode-UMI regions)
- **STAR** (End-to-end alignment of candidate barcode-UMI sequences against the N-padded 10x barcode whitelist) — https://github.com/alexdobin/STAR
- **starcode** (Downstream collapse of UMIs within a Levenshtein distance of 3, since UMIs (unlike cell barcodes) lack a whitelist for error correction) — https://github.com/gui11aume/starcode

## Examples

```
starcode -d 3 umis.fasta > umi_clusters.tsv  # collapse 10 nt UMIs by Levenshtein distance 3 after nanoranger barcode assignment
```

## Evaluation signals

- Every accepted barcode alignment contains at most 1 mismatch or indel in the end-to-end alignment against the N-padded whitelist
- Each extracted UMI is exactly 10 nt and located immediately 3' of the assigned barcode in the candidate sequence
- barcode_umi_assignments.tsv contains one row per processed read with non-empty barcode/UMI fields for accepted reads
- Candidate motif regions fall within the 5' softclip and the search was restricted to a 200 nt window with maximum edit distance 5
- Downstream starcode clustering collapses UMI groups within a Levenshtein distance of 3

## Limitations

- Barcode assignment is capped at 1 mismatch/indel, so heavily errored ONT barcodes may be rejected; the 10x whitelist cannot correct UMIs, which instead require distance-3 starcode collapse
- The N-padded whitelist absorbs adjacent adapter/UMI sequence, but assignments depend on the motif search succeeding within the 200 nt softclip window
- Designed for 5'-anchored 10x Genomics whole-transcriptome cDNA libraries or targeted amplicons from those libraries; variable barcode/UMI location must still be resolved by the softclip motif search

## Evidence

- [methods] edlib softclip motif search: [evidence span withheld — anchor status: not_found]
- [methods] N-padded whitelist rationale: [evidence span withheld — anchor status: not_found]
- [methods] STAR assignment rule: [evidence span withheld — anchor status: not_found]
- [readme] UMI collapse rationale: [evidence span withheld — anchor status: not_found]
- [readme] input libraries and platforms: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification, with Oxford Nanopore Technologies (ONT) or"
