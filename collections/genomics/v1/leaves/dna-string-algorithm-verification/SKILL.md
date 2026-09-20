---
name: dna-string-algorithm-verification
description: Use when re-implementing or validating a DNA decomposition algorithm
  (e.g., uTR's greedy unit selection) and you need to confirm the implementation reproduces
  the paper's stated extended-penalty values and selected-unit set.
when_to_use_negative:
- You only need to run uTR end-to-end on reads to annotate tandem repeat patterns
  (use uTR directly with -f/-o rather than re-implementing the selection loop)
- The repeat structure involves variant units with substitutions/indels in reads —
  vTR's variant-unit decomposition, not the exact-match greedy loop, is the appropriate
  tool
- Units longer than 20 nt need to be discovered first — use mTR to precompute long
  representative units before any selection/verification
tools:
- name: uTR
  role: Reference implementation of the mosaic tandem repeat decomposition whose greedy
    unit-selection loop is being re-implemented and verified; rotated unit strings
    (e.g., CAG vs AGC) are permitted candidates
  repo: https://github.com/morisUtokyo/uTR
- name: mTR
  role: Precomputes long representative units (> 20 nt) that can be fed to uTR via
    -u when candidate units cannot be enumerated naively
  repo: https://github.com/morisUtokyo/mTR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad185
    title: Decomposing mosaic tandem repeats accurately from long reads
  source_content_hash: sha256:e6b831c7e2ab310cd76fe37ab19f2375302d0fe7d4616ad49e02e62614e675c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morisUtokyo/uTR
  tools:
  - mTR
  - uTR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# dna-string-algorithm-verification

## Summary

Implement and empirically verify DNA string algorithms from published descriptions — here, uTR's greedy unit-selection loop that minimizes the extended penalty sum(|u|+o(u)) + sum of uncovered decomposition lengths — by re-implementing the control loop and checking it reproduces the paper's running example (S=ACCGACCGACCG → U={ACCG}).

## When to use

Use when re-implementing or validating a DNA decomposition algorithm (e.g., uTR's greedy unit selection) and you need to confirm the implementation reproduces the paper's stated extended-penalty values and selected-unit set.

## When NOT to use

- You only need to run uTR end-to-end on reads to annotate tandem repeat patterns (use uTR directly with -f/-o rather than re-implementing the selection loop)
- The repeat structure involves variant units with substitutions/indels in reads — vTR's variant-unit decomposition, not the exact-match greedy loop, is the appropriate tool
- Units longer than 20 nt need to be discovered first — use mTR to precompute long representative units before any selection/verification

## Inputs

- Input DNA string S as a plain string (test string: ACCGACCGACCG)
- Candidate unit set V (all substrings/rotations permitted as repeat units)
- Decomposition D of S into substring strings used to compute residual coverage

## Outputs

- Selected unit set U (expected: {ACCG} for the running example)
- greedy_selection_report.txt with the selection trace: candidate and extended penalty at each iteration

## How to apply

Formalize the extended penalty as sum over selected units U of (|u|+o(u)) plus the total length of decomposition strings not covered by U, which trades unit length and occurrence count against residual decomposition length. Enumerate the candidate unit set V, allowing rotated unit strings as uTR does for CAG/AGC-style rotations, and initialize U empty. In each greedy iteration, evaluate the extended penalty for every candidate unit added to U, add the minimizer, and terminate when no candidate reduces the penalty. Validate against the running example S=ACCGACCGACCG: the first iteration must select ACCG with extended penalty 7, beating AC/CG (11) and CGAC (10), and the loop must terminate with U={ACCG}; write the full per-iteration trace (candidate, penalty) to greedy_selection_report.txt.

## Related tools

- **uTR** (Reference implementation of the mosaic tandem repeat decomposition whose greedy unit-selection loop is being re-implemented and verified; rotated unit strings (e.g., CAG vs AGC) are permitted candidates) — https://github.com/morisUtokyo/uTR
- **mTR** (Precomputes long representative units (> 20 nt) that can be fed to uTR via -u when candidate units cannot be enumerated naively) — https://github.com/morisUtokyo/mTR

## Examples

```
python -c "U=[];V=['ACCG','AC','CG','CGAC'];print(min(V,key=lambda u:pen(U+[u],D)))"  # first iteration must yield 'ACCG' with extended penalty 7
```

## Evaluation signals

- First greedy iteration selects ACCG with extended penalty 7 (vs. 11 for AC/CG and 10 for CGAC)
- Loop terminates with U={ACCG} — no further candidate reduces the extended penalty
- greedy_selection_report.txt exists and contains the full candidate/penalty trace for every iteration
- ACCG fully covers decomposition D1 of the running example, so residual uncovered decomposition length is 0 at termination

## Limitations

- Rotated unit strings (e.g., CAG vs AGC) are equivalent; uTR tries to output a pattern whose first character matches the input string's first character, so rotation choice affects reported patterns but not the selected unit set
- The greedy selection is heuristic — it minimizes extended penalty step-wise and may not reach the global optimum on other inputs
- Default uTR mode trades simpler patterns with more mismatches against more complex patterns with fewer mismatches (-y) and shorter vs. longer units (-z), so verified behavior depends on which objective variant is being re-implemented

## Evidence

- [intro] The program uTR decomposes a DNA string into mosaic tandem repeats with different repeat units.: "The program uTR decomposes a DNA string into mosaic tandem repeats with different repeat units."
- [readme] Unit strings can be rotated to represent tandem repeating patterns.: "Unit strings can be rotated to represent tandem repeating patterns. For example, in the example above, CAG can be replaced with AGC."
- [readme] Occam's Razor trade-off between simple and complex decompositions: "The background to providing this mode is Occam's Razor on how to generate better decompositions; that is, a trade-off between simpler patterns with more mismatches and more complex patterns with"
