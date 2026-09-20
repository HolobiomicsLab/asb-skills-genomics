---
name: dynamic-programming-chaining
description: 'Use when segmenting ultra-long nucleotide sequences for multiple sequence
  alignment: MEMs have been found via suffix array and must be filtered (colinearity,
  size, coverage c) and chained into partial chains for downstream segmentation.'
when_to_use_negative:
- MEMs have not yet been found — this skill filters and chains existing MEMs, it does
  not do suffix-array MEM discovery
- Input is protein or already-aligned data — FMAlign2 targets ultra-long nucleotide
  (DNA) FASTA input
- A single global chain over all sequences is desired — this skill intentionally replaces
  FMAlign's global-chain search with partial chains
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: FMAlign2
  role: Implements the global/local dynamic-programming MEM filtering and partial-chain
    formation (Step 2) used for sequence segmentation
  repo: https://github.com/malabz/FMAlign2
- name: SSW (striped Smith–Waterman)
  role: Finds similar substrings in sequences where a valid MEM is absent; local alignments
    with gap proportion > 0.8 are discarded
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae014
    title: 'FMAlign2: a novel fast multiple nucleotide sequence alignment method for
      ultralong datasets'
  source_content_hash: sha256:90ff10ea03f45fd9225c9ef21d0510141f128052cff3b139f12284edcf1ab6d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/malabz/FMAlign2
  tools:
  - FMAlign2
  - SSW (striped Smith–Waterman)
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# dynamic-programming-chaining

## Summary

Reconstructs FMAlign2's Step 2: filtering suffix-array-derived MEMs with global/local dynamic programming (colinearity, MEM size, sequence-coverage threshold c) and augmenting them with striped Smith–Waterman (SSW) similar substrings to form partial chains instead of FMAlign's global chains.

## When to use

Use when segmenting ultra-long nucleotide sequences for multiple sequence alignment: MEMs have been found via suffix array and must be filtered (colinearity, size, coverage c) and chained into partial chains for downstream segmentation.

## When NOT to use

- MEMs have not yet been found — this skill filters and chains existing MEMs, it does not do suffix-array MEM discovery
- Input is protein or already-aligned data — FMAlign2 targets ultra-long nucleotide (DNA) FASTA input
- A single global chain over all sequences is desired — this skill intentionally replaces FMAlign's global-chain search with partial chains

## Inputs

- MEMs artifact from the upstream suffix-array MEMs-finding step
- Source nucleotide sequences in FASTA format (from the Zenodo deposit or user input)
- Coverage threshold c (0≤c≤1) and minimum MEM length (default 30)

## Outputs

- partial_chains.txt containing the partial chains (colinear MEM chains plus accepted SSW similar substrings)
- Segmentation of the input sequences ready for parallel subMSA

## How to apply

Load the MEMs produced by the upstream suffix-array MEMs-finding step together with the input nucleotide FASTA. Filter the MEMs using global or local dynamic programming, enforcing colinearity, a minimum MEM length (default 30 via -l), MEM size, and discarding MEMs whose sequence coverage proportion is below threshold c (0≤c≤1); the README exposes a fast (-f fast) versus accurate (default) filtering mode. For sequences where a valid MEM is absent, run the striped Smith–Waterman (SSW) algorithm to find similar substrings and discard any local alignment whose gap proportion exceeds the default 0.8. Combine the colinear MEM chains with the surviving SSW anchors to form the partial chains (saved as partial_chains.txt) that replace FMAlign's global-chain search and define the segmentation points for subsequent subMSA.

## Related tools

- **FMAlign2** (Implements the global/local dynamic-programming MEM filtering and partial-chain formation (Step 2) used for sequence segmentation) — https://github.com/malabz/FMAlign2
- **SSW (striped Smith–Waterman)** (Finds similar substrings in sequences where a valid MEM is absent; local alignments with gap proportion > 0.8 are discarded)

## Examples

```
FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta -l 30 -f accurate
```

## Evaluation signals

- All retained MEM chains are colinear and satisfy the minimum MEM length (default 30) and coverage threshold c
- No accepted SSW local alignment has a gap proportion greater than 0.8
- partial_chains.txt exists and covers the same n input sequences as the source FASTA
- Few-MEM failure mode is detectable: if too few MEMs are found, lowering -l below 30 should increase anchor count
- Downstream segmentation into subMSA tasks succeeds using the partial chains

## Limitations

- The fast filtering mode (-f fast) runs faster but may find fewer MEMs at the cost of sensitivity
- If too few MEMs are found, the minimum MEM length may need to be lowered below the default 30
- Similar substrings whose local alignment gap proportion exceeds 0.8 are discarded, which can leave sequences without anchors

## Evidence

- [abstract] suffix array MEMs to partial chains: "FMAlign2 leverages the suffix array to identify maximal exact matches, redefining the approach of FMAlign from searching for global chains to partial chains."
- [other] SSW similar substrings and 0.8 gap filter: [evidence span withheld — anchor status: not_found]
- [other] coverage threshold and gap proportion: [evidence span withheld — anchor status: not_found]
- [readme] fast vs accurate filtering mode: "If MEM detection takes too long, consider `-f fast`; it runs faster but may find fewer MEMs."
