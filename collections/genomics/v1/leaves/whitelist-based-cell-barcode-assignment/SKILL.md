---
name: whitelist-based-cell-barcode-assignment
description: Use when processing long reads (ONT/PacBio) from 5'-anchored 10x Genomics
  scRNA-seq cDNA libraries and you must recover per-read cell barcodes and 10 nt UMIs
  despite sequencing error and variable barcode/UMI positions.
when_to_use_negative:
- Input reads lack 5' softclips (e.g. 3'-anchored 10x libraries) so no barcode-UMI
  motif can be found within the 200 nt window
- Data are short-read Illumina 10x libraries where standard Cell Ranger barcode correction,
  not long-read alignment, is appropriate
- UMI-level error correction is the goal — UMIs have no whitelist and are instead
  collapsed downstream (e.g. starcode at Levenshtein distance 3)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: nanoranger
  role: Implements the softclip motif search (edlib), N-padded whitelist construction,
    STAR alignment, barcode/UMI extraction, and TSV output
  repo: github.com/mehdiborji/nanoranger
- name: STAR
  role: Performs end-to-end alignment of candidate barcode-UMI sequences against the
    N-padded barcode whitelist
  repo: github.com/alexdobin/STAR
- name: starcode
  role: Downstream UMI error correction in nanoranger.R by collapsing UMIs within
    Levenshtein distance 3 (UMIs lack a whitelist)
  repo: github.com/liviuspenter/nanoranger.R
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:8065d3a68564cb6c6efccfebd26436268e5989fac6b730cdcd680781e902708d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Genomics
  - Pacific Biosciences
  - STAR
  - nanoranger
  - starcode
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# whitelist-based-cell-barcode-assignment

## Summary

Assign cell barcodes and UMIs to ONT/PacBio single-cell long reads by end-to-end STAR alignment of 5'-softclip barcode-UMI candidates against an N-padded 10x Genomics barcode whitelist, tolerating ONT's higher sequencing error.

## When to use

Use when processing long reads (ONT/PacBio) from 5'-anchored 10x Genomics scRNA-seq cDNA libraries and you must recover per-read cell barcodes and 10 nt UMIs despite sequencing error and variable barcode/UMI positions.

## When NOT to use

- Input reads lack 5' softclips (e.g. 3'-anchored 10x libraries) so no barcode-UMI motif can be found within the 200 nt window
- Data are short-read Illumina 10x libraries where standard Cell Ranger barcode correction, not long-read alignment, is appropriate
- UMI-level error correction is the goal — UMIs have no whitelist and are instead collapsed downstream (e.g. starcode at Levenshtein distance 3)

## Inputs

- Long reads from 10x Genomics 5'-anchored whole-transcriptome cDNA libraries (ONT or PacBio) FASTQ/BAM with cDNA alignments and 5' softclips
- 10x Genomics 5' cell-barcode whitelist

## Outputs

- barcode_umi_assignments.tsv with per-read barcode and UMI assignments

## How to apply

Load long reads with their cDNA alignments and retain reads carrying 5' softclips, since 10x Genomics 5'-anchored libraries place the barcode-UMI motif upstream of the cDNA. Run an edlib Levenshtein-distance motif search over each 5' softclip (maximum edit distance 5, restricted to a 200 nt window) to locate candidate barcode-UMI regions. Build an N-padded version of the 10x Genomics 5' cell-barcode whitelist so that residual adapter and UMI sequence kept in candidate reads is not penalized during alignment. Align candidates end-to-end against the padded whitelist with STAR, and accept a barcode assignment when the end-to-end alignment contains at most 1 mismatch or indel, extracting the 10 nt immediately following the barcode as the UMI. This addresses the challenge that, unlike UMIs, cell barcodes have a whitelist usable for correcting sequencing error.

## Related tools

- **nanoranger** (Implements the softclip motif search (edlib), N-padded whitelist construction, STAR alignment, barcode/UMI extraction, and TSV output) — github.com/mehdiborji/nanoranger
- **STAR** (Performs end-to-end alignment of candidate barcode-UMI sequences against the N-padded barcode whitelist) — github.com/alexdobin/STAR
- **starcode** (Downstream UMI error correction in nanoranger.R by collapsing UMIs within Levenshtein distance 3 (UMIs lack a whitelist)) — github.com/liviuspenter/nanoranger.R

## Evaluation signals

- Every accepted barcode alignment is end-to-end with at most 1 mismatch or indel against the N-padded whitelist
- Extracted UMI is exactly the 10 nt immediately following the assigned barcode in the candidate read
- Output barcode_umi_assignments.tsv contains one assignment per processed read with both CB and UMI fields
- Candidate motif calls fall within the 200 nt window of the 5' softclip and pass the edit-distance-≤5 threshold
- Reads without a detectable 5' softclip are excluded from barcode assignment rather than force-assigned

## Limitations

- Whitelist-based correction cannot recover barcodes absent from the 10x whitelist; a mismatch/indel-free requirement beyond 1 error rejects otherwise valid reads under high ONT error
- UMIs cannot be corrected via this whitelist strategy because no UMI whitelist exists; they are only collapsed downstream at Levenshtein distance 3
- Barcode position is variable within each transcript read, so fixed-offset extraction fails without the motif-search step

## Evidence

- [full_text] nanoranger pads the whitelist of known barcodes with unknown nucleotides so that the adapter and UMI sequences kept in barcode candidate reads are not penalized: [evidence span withheld — anchor status: not_found]
- [full_text] Align candidate barcode-UMI sequences against the N-padded whitelist with STAR and accept at most 1 mismatch or indel: [evidence span withheld — anchor status: not_found]
- [readme] The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification"
- [readme] By using starcode, UMIs that arise due to sequencing error are collapsed in groups within a Levenshtein distance of 3: [evidence span withheld — anchor status: not_found]
