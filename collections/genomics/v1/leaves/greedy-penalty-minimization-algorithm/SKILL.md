---
name: greedy-penalty-minimization-algorithm
description: Use when implementing or reproducing uTR's greedy unit-selection step
  for mosaic tandem repeat decomposition — e.g., to validate that on S=ACCGACCGACCG
  the loop first selects ACCG (extended penalty 7) and terminates with U={ACCG}.
when_to_use_negative:
- The repeat units of interest are longer than 20 nt — uTR auto-estimates only units
  of length 20 nt or less, and long units should first be computed with mTR before
  running this selection loop
- You need the series of variant units within an already-estimated mosaic tandem repeat
  pattern (e.g., substitutions/indels per read) — use vTR, not the greedy selection
  loop
- You need to cluster reads into haplotype groups to reduce computation time — that
  is cTR's role, not this decomposition criterion
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0080
tools:
- name: uTR
  role: 'Reference implementation of the greedy unit-selection step; its penalty/rotation
    behavior (e.g., CAG vs AGC rotations, pattern output format #Pat) is the spec
    this skill reproduces'
  repo: https://github.com/morisUtokyo/uTR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad185
    title: Decomposing mosaic tandem repeats accurately from long reads
  source_content_hash: sha256:81bb3ef7ade03fc0542a4677300e600d5e3481958de3f5ecd2d803f3cc26dbf4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morisUtokyo/uTR
  tools:
  - uTR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# greedy-penalty-minimization-algorithm

## Summary

Reproduces uTR's greedy unit-selection loop, which builds a set U of repeat units by iteratively adding the candidate unit that minimizes an extended penalty — sum_{u in U}(|u|+o(u)) plus the residual length of unexplained decomposition strings — until no unit reduces it. This is the core decomposition criterion that lets uTR output mosaic tandem repeat patterns with different repeat units.

## When to use

Use when implementing or reproducing uTR's greedy unit-selection step for mosaic tandem repeat decomposition — e.g., to validate that on S=ACCGACCGACCG the loop first selects ACCG (extended penalty 7) and terminates with U={ACCG}.

## When NOT to use

- The repeat units of interest are longer than 20 nt — uTR auto-estimates only units of length 20 nt or less, and long units should first be computed with mTR before running this selection loop
- You need the series of variant units within an already-estimated mosaic tandem repeat pattern (e.g., substitutions/indels per read) — use vTR, not the greedy selection loop
- You need to cluster reads into haplotype groups to reduce computation time — that is cTR's role, not this decomposition criterion

## Inputs

- Input DNA string S (e.g., the mosaic tandem repeat string ACCGACCGACCG)
- Candidate unit set V, including rotated unit strings (e.g., CAG and AGC treated as equivalent rotations)
- Decomposition D of S into candidate substring strings used to compute residual penalty terms

## Outputs

- Selected unit set U (expected: U={ACCG} for the running example)
- greedy_selection_report.txt with the full selection trace (candidate and extended penalty at each greedy iteration)
- Mosaic tandem repeat pattern annotation in uTR's format, e.g., #Pat <CAG>19<CCG>38

## How to apply

Formalize the extended penalty as sum over selected units of (|u| + o(u), where o(u) is u's occurrence count) plus the summed length of decomposition strings s in D not covered by any selected unit; this trades unit length and occurrence count against residual unexplained sequence, mirroring uTR's Occam's-Razor trade-off between simpler patterns with more mismatches and more complex ones with fewer mismatches. Initialize U as empty and enumerate the candidate unit set V, allowing rotated unit strings (uTR permits CAG/AGC-style rotations and tries to output a pattern whose first character matches the input string). At each iteration, evaluate the extended penalty for every candidate added to the current U, add the unit that minimizes it, and stop when no candidate lowers the penalty. On the running example S=ACCGACCGACCG, the first iteration must select ACCG (penalty 7, beating 11 for AC/CG and 10 for CGAC), and the loop must terminate with U={ACCG} since no further unit reduces the penalty. Record the full selection trace (candidate, penalty per iteration) in greedy_selection_report.txt for validation.

## Related tools

- **uTR** (Reference implementation of the greedy unit-selection step; its penalty/rotation behavior (e.g., CAG vs AGC rotations, pattern output format #Pat) is the spec this skill reproduces) — https://github.com/morisUtokyo/uTR

## Examples

```
U=[]; while True: best=min(V,key=lambda u:ext_penalty(U+[u],D)); if ext_penalty(U+[u],D)>=ext_penalty(U,D): break; U.append(best)  # expect U=['ACCG'] for S='ACCGACCGACCG'
```

## Evaluation signals

- On S=ACCGACCGACCG, the first greedy iteration selects ACCG with extended penalty 7, strictly beating AC/CG (11) and CGAC (10)
- The loop terminates with U={ACCG} — no remaining candidate unit reduces the extended penalty
- Penalty values decompose correctly as sum(|u|+o(u)) over selected units plus summed lengths of uncovered decomposition strings
- Rotated unit strings (e.g., CAG/AGC) are treated as valid, interchangeable candidates in V
- greedy_selection_report.txt contains a complete candidate-and-penalty trace for every iteration, not just the final U

## Limitations

- Greedy selection is a heuristic: it terminates at a local minimum where no single unit reduces the extended penalty, which may differ from the global optimum
- uTR tries to output a tandem repeating pattern whose first character matches the input string, so rotation choice in V affects the reported pattern even when decompositions are equivalent
- There is an explicit trade-off between simpler patterns with more mismatches and more complex patterns with fewer mismatches (uTR's default vs -y mode); a single greedy criterion cannot satisfy both preferences simultaneously

## Evidence

- [methods] First-iteration selection of ACCG with extended penalty 7 in the running example: [evidence span withheld — anchor status: not_found]
- [readme] uTR decomposes strings into mosaic tandem repeats with different repeat units: "The program uTR decomposes a DNA string into mosaic tandem repeats with different repeat units."
- [readme] Candidate units may be rotated: "Unit strings can be rotated to represent tandem repeating patterns. For example, in the example above, CAG can be replaced with AGC."
- [readme] Objective trades alignment score against number of units: "our program optimizes the alignment score minus the number of units in the decomposition"
- [readme] Occam's Razor trade-off motivating the penalty design: "a trade-off between simpler patterns with more mismatches and more complex patterns with fewer mismatches"
