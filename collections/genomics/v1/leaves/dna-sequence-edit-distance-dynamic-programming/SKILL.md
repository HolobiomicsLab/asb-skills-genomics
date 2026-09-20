---
name: dna-sequence-edit-distance-dynamic-programming
description: Use when you need a unique pairwise alignment (with error-type counts)
  between two equal-ish-length DNA barcodes so that insertions, deletions and substitutions
  can be weighted independently, e.g. to compute ProDiv for demultiplexing.
when_to_use_negative:
- When only a plain edit-distance value (not error-type counts) is needed — edlib's
  built-in distance suffices without backtracking counts
- When barcodes are not short equal-length DNA sequences; the paper's probability-divergence
  model is defined for short DNA sequences used as nanopore barcodes, not long genomic
  reads
- When error types cannot be assumed independent — ProDiv is defined under the assumption
  that insertion, deletion, and substitution of each base occur independently during
  sequencing
edam_operation: http://edamontology.org/operation_0259
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: PRO
  role: 'Downstream consumer of the alignment counts: PRO-dex demultiplexes corrupted
    barcodes extracted via flanking sequences by probability divergence computed from
    m/i/d/s'
  repo: https://github.com/rztongr/PRO
- name: Python 3.7.0
  role: Interpreter in which the edit-distance DP matrix, backtracking, and ProDiv
    computation are implemented
  repo: https://github.com/rztongr/PRO
- name: edlib 1.3.9
  role: Reference edit-distance library for validating the DP alignment implementation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.fmre.2024.04.014
    title: Generating barcodes for nanopore sequencing data with PRO
  source_content_hash: sha256:014efea227828505a627bc0e5349eeef2f82c6b7fafed717e64317291db9cb8c
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
  - PRO
  - Python
  - Python 3.7.0
  - edlib 1.3.9
  edam_operation: http://edamontology.org/operation_0259
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# dna-sequence-edit-distance-dynamic-programming

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements edit-distance dynamic programming to align two short DNA sequences and count matches m, insertions i, deletions d, and substitutions s, which are then fed into PRO's probability-divergence formula ProDiv(X,Y) = pm^m * pd^d * (ps/3)^s * (pi/4)^i with pd = ps = pi = (1-pm)/3 for barcode demultiplexing.

## When to use

Use when you need a unique pairwise alignment (with error-type counts) between two equal-ish-length DNA barcodes so that insertions, deletions and substitutions can be weighted independently, e.g. to compute ProDiv for demultiplexing.

## When NOT to use

- When only a plain edit-distance value (not error-type counts) is needed — edlib's built-in distance suffices without backtracking counts
- When barcodes are not short equal-length DNA sequences; the paper's probability-divergence model is defined for short DNA sequences used as nanopore barcodes, not long genomic reads
- When error types cannot be assumed independent — ProDiv is defined under the assumption that insertion, deletion, and substitution of each base occur independently during sequencing

## Inputs

- Two short DNA sequences (X, Y) as strings, e.g. candidate barcode sequences from the 4^n k-mer space or user-supplied barcode FASTA/txt files
- Sequencing error rate a (e.g. 0.12 in the paper's examples, giving pm = 0.88)

## Outputs

- Alignment matrix and unique backtracked alignment with counts of matches m, insertions i, deletions d, and substitutions s
- Probability divergence ProDiv(X,Y) and symmetric weight max{ProDiv(X,Y), ProDiv(Y,X)} per pair, e.g. tabulated in probability_divergence_table.csv

## How to apply

Build the edit-distance dynamic-programming matrix in Python (e.g. with edlib==1.3.9 available as a reference implementation) to semiglobally align two DNA sequences of the barcodes. Backtrack through the matrix with a fixed tie-break order — deletion first, then substitution, then insertion — to obtain a single unique alignment, and record the counts of matches m, insertions i, deletions d, and substitutions s. Plug these counts into ProDiv(X,Y) = pm^m * pd^d * (ps/3)^s * (pi/4)^i under the paper's model that insertion, deletion and substitution of each base occur independently during sequencing, with pd = ps = pi = (1-pm)/3 (e.g. error rate 0.12 gives pm = 0.88). For symmetric comparison of two barcodes use weight(X,Y) = max{ProDiv(X,Y), ProDiv(Y,X)}, since PRO identifies barcodes 'according to the probability divergence'.

## Related tools

- **PRO** (Downstream consumer of the alignment counts: PRO-dex demultiplexes corrupted barcodes extracted via flanking sequences by probability divergence computed from m/i/d/s) — https://github.com/rztongr/PRO
- **Python 3.7.0** (Interpreter in which the edit-distance DP matrix, backtracking, and ProDiv computation are implemented) — https://github.com/rztongr/PRO
- **edlib 1.3.9** (Reference edit-distance library for validating the DP alignment implementation)

## Examples

```
pip install edlib==1.3.9  # then in Python: D[i][j] = min(D[i-1][j]+1, D[i][j-1]+1, D[i-1][j-1]+cost) to build the DP matrix, backtrack (del>sub>ins) and compute ProDiv = pm**m * pd**d * (ps/3)**s * (pi/4)**i
```

## Evaluation signals

- The implementation must reproduce the paper's worked example: ProDiv(AGTC,AGGT) = (1/4) pm^3 pd pi at error rate 0.12 (pm = 0.88), since AGTC→AGGT has 3 matches, 1 substitution, 0 insertions/deletions
- Backtracking with the fixed preference (deletion > substitution > insertion) must yield a single unique alignment per pair
- Neighborhood sums of probability divergences from an example S5 sequence to all sequences in S4, S5 and S6 at error rate 0.12 must be greater than 0.85 but strictly less than 1
- All pairwise divergence values and neighborhood sums should be exported to probability_divergence_table.csv with no missing entries
- Cross-check edit distances against edlib==1.3.9 for random sequence pairs

## Limitations

- PRO underperforms Guppy when flanking sequences are similar to the barcode and cannot be accurately extracted, so alignments start from corrupted barcode extractions
- ONT's design strategy for barcode kits B1, B2 and B3 does not ensure large enough probability divergence between barcodes, degrading downstream demultiplexing
- The model assumes independent per-base insertion, deletion and substitution events; other error models are not supported
- No tests on PacBio data have been performed, so applicability beyond ONT is unverified

## Evidence
