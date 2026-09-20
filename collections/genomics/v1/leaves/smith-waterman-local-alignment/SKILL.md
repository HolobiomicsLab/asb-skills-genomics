---
name: smith-waterman-local-alignment
description: 'Use when reconstructing FMAlign2''s MEMs-filtering and partial-chain-formation
  step: after MEMs are found and filtered, similar substrings must be recovered in
  sequences lacking MEMs via local alignment before segmentation.'
when_to_use_negative:
- The task is a standalone pairwise local alignment — this skill is specific to FMAlign2's
  segmentation pipeline, not general-purpose SSW alignment.
- Input is protein or short-read data — FMAlign2 targets ultra-long nucleotide sequences
  reaching billions of bases.
edam_operation: http://edamontology.org/operation_0495
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: FMAlign2
  role: Implements the MEMs filtering (global/local DP modes) and runs striped Smith–Waterman
    (SSW) to find similar substrings that form partial chains for sequence segmentation
  repo: https://github.com/malabz/FMAlign2
- name: FMAlign
  role: Predecessor whose global-chain search FMAlign2 redefines into partial-chain
    formation
  repo: https://github.com/malabz/FMAlign2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae014
    title: 'FMAlign2: a novel fast multiple nucleotide sequence alignment method for
      ultralong datasets'
  source_content_hash: sha256:b442632e45f3516b868c9532d7636662c57448b11b3bc5d41c69e5e2b8f97ae8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/malabz/FMAlign2
  tools:
  - FMAlign
  - FMAlign2
  edam_operation: http://edamontology.org/operation_0495
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# smith-waterman-local-alignment

## Summary

Runs the striped Smith–Waterman (SSW) algorithm on sequences where MEMs are absent to identify similar substrings, which are combined with colinear MEM chains to form the partial chains FMAlign2 uses for sequence segmentation. This is Step 2 of FMAlign2, replacing FMAlign's global-chain search.

## When to use

Use when reconstructing FMAlign2's MEMs-filtering and partial-chain-formation step: after MEMs are found and filtered, similar substrings must be recovered in sequences lacking MEMs via local alignment before segmentation.

## When NOT to use

- The task is a standalone pairwise local alignment — this skill is specific to FMAlign2's segmentation pipeline, not general-purpose SSW alignment.
- Input is protein or short-read data — FMAlign2 targets ultra-long nucleotide sequences reaching billions of bases.

## Inputs

- MEMs artifact from the upstream MEMs-finding step
- Source nucleotide sequences in FASTA format (ultra-long nucleotide dataset)
- Coverage threshold c (0≤c≤1) and gap-proportion limit (default 0.8)

## Outputs

- partial_chains.txt: partial chains combining colinear MEM chains with accepted SSW similar-substring anchors
- Filtered MEMs set (colinear, size-filtered, coverage ≥ c)
- Segmented sequences passed to the downstream MSA backend

## How to apply

Load the MEMs artifact from the upstream MEMs-finding step together with the source nucleotide FASTA (e.g. from the Zenodo deposit 10435770). First filter the MEMs with global or local dynamic programming, enforcing colinearity, MEM size (minimum MEM length, default 30 via -l), and the coverage threshold c (0≤c≤1), discarding MEMs that cover proportion ≤ c of the n sequences. Then run the striped Smith–Waterman (SSW) algorithm on the sequences where MEMs are absent to locate similar substrings for each valid MEM; discard any local alignment whose gap proportion exceeds 0.8 (default). Combine the surviving similar substrings with the colinear MEM chains to form partial chains (saved as partial_chains.txt), which replace the global-chain search of the original FMAlign approach and drive sequence segmentation.

## Related tools

- **FMAlign2** (Implements the MEMs filtering (global/local DP modes) and runs striped Smith–Waterman (SSW) to find similar substrings that form partial chains for sequence segmentation) — https://github.com/malabz/FMAlign2
- **FMAlign** (Predecessor whose global-chain search FMAlign2 redefines into partial-chain formation) — https://github.com/malabz/FMAlign2

## Examples

```
FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta -l 30 -f accurate
```

## Evaluation signals

- No retained local alignment has a gap proportion greater than 0.8
- partial_chains.txt exists and every partial chain is colinear (no inconsistent MEM ordering across the n sequences)
- All retained MEMs cover a proportion greater than the coverage threshold c of the n sequences
- Sequences lacking MEMs each receive at least one accepted SSW similar substring so segmentation can proceed

## Limitations

- Similar substrings whose local alignment gap proportion exceeds 0.8 (default) are discarded, potentially leaving anchors missing in highly divergent sequences
- If too few MEMs are found, the minimum MEM length (-l, default 30) may need to be lowered below 30
- The '-f fast' MEM filtering mode runs faster but may find fewer MEMs, trading sensitivity for speed

## Evidence

- [methods] MEMs filtering and partial-chain formation: [evidence span withheld — anchor status: not_found]
- [methods] SSW similar substrings: [evidence span withheld — anchor status: not_found]
- [methods] gap proportion 0.8: [evidence span withheld — anchor status: not_found]
- [abstract] partial chains replace global chains: "FMAlign2 leverages the suffix array to identify maximal exact matches, redefining the approach of FMAlign from searching for global chains to partial chains."
- [readme] MEM length parameter: "If too few MEMs are found, try lowering `-l` (e.g., below 30)."
