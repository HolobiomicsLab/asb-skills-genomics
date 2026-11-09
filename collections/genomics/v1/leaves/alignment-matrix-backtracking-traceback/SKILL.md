---
name: alignment-matrix-backtracking-traceback
description: Use when computing probability divergence ProDiv between two equal-length
  DNA barcode sequences requires a global edit-distance DP matrix plus a deterministic
  deletion-first traceback to obtain unique alignment counts (m, s, i, d).
when_to_use_negative:
- Computing edit distances between barcodes and flanking sequences, where PRO uses
  semiglobal alignment rather than the global matrix-plus-traceback used for ProDiv
  between equal-length barcodes
- Data from platforms whose errors are not base-level independent substitution/indel
  events — the paper notes no PacBio tests have been performed
- Demultiplexing an already-selected barcode kit, where PRO-dex applies the precomputed
  divergence rather than re-deriving alignments
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0097
- http://edamontology.org/topic_0091
tools:
- name: PRO
  role: The package whose barcode generation and PRO-dex demultiplexing consume the
    probability divergence produced by this alignment/traceback implementation
  repo: https://github.com/renztr/PRO.git
- name: Python
  role: Language in which the edit-distance DP algorithm, traceback, and ProDiv computation
    are implemented (recommended version 3.7.0 and above)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.fmre.2024.04.014
    title: Generating barcodes for nanopore sequencing data with PRO
  source_content_hash: sha256:5c71d284ec67094cbf418d9331807f1dbccf90f9f1cca3d3f3ec8a664463fe28
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0097
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# alignment-matrix-backtracking-traceback

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements the edit-distance dynamic-programming alignment and deterministic traceback used by PRO to count matches, substitutions, insertions, and deletions between two DNA barcode sequences, enabling the probability divergence ProDiv(X,Y) computation that underpins barcode generation and demultiplexing.

## When to use

Use when computing probability divergence ProDiv between two equal-length DNA barcode sequences requires a global edit-distance DP matrix plus a deterministic deletion-first traceback to obtain unique alignment counts (m, s, i, d).

## When NOT to use

- Computing edit distances between barcodes and flanking sequences, where PRO uses semiglobal alignment rather than the global matrix-plus-traceback used for ProDiv between equal-length barcodes
- Data from platforms whose errors are not base-level independent substitution/indel events — the paper notes no PacBio tests have been performed
- Demultiplexing an already-selected barcode kit, where PRO-dex applies the precomputed divergence rather than re-deriving alignments

## Inputs

- Two short DNA barcode sequences of equal length (e.g., AGTC and AGGT)
- Sequencing error rate / match probability pm (paper example: error rate 0.12, pm = 0.88)
- Python script implementing the DP matrix and traceback

## Outputs

- Alignment matrix for the two sequences
- Counts of matches m, substitutions s, insertions i, deletions d from the unique traceback
- ProDiv(X,Y) = pm^m * pd^d * (ps/3)^s * (pi/4)^i value per pair
- Symmetric weight weight(X,Y) = max{ProDiv(X,Y), ProDiv(Y,X)} per pair
- Neighborhood sums from an S5 sequence to all sequences in S4, S5, and S6
- probability_divergence_table.csv with all pairwise divergence values and neighborhood sums

## How to apply

Implement the edit-distance dynamic-programming algorithm in Python to align two DNA sequences and build the alignment matrix. Backtrack through the matrix preferring deletion first, then substitution, then insertion — this fixed preference order is what makes the alignment unique, so the counted matches m, insertions i, deletions d, and substitutions s are reproducible. Compute ProDiv(X,Y) = pm^m * pd^d * (ps/3)^s * (pi/4)^i under the assumption that insertion, deletion, and substitution of each base occur independently during sequencing, with pd = ps = pi = (1-pm)/3; at the paper's error rate 0.12, pm = 0.88 (the same value passed as -a to pro_FPS_select.py). Validate against the paper's worked example, ProDiv(AGTC,AGGT) = (1/4) pm^3 pd pi. Because ProDiv is directional, tabulate the symmetric weight weight(X,Y) = max{ProDiv(X,Y), ProDiv(Y,X)} for all evaluated pairs and sum divergences from an example S5 sequence to all sequences in S4, S5, and S6, which must land in (0.85, 1).

## Related tools

- **PRO** (The package whose barcode generation and PRO-dex demultiplexing consume the probability divergence produced by this alignment/traceback implementation) — https://github.com/renztr/PRO.git
- **Python** (Language in which the edit-distance DP algorithm, traceback, and ProDiv computation are implemented (recommended version 3.7.0 and above))

## Examples

```
ProDiv = pm**m * pd**d * (ps/3)**s * (pi/4)**i  # pd = ps = pi = (1-pm)/3; assert ProDiv(AGTC,AGGT) == 0.25*pm**3*pd*pi at pm=0.88
```

## Evaluation signals

- Script reproduces the paper's worked example ProDiv(AGTC,AGGT) = (1/4) pm^3 pd pi at error rate 0.12 (pm = 0.88)
- Traceback preference order (deletion, then substitution, then insertion) yields a single unique alignment; counts satisfy m + s + d = len(X) and m + s + i = len(Y) for equal-length inputs
- Error-probability invariant pd = ps = pi = (1-pm)/3 holds throughout all ProDiv computations
- Computed neighborhood sums from the S5 example sequence to S4, S5, and S6 are each greater than 0.85 but strictly less than 1, matching the reported property
- probability_divergence_table.csv is emitted containing all pairwise divergence values and neighborhood sums, with weight(X,Y) = max{ProDiv(X,Y), ProDiv(Y,X)} tabulated

## Limitations

- The unique alignment — and hence the (m, s, i, d) counts feeding ProDiv — depends on the tie-breaking preference (deletion > substitution > insertion); a different order changes the divergence value
- ProDiv is asymmetric, so the symmetric max weight must be applied explicitly when comparing barcode pairs
- The independence assumption on base-level errors was validated on ONT simulated data (DeepSimulator 1.5) only; no PacBio tests have been performed
- PRO underperforms Guppy when flanking sequences are similar to the barcode, since corrupted barcodes cannot be accurately extracted before divergence-based identification

## Evidence

- [methods] ProDiv(X,Y)=pmmpdd(13ps)s(14pi)i.: "ProDiv(X,Y)=pmmpdd(13ps)s(14pi)i."
- [readme] PRO is based on probability divergence, a "metric" we propose for the dissimilarity between two DNA sequences.: "PRO is based on probability divergence, a "metric" we propose for the dissimilarity between two DNA sequences."
- [other] Backtrack through the alignment matrix preferring deletion first, then substitution, then insertion, to obtain the unique alignment: [evidence span withheld — anchor status: not_found]
- [other] Compute ProDiv(X,Y) = pm^m * pd^d * (ps/3)^s * (pi/4)^i with the assumption pd = ps = pi = (1-pm)/3: [evidence span withheld — anchor status: not_found]
