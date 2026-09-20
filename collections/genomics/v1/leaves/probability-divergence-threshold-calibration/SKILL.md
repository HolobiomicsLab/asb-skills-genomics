---
name: probability-divergence-threshold-calibration
description: Use when generating a PRO barcode kit and you must translate a target
  edit-error tolerance (e.g., 2 errors in a 24-mer) into the probability-divergence
  threshold t that governs candidate removal and barcode selection.
when_to_use_negative:
- The task is demultiplexing already-sequenced reads with Guppy official configs (barcode_arrs_nb12.cfg,
  barcode_arrs_16s.cfg, barcode_arrs_pcr96.cfg) — no PRO threshold calibration is
  involved.
- The candidate set has not been preprocessed (64-bit hashing and flanking-similarity
  removal with edlib, edit distance <= 1), so the threshold would be applied over
  an unnecessarily large 4^n space.
- The platform is PacBio — the article states no tests on PacBio data have been performed,
  so the ONT-calibrated threshold may not transfer.
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: PRO
  role: Package whose probability-divergence measure defines the threshold and whose
    pro_FPS_select.py applies it during farthest-point-sampling barcode selection
  repo: https://github.com/rztongr/PRO
- name: edlib 1.3.9
  role: Computes semiglobal-alignment edit distances used to remove candidates too
    similar to flanking sequences before thresholded selection
  repo: https://github.com/renztr/PRO.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.fmre.2024.04.014
    title: Generating barcodes for nanopore sequencing data with PRO
  source_content_hash: sha256:5d0b95fd056662c6211b01d11282cb6dfdf1e8b424487c750369441637eca135
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/rztongr/PRO
  repo_url: https://github.com/rztongr/PRO
  tools:
  - Anaconda3
  - PRO
  - Python
  - edlib 1.3.9
  - tqdm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# probability-divergence-threshold-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calibrates PRO's probability-divergence threshold t used during barcode generation by converting the number of edit errors a 24 bp barcode must tolerate (e.g., -th 2 at sequencing accuracy -a 0.88) into ProDiv units, then filtering the candidate k-mer space accordingly before farthest-point sampling.

## When to use

Use when generating a PRO barcode kit and you must translate a target edit-error tolerance (e.g., 2 errors in a 24-mer) into the probability-divergence threshold t that governs candidate removal and barcode selection.

## When NOT to use

- The task is demultiplexing already-sequenced reads with Guppy official configs (barcode_arrs_nb12.cfg, barcode_arrs_16s.cfg, barcode_arrs_pcr96.cfg) — no PRO threshold calibration is involved.
- The candidate set has not been preprocessed (64-bit hashing and flanking-similarity removal with edlib, edit distance <= 1), so the threshold would be applied over an unnecessarily large 4^n space.
- The platform is PacBio — the article states no tests on PacBio data have been performed, so the ONT-calibrated threshold may not transfer.

## Inputs

- Target barcode length (-l, e.g., 24 bp matching official ONT barcodes)
- Edit-error tolerance (-th, e.g., 2 errors a barcode must survive)
- Sequencing accuracy/error rate (-a, e.g., 0.88)
- Preselected candidate 24-mer set from the 4^24 space (after 64-bit hashing and binning)
- Flanking-sequence FASTQ used in library preparation (-f)

## Outputs

- Calibrated probability-divergence threshold t
- Filtered candidate set (candidates with ProDiv(s, seq1) > t removed)
- pre_processing_barcodes.txt and pre_processing_barcodes_number.txt (preprocessed sequences and hashed integers)
- select_barcode.txt containing the PRO-generated barcode kit (e.g., exactly 96 barcodes of 24 bp)

## How to apply

First fix the design parameters: barcode length (-l, e.g., 24 bp), the number of edit errors the barcode should tolerate (-th, e.g., 2), and the sequencing error/accuracy assumption (-a, e.g., 0.88). Compute the threshold t by converting the number of errors expected to be correctly demultiplexed into probability divergence, using PRO's divergence measure ProDiv(X,Y) built from the probabilistic model of mutations (substitutions/insertions/deletions at ps and pi rates). Apply t as a filter: any candidate sequence s with ProDiv(s, seq1) > t is removed from the candidate set to cut the calculation load, alongside the separate pre-filter that drops candidates whose semiglobal-alignment edit distance to both flanking sequences is <= 1 (computed with edlib). Then run pro_FPS_select.py with -l, -th, -a, the flanking FASTQ (-f) and the preselected size (-q, e.g., 96 for matching the official ONT kit size) so farthest-point sampling emits select_barcode.txt.

## Related tools

- **PRO** (Package whose probability-divergence measure defines the threshold and whose pro_FPS_select.py applies it during farthest-point-sampling barcode selection) — https://github.com/rztongr/PRO
- **edlib 1.3.9** (Computes semiglobal-alignment edit distances used to remove candidates too similar to flanking sequences before thresholded selection) — https://github.com/renztr/PRO.git

## Examples

```
python pro_FPS_select.py -l 24 -th 2 -a 0.88 -f flanking.fastq -q 100000 -o examples/testoutput/k_mer_select
```

## Evaluation signals

- The threshold t is traceable to the intended error count: running with -th 2 and -a 0.88 yields a t that lets barcodes survive two edit errors in simulated demultiplexing.
- All candidates with ProDiv(s, seq1) > t are absent from the working set, and the candidate count is visibly reduced relative to the full preselected 4^24-derived space.
- pre_processing_barcodes.txt and pre_processing_barcodes_number.txt exist with matching counts of sequences and 64-bit hash integers.
- select_barcode.txt contains exactly the requested number (-q) of barcodes at the specified length (-l, e.g., 24 bp), with pairwise probability divergence above t.
- Downstream validation: simulated ONT reads carrying the kit (96 barcodes, 24 bp) demultiplex with PRO at accuracy comparable to or above Guppy 6.5.7, as in the task_004 comparison (~4.34% higher on average).

## Limitations

- The threshold is calibrated to ONT-like error models (accuracy ~0.88, deepsimulator 1.5 data); no PacBio validation exists.
- Finding the optimal largest barcode set in the equal-length sequence space is NP-complete, so the FPS selection under this threshold is heuristic, not globally optimal.
- Candidates similar to flanking sequences (edit distance <= 1 to both flanks) cannot be accurately extracted during demultiplexing, which contributed to PRO underperforming Guppy on official kits B1 (EXP-NBD104), B2 (SQK-16S024), and B3 (EXP-PBC096).

## Evidence
