---
name: tandem-repeat-unit-selection
description: Use when a DNA read (e.g. S=ACCGACCGACCG) must be decomposed into mosaic
  tandem repeat units and you need a principled, penalty-driven criterion for choosing
  which unit strings to include in the decomposition.
when_to_use_negative:
- When units longer than 20 nt are needed — uTR auto-estimates units of length 20
  nt or less only; use mTR for long units
- When the input already has a known representative unit — pass it via uTR's -u flag
  instead of re-running greedy selection
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0102
tools:
- name: uTR
  role: Implements the greedy unit-selection loop and candidate enumeration for mosaic
    tandem repeat decomposition
  repo: https://github.com/morisUtokyo/uTR
- name: mTR
  role: Computes long repeat units (>20 nt) that can be fed to uTR as representative
    units
  repo: https://github.com/morisUtokyo/mTR
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad185
    title: Decomposing mosaic tandem repeats accurately from long reads
  source_content_hash: sha256:a5f7b1647fe050b2ba9cfc18c618c23f3dbaaa30fdf5f9f938e5fb4b55d79fb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morisUtokyo/uTR
  tools:
  - mTR
  - uTR
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# tandem-repeat-unit-selection

## Summary

Selects repeat-unit strings for mosaic tandem repeat decomposition by first enumerating candidate substrings that are simultaneously repetitive (suffix-array LEN test) and non-self-overlapping (Morris-Pratt OLi test), then running uTR's greedy loop that iteratively adds the unit minimizing an extended penalty until no candidate reduces it.

## When to use

Use when a DNA read (e.g. S=ACCGACCGACCG) must be decomposed into mosaic tandem repeat units and you need a principled, penalty-driven criterion for choosing which unit strings to include in the decomposition.

## When NOT to use

- When units longer than 20 nt are needed — uTR auto-estimates units of length 20 nt or less only; use mTR for long units
- When the input already has a known representative unit — pass it via uTR's -u flag instead of re-running greedy selection

## Inputs

- Input DNA string S (e.g. S=ACCGACCGACCG) or FASTA read file
- Candidate unit set V (repetitive, non-self-overlapping substrings; rotated unit strings permitted)

## Outputs

- Selected unit set U (e.g. U={ACCG} for the running example)
- Selection trace recording each candidate and its extended penalty per iteration (greedy_selection_report.txt)
- List of repetitive non-self-overlapping substrings with (i,j) coordinates (repetitive_substrings.txt)

## How to apply

Enumerate candidate units with Algorithm 2: build the suffix array of S via SA-IS, derive the LCP array with Kasai's algorithm to compute LEN[i]=max{lcp(i,j)|j!=i}, and compute OLi per position with the Morris-Pratt failure function; retain substrings (i,j) only when LEN[i]>=j-i (repetitive) and OLi[j-i]=0 (non-self-overlapping), giving O(n^2) complexity. Initialize the selected set U as empty. At each greedy iteration, evaluate the extended penalty sum_{u in U}(|u|+o(u)) + sum_{s in D, s not in U}|s| — which trades total unit length and occurrence count against residual decomposition length — for every candidate unit added to U, and add the unit that minimizes it. Permit rotated unit strings (as uTR does for CAG/AGC) and terminate when no candidate reduces the penalty. On the running example S=ACCGACCGACCG, the first iteration selects ACCG with extended penalty 7 (versus 11 for AC/CG and 10 for CGAC), and the loop terminates with U={ACCG}.

## Related tools

- **uTR** (Implements the greedy unit-selection loop and candidate enumeration for mosaic tandem repeat decomposition) — https://github.com/morisUtokyo/uTR
- **mTR** (Computes long repeat units (>20 nt) that can be fed to uTR as representative units) — https://github.com/morisUtokyo/mTR

## Examples

```
uTR -f realdata/realdata.fasta -o annotated_reads.fasta
```

## Evaluation signals

- On S=ACCGACCGACCG the first greedy iteration must select ACCG with extended penalty 7, beating 11 for AC/CG and 10 for CGAC
- The greedy loop must terminate with U={ACCG}; report the full candidate/penalty trace in greedy_selection_report.txt
- Every enumerated candidate substring must pass both tests: LEN[i]>=j-i (repetitive) and OLi[j-i]=0 (non-self-overlapping)
- Candidate enumeration should run in O(n^2) overall (SA-IS suffix array, Kasai LCP, Morris-Pratt failure function)

## Limitations

- Greedy selection is a local heuristic: it stops as soon as no single candidate reduces the extended penalty, which may not yield the globally optimal unit set
- uTR automatically estimates tandem repeat units of length 20 nt or less; longer units require mTR
- There is a trade-off between simpler patterns with more mismatches and more complex patterns with fewer mismatches (Occam's Razor; -y mode), so the default decomposition may have a high dissimilarity ratio (default threshold 0.3)

## Evidence

- [intro] The program uTR decomposes a DNA string into mosaic tandem repeats with different repeat units.: "The program uTR decomposes a DNA string into mosaic tandem repeats with different repeat units."
- [readme] Unit strings can be rotated to represent tandem repeating patterns.: "Unit strings can be rotated to represent tandem repeating patterns. For example, in the example above, CAG can be replaced with AGC."
- [readme] -y : Produce a more complex pattern with fewer mismatches, while the default mode outputs a simpler pattern with more mismatches.: "-y : Produce a more complex pattern with fewer mismatches, while the default mode outputs a simpler pattern with more mismatches."
- [readme] If it is not specified, uTR automatically estimates tandem repeat units of length 20 nt or less.: "If it is not specified, uTR automatically estimates tandem repeat units of length 20 nt or less."
