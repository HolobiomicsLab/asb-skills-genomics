---
name: sequencing-error-threshold-calibration
description: Use when generating error-tolerant DNA barcode kits for nanopore sequencing
  and you need to translate a target sequencing error rate and tolerated error count
  into PRO's probability-divergence threshold t.
when_to_use_negative:
- Input is already a validated barcode kit (e.g., official ONT kits EXP-NBD104, SQK-16S024,
  EXP-PBC096) — demultiplex with PRO-dex instead of calibrating a generation threshold
- The sequencing platform is PacBio with a different error profile — the article states
  no tests on PacBio data have been performed, so the error-rate conversion is unvalidated
  there
- Flanking sequences used in library preparation are similar to candidate barcodes
  — threshold calibration does not fix extraction failures; such candidates must be
  removed by semiglobal-alignment edit distance <= 1 before generation
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
tools:
- name: PRO
  role: Implements the threshold determination and uses t during farthest-point sampling
    barcode generation (pro_FPS_select.py with -th and -a parameters)
  repo: https://github.com/renztr/PRO.git
- name: Python
  role: Interpreter (>=3.7.0) in which the threshold conversion and probability-divergence
    computation are executed
- name: tqdm
  role: Progress tracking for the iterative threshold-driven candidate filtering loop
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.fmre.2024.04.014
    title: Generating barcodes for nanopore sequencing data with PRO
  source_content_hash: sha256:115b59b44bd5dd5d5b05794ba1ea54fc423534b2bda6f1795622fd0c162e02ea
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/rztongr/PRO
  repo_url: https://github.com/renztr/PRO.git
  tools:
  - PRO
  - Python
  - tqdm
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# sequencing-error-threshold-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calibrates the probability-divergence threshold t used by PRO's farthest-point sampling by converting the nanopore sequencing error rate p and the number of errors k the kit must tolerate into a divergence cutoff for barcode candidate filtering.

## When to use

Use when generating error-tolerant DNA barcode kits for nanopore sequencing and you need to translate a target sequencing error rate and tolerated error count into PRO's probability-divergence threshold t.

## When NOT to use

- Input is already a validated barcode kit (e.g., official ONT kits EXP-NBD104, SQK-16S024, EXP-PBC096) — demultiplex with PRO-dex instead of calibrating a generation threshold
- The sequencing platform is PacBio with a different error profile — the article states no tests on PacBio data have been performed, so the error-rate conversion is unvalidated there
- Flanking sequences used in library preparation are similar to candidate barcodes — threshold calibration does not fix extraction failures; such candidates must be removed by semiglobal-alignment edit distance <= 1 before generation

## Inputs

- Target barcode length n (e.g., 24 nt, matching official ONT barcodes)
- Nanopore sequencing error rate p (e.g., p=0.12 passed as -a 0.88)
- Number of errors k the kit should tolerate (e.g., -th 2)
- Preselected candidate DNA sequences from the 4^n k-mer space (e.g., q=100,000 after 64-bit hash preselection)

## Outputs

- Probability-divergence threshold t governing barcode acceptance during farthest-point sampling
- Filtering decision: discard candidate s whenever ProDiv(s, seq1) > t
- select_barcode.txt containing the generated barcode kit whose maximum pairwise probability divergence is <= t

## How to apply

Start from the target barcode length n (e.g., 24 nt for ONT official barcodes) and the sequencing error rate p. Convert p into the insertion-worst-case probability p' = (1-p)^n * (1/4 * pi)^k, where k is the number of errors the kit should tolerate and pi is the substitution probability term from PRO's probability divergence definition ProDiv(X,Y)=pmmpdd(13ps)s(14pi)i. Derive the threshold t such that any two selected barcodes have pairwise probability-divergence weight <= t, guaranteeing correct demultiplexing of up to k errors. Pass t (via -th for tolerated errors and -a for accuracy/error rate) into pro_FPS_select.py; during farthest-point sampling, candidates s with ProDiv(s, seq1) > t are discarded to shrink the candidate set. The rationale is that probability divergence is a more accurate dissimilarity measure than edit distance for the high nanopore error rate, so the threshold directly encodes the error model rather than an edit-distance proxy.

## Related tools

- **PRO** (Implements the threshold determination and uses t during farthest-point sampling barcode generation (pro_FPS_select.py with -th and -a parameters)) — https://github.com/renztr/PRO.git
- **Python** (Interpreter (>=3.7.0) in which the threshold conversion and probability-divergence computation are executed)
- **tqdm** (Progress tracking for the iterative threshold-driven candidate filtering loop)

## Examples

```
python pro_FPS_select.py -l 24 -th 2 -a 0.88 -o output/k_mer_select -q 100000
```

## Evaluation signals

- Every pair of barcodes in the output select_barcode.txt has maximum pairwise probability-divergence weight <= t
- All emitted barcode sequences have the requested length n (e.g., 24 nt) and the kit reaches the expected maximum size (2,292 for n=24, seed 0)
- Pre-processed candidate count matches the -q preselection setting (e.g., 100,000 candidates) and candidates with ProDiv > t are absent from the final kit
- Simulated demultiplexing of the kit (e.g., via DeepSimulator 1.5 data) achieves the targeted error tolerance, with reported accuracy 98.29% for large kits and comparable-to-Guppy accuracy at official kit sizes

## Limitations

- The threshold is calibrated for ONT's high error rate; it does not address PRO's failure when flanking sequences are similar to barcodes (e.g., B1–B3), which degrades demultiplexing regardless of t
- The insertion-worst-case conversion assumes equal substitution probabilities (1/4 * pi)^k term; ONT's design strategy differs and does not ensure large probability divergence between its official barcodes
- Validated only on ONT simulations (deepsimulator 1.5, Guppy 6.5.7 comparisons); no PacBio validation exists

## Evidence
