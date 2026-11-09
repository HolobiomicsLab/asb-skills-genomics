---
name: combinatorial-maximum-clique-heuristic-selection
description: Use when the goal is to generate the largest possible barcode kit from
  a 4^n equal-length DNA k-mer space (e.g., 24-mer) under an error-tolerance threshold,
  where exhaustive or exact maximum-clique search is infeasible and a greedy heuristic
  is required.
when_to_use_negative:
- The task is demultiplexing already-sequenced reads rather than generating a barcode
  kit — use PRO's demultiplexing mode (PRO-dex with probability divergence on flanking-extracted
  corrupted barcodes) instead.
- The candidate sequences have unequal lengths — the NP-completeness result and the
  equal-length k-mer preprocessing (4^n hashing) assume all sequences share the same
  length n.
- The platform is PacBio rather than ONT — the paper states no tests on PacBio data
  have been performed, so kit sizing and thresholds are unvalidated there.
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0091
tools:
- name: PRO
  role: Implements the preprocessing (64-bit hashing, preselection), probability-divergence
    thresholding, and farthest-point sampling (Algorithm 2) that heuristically solve
    the NP-complete maximum barcode-set selection problem; outputs select_barcode.txt.
  repo: https://github.com/rztongr/PRO
- name: Python 3.7.0
  role: Interpreter required to run pro_FPS_select.py and the barcode-generation workflow
    (recommended version 3.7.0 and above).
- name: edlib 1.3.9
  role: Provides semiglobal-alignment edit distances used to remove preselected candidates
    similar to flanking sequences (both edit distances <= 1) before sampling.
- name: tqdm
  role: Progress reporting during the iterative farthest-point sampling loop over
    large candidate sets.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.fmre.2024.04.014
    title: Generating barcodes for nanopore sequencing data with PRO
  source_content_hash: sha256:8ec9256390b08d691dbb446a79e1b6e46bd3f3ce12960222fd203cd1140c092f
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
  - tqdm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# combinatorial-maximum-clique-heuristic-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects a provably hard (NP-complete) maximal set of mutually distinguishable, equal-length DNA barcodes by replacing exhaustive maximum-clique search with PRO's farthest-point sampling heuristic over a probability-divergence distance, yielding kits as large as 2,292 barcodes at 24 nt in the 24-mer space.

## When to use

Use when the goal is to generate the largest possible barcode kit from a 4^n equal-length DNA k-mer space (e.g., 24-mer) under an error-tolerance threshold, where exhaustive or exact maximum-clique search is infeasible and a greedy heuristic is required.

## When NOT to use

- The task is demultiplexing already-sequenced reads rather than generating a barcode kit — use PRO's demultiplexing mode (PRO-dex with probability divergence on flanking-extracted corrupted barcodes) instead.
- The candidate sequences have unequal lengths — the NP-completeness result and the equal-length k-mer preprocessing (4^n hashing) assume all sequences share the same length n.
- The platform is PacBio rather than ONT — the paper states no tests on PacBio data have been performed, so kit sizing and thresholds are unvalidated there.

## Inputs

- 4^n candidate DNA sequences of the equal-length k-mer space (e.g., 4^24 24-nt sequences), or a user-specified .txt file of DNA sequences passed via -i
- Barcode length n (parameter -l, e.g., 24 to match official ONT barcodes)
- Number of tolerable edit errors k (parameter -th, e.g., 2)
- Sequencing accuracy parameter a / error rate p (e.g., -a 0.88) used to derive the probability-divergence threshold t
- Preselection size q (parameter -q, default 100,000)
- Optional FASTQ of library-preparation flanking sequences (parameter -f) for edit-distance filtering

## Outputs

- select_barcode.txt containing the generated barcode kit (e.g., 2,292 barcodes of 24 nt)
- pre_processing_barcodes.txt with the pre-processed candidate DNA sequences
- pre_processing_barcodes_number.txt with the 64-bit hashed integers corresponding to the pre-processed sequences

## How to apply

First preprocess the 4^n candidate space (n=24 gives 4^24 sequences) by hashing each sequence into a 64-bit integer with mapping N(G)=0, N(A)=1, N(T)=2, N(C)=3, sorting integers in descending order, and preselecting q=100,000 candidates (default q=100,000 per the README; adjustable via -q). Convert the nanopore sequencing error rate p and the number of errors k the kit should tolerate (parameter -th) into the probability-divergence threshold t using the insertion-worst-case probability p' = (1-p)^n * (1/4 * pi)^k for barcode length n. Run the adapted farthest-point sampling algorithm (Algorithm 2), iteratively selecting the candidate minimizing the weight(seq, {barcode}set) = max{ProDiv} and discarding candidates s with ProDiv(s, seq1) > t to prune the search. Optionally, if flanking sequences are supplied (-f), remove preselected candidates whose semiglobal-alignment edit distances to the flanks are both <= 1 before sampling. Write the kit to select_barcode.txt; in the 24-mer space with seed 0 this reproduces PRO's reported maximum kit of 2,292 barcodes at the official ONT 24 bp length.

## Related tools

- **PRO** (Implements the preprocessing (64-bit hashing, preselection), probability-divergence thresholding, and farthest-point sampling (Algorithm 2) that heuristically solve the NP-complete maximum barcode-set selection problem; outputs select_barcode.txt.) — https://github.com/rztongr/PRO
- **Python 3.7.0** (Interpreter required to run pro_FPS_select.py and the barcode-generation workflow (recommended version 3.7.0 and above).)
- **edlib 1.3.9** (Provides semiglobal-alignment edit distances used to remove preselected candidates similar to flanking sequences (both edit distances <= 1) before sampling.)
- **tqdm** (Progress reporting during the iterative farthest-point sampling loop over large candidate sets.)

## Examples

```
python pro_FPS_select.py -l 10 -th 2 -a 0.88 -o examples/testoutput/k_mer_select -q 10000
```

## Evaluation signals

- select_barcode.txt exists and contains exactly 2,292 barcode sequences when run in the 24-mer space with seed 0, matching the reported maximum kit size.
- Every sequence in the output kit has length exactly 24 nt (equal to the official ONT barcode length).
- The maximum pairwise probability-divergence weight between barcodes is <= t, where t was derived from the error rate p and tolerated errors k via p' = (1-p)^n * (1/4 * pi)^k.
- No retained candidate s satisfies ProDiv(s, seq1) > t — the threshold-pruning step removed all such sequences before sampling.
- The preprocessing artifacts pre_processing_barcodes.txt and pre_processing_barcodes_number.txt are emitted and contain the preselected q candidates (default 100,000) with their hashed integers.

## Limitations

- Finding the optimal largest barcode set in an equal-length DNA sequence space is NP-complete, so the farthest-point sampling heuristic guarantees distinguishability (large probability divergence) but not a globally optimal kit.
- PRO underperforms Guppy on kits B1, B2, and B3 when the flanking sequences are similar to the barcode, because PRO cannot accurately extract such barcodes.
- ONT's design strategy for B1–B3 differs and does not ensure large probability divergence between barcodes, so comparisons against official kits can reflect divergent design criteria rather than kit size alone.
- No tests on PacBio data have been performed, although the authors note the approach could in principle be employed for both ONT and PacBio sequencing data.

## Evidence

- [abstract] prove that finding the optimal largest barcode set in a given DNA sequence space in which all sequences have the same length is theoretically NP-complete: "prove that finding the optimal largest barcode set in a given DNA sequence space in which all sequences have the same length is theoretically NP-complete"
- [methods] it then employs the farthest-point sampling algorithm to select the barcodes, which ensures that the generated sequences are adequately distinguishable: "it then employs the farthest-point sampling algorithm to select the barcodes, which ensures that the generated sequences are adequately distinguishable"
- [methods] to improve the efficiency of barcode generation, we preselect the 4n DNA sequences to effectively reduce the number of candidate sequences: "to improve the efficiency of barcode generation, we preselect the 4n DNA sequences to effectively reduce the number of candidate sequences"
- [intro] we innovatively define a new measure of divergence between two short DNA sequences, namely, probability divergence: "we innovatively define a new measure of divergence between two short DNA sequences, namely, probability divergence"
- [readme] PRO is based on probability divergence, a "metric" we propose for the dissimilarity between two DNA sequences.: "PRO is based on probability divergence, a "metric" we propose for the dissimilarity between two DNA sequences."
