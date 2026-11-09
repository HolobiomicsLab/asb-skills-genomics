---
name: gap-proportion-alignment-filtering
description: Use when reconstructing or reimplementing FMAlign2's MEM-filtering and
  partial-chain-formation step, or when deciding which local-alignment anchors are
  trustworthy for segmenting ultralong nucleotide sequences.
when_to_use_negative:
- You only need the final MSA and do not care about the internal segmentation — just
  run FMAlign2 end-to-end rather than rebuilding the filtering stage
- The input is not ultralong nucleotide sequence data; FMAlign2 targets nucleotide
  sequences reaching billions in length, and MEM/SSW-chain segmentation is unnecessary
  for short, easy alignments
- A full global chain of MEMs across all sequences is required — this skill intentionally
  replaces FMAlign's global-chain search with partial chains
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: FMAlign2
  role: Implements MEM filtering (global/local DP modes, colinearity, size, coverage
    threshold c) and invokes SSW to build partial chains for sequence segmentation
  repo: https://github.com/malabz/FMAlign2
- name: SSW (striped Smith–Waterman)
  role: Runs local alignment in sequences where MEMs are absent to find similar substrings;
    alignments with gap proportion > 0.8 are discarded
  repo: https://github.com/malabz/FMAlign2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae014
    title: 'FMAlign2: a novel fast multiple nucleotide sequence alignment method for
      ultralong datasets'
  source_content_hash: sha256:a8e909312d13489b3b73f0d046d14f41e918d71238b8847c9ce7ac5ac8b10ed9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/malabz/FMAlign2
  tools:
  - FMAlign2
  - SSW (striped Smith–Waterman)
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# gap-proportion-alignment-filtering

## Summary

FMAlign2's Step 2 mechanism that filters maximal exact matches (MEMs) for colinearity, size, and sequence-coverage threshold c, then uses striped Smith–Waterman (SSW) local alignments — discarding any whose gap proportion exceeds 0.8 — to form partial chains for sequence segmentation.

## When to use

Use when reconstructing or reimplementing FMAlign2's MEM-filtering and partial-chain-formation step, or when deciding which local-alignment anchors are trustworthy for segmenting ultralong nucleotide sequences.

## When NOT to use

- You only need the final MSA and do not care about the internal segmentation — just run FMAlign2 end-to-end rather than rebuilding the filtering stage
- The input is not ultralong nucleotide sequence data; FMAlign2 targets nucleotide sequences reaching billions in length, and MEM/SSW-chain segmentation is unnecessary for short, easy alignments
- A full global chain of MEMs across all sequences is required — this skill intentionally replaces FMAlign's global-chain search with partial chains

## Inputs

- Maximal exact matches (MEMs) identified from suffix array over the input dataset
- Nucleotide sequences in FASTA format (the source input file, e.g. /path/to/input.fasta)
- Coverage threshold c (0≤c≤1)
- Gap-proportion cutoff (default 0.8)

## Outputs

- Colinear MEM chains filtered by size and coverage threshold
- Accepted SSW similar substrings (gap proportion ≤ 0.8)
- Partial chains used for sequence segmentation (e.g. partial_chains.txt)

## How to apply

Take the MEMs found via suffix array from the nucleotide FASTA and filter them with global or local dynamic programming, enforcing colinearity, MEM size, and a coverage threshold c (0≤c≤1) that discards MEMs covering ≤ c of the n sequences. For each valid MEMs set, run the striped Smith–Waterman (SSW) algorithm on sequences where the MEMs are absent to locate similar substrings. Discard any similar substring whose local alignment has a gap proportion greater than 0.8 (the default cutoff). Combine the surviving similar substrings with the MEMs into partial chains, replacing FMAlign's global-chain search; these partial chains drive the subsequent sequence segmentation. In practice the filtering mode is selected with FMAlign2's -f flag (default 'accurate'; 'fast' trades sensitivity for speed) and the minimum MEM length with -l (default 30).

## Related tools

- **FMAlign2** (Implements MEM filtering (global/local DP modes, colinearity, size, coverage threshold c) and invokes SSW to build partial chains for sequence segmentation) — https://github.com/malabz/FMAlign2
- **SSW (striped Smith–Waterman)** (Runs local alignment in sequences where MEMs are absent to find similar substrings; alignments with gap proportion > 0.8 are discarded) — https://github.com/malabz/FMAlign2

## Examples

```
FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta -f accurate -l 30
```

## Evaluation signals

- No retained similar substring has a local-alignment gap proportion greater than 0.8
- All retained MEMs satisfy colinearity and the coverage threshold c (no MEM covering ≤ c of the n sequences survives)
- Every discarded-gap scenario traces to an SSW local alignment performed only on sequences where the MEMs were absent
- The resulting partial chains (partial_chains.txt) show the shift from FMAlign's global chains to partial chains, i.e. improved sequence segmentation
- Changing -f from 'accurate' to 'fast' reduces MEM yield — a fast-mode run should never surface more anchors than the accurate mode on the same input

## Limitations

- Using the 'fast' MEM filtering mode (-f fast) runs faster but may find fewer MEMs; too few MEMs may require lowering -l below the default 30
- The 0.8 gap-proportion cutoff is a default; alignments just under the limit can still be gappy anchors
- Partial chains replace global chains, so coverage of the full-length relationship between distant sequences depends on anchor density

## Evidence

- [methods] gap-proportion discard rule: [evidence span withheld — anchor status: not_found]
- [abstract] partial-chain redefinition: "FMAlign2 leverages the suffix array to identify maximal exact matches, redefining the approach of FMAlign from searching for global chains to partial chains."
- [readme] filtering mode flag: "`-f <mode>` (default: `accurate`). MEM filtering mode; use `fast` to speed up at the cost of sensitivity."
