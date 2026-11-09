---
name: dna-barcode-set-generation
description: Use when designing a new ONT barcode kit from an equal-length DNA sequence
  space (e.g. all 4^24 24-mers) and you need a maximal set of barcodes that remains
  correctly demultiplexable under a specified number of sequencing errors.
when_to_use_negative:
- The barcode kit already exists and the goal is demultiplexing reads — use PRO's
  demultiplexing mode (pro-dex) instead of generation
- Sequences in the candidate space are of unequal length — the NP-complete optimality
  formulation and FPS selection assume a space where all sequences have the same length
- Data are PacBio reads — the article notes no tests on PacBio data have been performed
  yet
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3168
tools:
- name: PRO
  role: Runs dataset preselection (64-bit hashing), threshold determination, and farthest-point
    sampling to emit the barcode kit into select_barcode.txt
  repo: https://github.com/renztr/PRO.git
- name: Python 3.7.0
  role: Interpreter required to execute pro_FPS_select.py
- name: edlib 1.3.9
  role: Edit-distance computation used in preprocessing/flanking-sequence filtering
    of candidates
- name: tqdm
  role: Progress reporting during the long candidate-iteration loops
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.fmre.2024.04.014
    title: Generating barcodes for nanopore sequencing data with PRO
  source_content_hash: sha256:a8c723d1fd3e3504243655222005f0166c11ab8e1a1523a8ab00c57f4acd84c5
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
  - Python 3.7.0
  - edlib 1.3.9
  - tqdm
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# dna-barcode-set-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate a maximum-size, error-tolerant DNA barcode kit for nanopore sequencing by running PRO's farthest-point sampling algorithm over a preselected k-mer sequence space, using probability divergence as the dissimilarity measure and a threshold t derived from the sequencing error rate p and tolerated errors k.

## When to use

Use when designing a new ONT barcode kit from an equal-length DNA sequence space (e.g. all 4^24 24-mers) and you need a maximal set of barcodes that remains correctly demultiplexable under a specified number of sequencing errors.

## When NOT to use

- The barcode kit already exists and the goal is demultiplexing reads — use PRO's demultiplexing mode (pro-dex) instead of generation
- Sequences in the candidate space are of unequal length — the NP-complete optimality formulation and FPS selection assume a space where all sequences have the same length
- Data are PacBio reads — the article notes no tests on PacBio data have been performed yet

## Inputs

- Equal-length DNA k-mer candidate space (e.g. all 4^24 24-mer sequences) or a user-supplied txt file of candidate sequences (-i)
- Barcode length n (e.g. 24 nt to match official ONT barcodes)
- Sequencing error rate p (e.g. -a 0.88) and number of edit errors k to tolerate (-th, e.g. 2)
- Preselection size q (default 100,000)
- Optional fastq file of library-preparation flanking sequences (-f) to keep barcodes edit-distant from flanks

## Outputs

- select_barcode.txt containing the generated barcode sequences (e.g. 2,292 barcodes of 24 nt)
- pre_processing_barcodes.txt and pre_processing_barcodes_number.txt with pre-processed DNA sequences and their hash-function integers

## How to apply

Preprocess the 4^n candidate sequences by hashing each into a 64-bit integer (base mapping N(G)=0, N(A)=1, N(T)=2, N(C)=3), sorting in descending order, and preselecting the top q candidates (default q=100,000; use -q to control). Convert the sequencing error rate p and the number of tolerated errors k into the probability-divergence threshold t, using the insertion-worst-case probability p' = (1-p)^n * (1/4 * pi)^k for barcode length n (e.g. n=24). Run the farthest-point sampling algorithm (Algorithm 2): iteratively select the sequence minimizing weight(seq, {barcode}set) = max{ProDiv, ProDiv} and discard candidates s with ProDiv(s, seq1) > t until the threshold is exceeded; optionally pass flanking sequences via -f so barcodes keep edit distance from them. Write the resulting kit to select_barcode.txt; validate that every sequence has length n, the kit is maximal, and the maximum pairwise probability-divergence weight is <= t.

## Related tools

- **PRO** (Runs dataset preselection (64-bit hashing), threshold determination, and farthest-point sampling to emit the barcode kit into select_barcode.txt) — https://github.com/renztr/PRO.git
- **Python 3.7.0** (Interpreter required to execute pro_FPS_select.py)
- **edlib 1.3.9** (Edit-distance computation used in preprocessing/flanking-sequence filtering of candidates)
- **tqdm** (Progress reporting during the long candidate-iteration loops)

## Examples

```
python pro_FPS_select.py -l 10 -th 2 -a 0.88 -o examples/testoutput/k_mer_select -q 10000
```

## Evaluation signals

- Output file select_barcode.txt exists and contains exactly the expected number of barcodes (e.g. 2,292 for the 24-mer space with seed 0)
- Every emitted sequence has exactly n nucleotides (e.g. 24 nt, matching official ONT barcode length)
- Maximum pairwise probability-divergence weight between barcodes in the kit is <= threshold t
- Pre-processing outputs pre_processing_barcodes.txt and pre_processing_barcodes_number.txt are produced with consistent counts
- Demultiplexing accuracy of the generated kit on simulated ONT data (DeepSimulator 1.5) reaches ~98.29% for a kit of comparable size, exceeding Guppy

## Limitations

- Finding the optimal largest barcode set in an equal-length sequence space is NP-complete, so PRO's FPS result is a practical maximum, not a proven global optimum
- Barcodes similar to library flanking sequences cannot be accurately extracted by PRO; sequences with semiglobal-alignment edit distances <= 1 to both flanking sequences are removed before generation
- When kit size matches official ONT kits, PRO's accuracy is only comparable (not superior) to Guppy; ONT's own design strategy differs and does not ensure large probability divergence
- PRO runs on Linux only, per the README installation notes

## Evidence

- [readme] PRO ensures that any two barcode sequences in a barcode kit are sufficiently distinct from each other.: "PRO ensures that any two barcode sequences in a barcode kit are sufficiently distinct from each other."
- [methods] farthest-point sampling algorithm: "it then employs the farthest-point sampling algorithm to select the barcodes, which ensures that the generated sequences are adequately distinguishable"
- [methods] hashing function N: "each candidate DNA sequence with length n is hashed into a 64-bit integer with hashing function N"
