---
name: maximal-exact-match-colinearity-filtering
description: 'Use when segmenting ultra-long nucleotide datasets for MSA: after MEMs
  are found with the suffix array, this skill filters them into colinear partial chains
  so downstream subMSA can be run on manageable segments.'
when_to_use_negative:
- Input is a large number of short, highly similar DNA/RNA reads better served by
  the center-star/suffix-tree strategy of HAlign3 or HAlign4 rather than ultralong-sequence
  segmentation
- The sequences are already segmented or already aligned — rerunning MEM filtering
  would discard existing anchors without benefit
- Sequences are too short to contain meaningful maximal exact matches; lowering -l
  below the default 30 may still fail to yield usable anchors
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0157
tools:
- name: FMAlign2
  role: Implements the global/local dynamic-programming MEM filtering (colinearity,
    size, coverage threshold c) and combines MEMs with SSW similar substrings into
    partial chains; run via the FMAlign2 CLI
  repo: https://github.com/malabz/FMAlign2
- name: SSW (striped Smith–Waterman)
  role: Invoked by FMAlign2 on sequences where MEMs are absent to find similar substrings;
    local alignments with gap proportion > 0.8 are discarded
- name: FMAlign
  role: Predecessor whose global-chain search FMAlign2 replaces with partial chains
    formed by this filtering step
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae014
    title: 'FMAlign2: a novel fast multiple nucleotide sequence alignment method for
      ultralong datasets'
  source_content_hash: sha256:235bd86de27a2f514c3d945426704ed10b792335d1049c6d69d7c5ba7381b246
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/malabz/FMAlign2
  tools:
  - FMAlign
  - FMAlign2
  - SSW (striped Smith–Waterman)
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# maximal-exact-match-colinearity-filtering

## Summary

Filters suffix-array-derived maximal exact matches (MEMs) for colinearity, size, and minimum sequence coverage (threshold c, 0≤c≤1), then augments the surviving MEM chains with striped Smith–Waterman (SSW) similar substrings to form the partial chains FMAlign2 uses for sequence segmentation.

## When to use

Use when segmenting ultra-long nucleotide datasets for MSA: after MEMs are found with the suffix array, this skill filters them into colinear partial chains so downstream subMSA can be run on manageable segments.

## When NOT to use

- Input is a large number of short, highly similar DNA/RNA reads better served by the center-star/suffix-tree strategy of HAlign3 or HAlign4 rather than ultralong-sequence segmentation
- The sequences are already segmented or already aligned — rerunning MEM filtering would discard existing anchors without benefit
- Sequences are too short to contain meaningful maximal exact matches; lowering -l below the default 30 may still fail to yield usable anchors

## Inputs

- MEMs artifact from the upstream suffix-array MEMs-finding step
- Source nucleotide sequences in FASTA format from the Zenodo deposit (doi=10.5281/zenodo.10435770)
- Coverage threshold c (0≤c≤1) and MEM filtering mode (-f accurate|fast)

## Outputs

- partial_chains.txt containing colinear MEM chains combined with accepted SSW similar substrings
- Filtered MEM set satisfying colinearity, size, and coverage-threshold constraints

## How to apply

Load the MEMs artifact produced by the upstream MEMs-finding step together with the source nucleotide FASTA from the Zenodo deposit (doi=10.5281/zenodo.10435770). Run MEM filtering with global or local dynamic programming, enforcing colinearity between MEMs, MEM size, and the coverage threshold c; MEMs covering a proportion ≤ c of the n sequences are discarded (the README exposes this as the -f mode, default 'accurate', with 'fast' trading sensitivity for speed). For each valid MEMs, run the striped Smith–Waterman (SSW) algorithm on the sequences where the MEMs are absent to locate similar substrings, and discard any local alignment whose gap proportion exceeds the default limit of 0.8. Combine the surviving colinear MEM chains with the accepted SSW similar substrings to form partial chains — this replaces the global-chain search of the original FMAlign approach and yields better segmentation of ultralong sequences. Write the partial chains to partial_chains.txt before handing off to the vertical-division subMSA step.

## Related tools

- **FMAlign2** (Implements the global/local dynamic-programming MEM filtering (colinearity, size, coverage threshold c) and combines MEMs with SSW similar substrings into partial chains; run via the FMAlign2 CLI) — https://github.com/malabz/FMAlign2
- **SSW (striped Smith–Waterman)** (Invoked by FMAlign2 on sequences where MEMs are absent to find similar substrings; local alignments with gap proportion > 0.8 are discarded)
- **FMAlign** (Predecessor whose global-chain search FMAlign2 replaces with partial chains formed by this filtering step)

## Examples

```
FMAlign2 -i /path/to/input.fasta -o /path/to/output.fasta -l 30 -f accurate
```

## Evaluation signals

- No retained SSW local alignment has a gap proportion greater than the 0.8 default limit
- Every retained MEM is colinear with the others, exceeds the size constraint, and covers a proportion > c of the n sequences
- partial_chains.txt is produced and its partial chains correctly segment the input sequences for the downstream subMSA step
- Switching -f from 'fast' to 'accurate' recovers more MEMs (sensitivity), consistent with the README's stated speed/sensitivity trade-off
- Downstream alignment SP score computed via sp.py (e.g., --match 0 --mismatch 1 --gap1 2 --gap2 2) remains competitive with FMAlign while runtime drops on ultralong datasets

## Limitations

- The 'fast' MEM filtering mode (-f fast) runs faster but may find fewer MEMs, reducing sensitivity
- If too few MEMs are found, the minimum MEM length -l must be lowered below the default 30 to recover anchors
- MEMs covering a proportion ≤ c of the n sequences are discarded, so highly divergent sequences lacking exact matches depend entirely on SSW similar substrings passing the 0.8 gap-proportion cutoff

## Evidence

- [abstract] redefining FMAlign global chains: "FMAlign2 leverages the suffix array to identify maximal exact matches, redefining the approach of FMAlign from searching for global chains to partial chains."
- [readme] MEM filtering mode parameter: "-f <mode>` (default: `accurate`). MEM filtering mode; use `fast` to speed up at the cost of sensitivity."
- [readme] minimum MEM length parameter: "-l <int>` (default: 30). Minimum MEM length."
- [methods] conda installation of FMAlign2: "We recommend installing **FMAlign2** with **conda**."
