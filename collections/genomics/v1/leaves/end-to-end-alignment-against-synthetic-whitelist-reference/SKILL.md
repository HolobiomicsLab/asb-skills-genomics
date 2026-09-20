---
name: end-to-end-alignment-against-synthetic-whitelist-reference
description: Use when assigning 10x cell barcodes to noisy ONT/PacBio single-cell
  long reads whose barcode and UMI positions vary within the 5' softclip, and a whitelist-based
  exact match is infeasible.
when_to_use_negative:
- Reads lack 5' softclips or a detectable barcode-UMI motif (edlib search fails within
  the 200 nt window / max edit distance 5), so no barcode candidate exists to align
- Data are short-read 10x libraries where Read 1 barcode/UMI structure is fixed and
  standard Cell Ranger whitelisting suffices
- The protocol does not use a 10x Genomics whitelist (e.g. custom barcodes without
  a known reference set)
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: nanoranger
  role: Pipeline that performs the edlib motif search, builds the N-padded whitelist,
    and orchestrates the STAR barcode alignment
  repo: github.com/mehdiborji/nanoranger
- name: STAR
  role: Performs the end-to-end alignment of candidate barcode-UMI sequences against
    the N-padded whitelist reference
  repo: https://github.com/alexdobin/STAR
- name: 10x Genomics
  role: Source of the 5' cell-barcode whitelist and of the whole-transcriptome cDNA
    libraries being barcoded
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:7a4688b8281db11bd41f22676ed02deca9ec7c5213c9da25f81caa5c7434a833
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Genomics
  - Pacific Biosciences
  - STAR
  - nanoranger
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# end-to-end-alignment-against-synthetic-whitelist-reference

## Summary

A nanoranger barcode-calling strategy that pads the 10x Genomics 5' cell-barcode whitelist with unknown (N) nucleotides and aligns candidate barcode-UMI reads end-to-end with STAR, so adapter and UMI sequence in the query is not penalized against ONT sequencing error.

## When to use

Use when assigning 10x cell barcodes to noisy ONT/PacBio single-cell long reads whose barcode and UMI positions vary within the 5' softclip, and a whitelist-based exact match is infeasible.

## When NOT to use

- Reads lack 5' softclips or a detectable barcode-UMI motif (edlib search fails within the 200 nt window / max edit distance 5), so no barcode candidate exists to align
- Data are short-read 10x libraries where Read 1 barcode/UMI structure is fixed and standard Cell Ranger whitelisting suffices
- The protocol does not use a 10x Genomics whitelist (e.g. custom barcodes without a known reference set)

## Inputs

- BAM/alignments of ONT or PacBio long reads from 10x Genomics 5'-anchored whole-transcriptome cDNA libraries, retaining reads with 5' softclips
- 10x Genomics 5' cell-barcode whitelist (whitelist of known barcodes)
- Candidate barcode-UMI motif regions extracted from 5' softclips

## Outputs

- Per-read cell barcode and UMI assignments written to barcode_umi_assignments.tsv
- N-padded whitelist FASTA used as the STAR reference

## How to apply

First locate candidate barcode-UMI motif regions by running an edlib Levenshtein-distance search over each read's 5' softclip (maximum edit distance 5, restricted to a 200 nt window). Build an N-padded version of the 10x Genomics 5' cell-barcode whitelist so the adapter and UMI sequences retained in candidate reads are not penalized during alignment. Align the candidate barcode-UMI sequences end-to-end against this synthetic reference with STAR, then accept an assigned barcode only when the end-to-end alignment contains at most 1 mismatch or indel; extract the 10 nt immediately following the barcode as the UMI. The rationale is that padding absorbs the variable-length flanking sequence so the 16 nt barcode itself drives the alignment, tolerating ONT's higher error rate via the ≤1 edit threshold.

## Related tools

- **nanoranger** (Pipeline that performs the edlib motif search, builds the N-padded whitelist, and orchestrates the STAR barcode alignment) — github.com/mehdiborji/nanoranger
- **STAR** (Performs the end-to-end alignment of candidate barcode-UMI sequences against the N-padded whitelist reference) — https://github.com/alexdobin/STAR
- **10x Genomics** (Source of the 5' cell-barcode whitelist and of the whole-transcriptome cDNA libraries being barcoded)

## Examples

```
STAR --genomeDir padded_10x_whitelist_index --readFilesIn barcode_candidates.fa --outSAMtype None --outFilterMismatchNmax 1
```

## Evaluation signals

- Accepted barcode assignments have end-to-end alignments with ≤1 mismatch or indel against the N-padded whitelist reference
- Every accepted barcode call yields a 10 nt UMI extracted immediately downstream of the barcode match
- barcode_umi_assignments.tsv is produced with one row per processed candidate read
- Candidate barcode-UMI regions were found within the edlib constraints (max edit distance 5, 200 nt window on the 5' softclip)
- Whitelist barcodes recovered match entries of the 10x Genomics 5' whitelist (no novel barcodes accepted)

## Limitations

- Assignment tolerates at most 1 mismatch or indel, so reads with more sequencing errors in the barcode are discarded and lost as cells
- Requires 5' softclips containing the barcode-UMI motif; transcripts whose barcode region is absent or too degraded cannot be assigned
- UMI correction relies on downstream collapsing (e.g. starcode Levenshtein distance 3 in nanoranger.R) because UMIs have no whitelist for error correction

## Evidence

- [methods] N-padded whitelist absorbs adapter/UMI flanks: [evidence span withheld — anchor status: not_found]
- [methods] STAR end-to-end acceptance rule: [evidence span withheld — anchor status: not_found]
- [methods] edlib motif search parameters: [evidence span withheld — anchor status: not_found]
- [methods] UMI extraction rule: [evidence span withheld — anchor status: not_found]
- [readme] UMI error collapsing rationale: [evidence span withheld — anchor status: external_doc]
