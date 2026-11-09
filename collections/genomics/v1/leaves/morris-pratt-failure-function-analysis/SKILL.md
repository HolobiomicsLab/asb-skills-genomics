---
name: morris-pratt-failure-function-analysis
description: Use when enumerating candidate repeat-unit substrings of a DNA string
  for tandem repeat decomposition and you must filter out substrings whose proper
  suffix matches a prefix (self-overlapping candidates).
when_to_use_negative:
- The input is not a DNA string but raw sequencing reads that still need preprocessing,
  clustering, or basecalling before unit enumeration
- The goal is annotation of an existing tandem repeat pattern (e.g., variant-unit
  output) rather than de novo candidate unit enumeration
- Representative units longer than 20 nt are already known and should be supplied
  via -u (use mTR for long units) rather than re-derived by enumeration
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: uTR
  role: Implements Algorithm 2, which applies the Morris-Pratt OLi test together with
    the LEN test to enumerate repetitive non-self-overlapping unit candidates
  repo: https://github.com/morisUtokyo/uTR
- name: mTR
  role: Complementary tool for computing long units (> 20 nt) that fall outside uTR's
    automatic unit-estimation range
  repo: https://github.com/morisUtokyo/mTR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad185
    title: Decomposing mosaic tandem repeats accurately from long reads
  source_content_hash: sha256:86e5768d38e3db25990b448c8be87e9aaa09c61290401266ab816c46e8a7d598
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morisUtokyo/uTR
  tools:
  - mTR
  - uTR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# morris-pratt-failure-function-analysis

## Summary

Computes the Morris-Pratt failure-function array OLi for each suffix position of a DNA string, as used inside uTR's Algorithm 2 to certify that candidate repeat units are non-self-overlapping. Combined with the suffix-array-based LEN test, this enables O(n^2) enumeration of repetitive, non-self-overlapping substring candidates.

## When to use

Use when enumerating candidate repeat-unit substrings of a DNA string for tandem repeat decomposition and you must filter out substrings whose proper suffix matches a prefix (self-overlapping candidates).

## When NOT to use

- The input is not a DNA string but raw sequencing reads that still need preprocessing, clustering, or basecalling before unit enumeration
- The goal is annotation of an existing tandem repeat pattern (e.g., variant-unit output) rather than de novo candidate unit enumeration
- Representative units longer than 20 nt are already known and should be supplied via -u (use mTR for long units) rather than re-derived by enumeration

## Inputs

- Input DNA string S (e.g., the uTR running-example sequence ACCGACCGACCG, or a read from a FASTA file)
- Suffix array of S computed with SA-IS
- LCP array of S computed with Kasai's algorithm (for the companion LEN test)

## Outputs

- OLi failure-function array per suffix position i (periodic border information, OLi[k]=0 flagging non-self-overlapping lengths)
- LEN array of longest repeat extensions per starting position
- List of repetitive non-self-overlapping substrings with their (i,j) coordinates (e.g., written to repetitive_substrings.txt)

## How to apply

For each suffix starting position i of the input string S, build OLi with the Morris-Pratt failure function in O(n-i) time so that OLi[k]=0 indicates S[i,i+k) has no proper suffix that matches a prefix (non-self-overlapping). In parallel, build the suffix array of S with SA-IS in linear time, derive the LCP array with Kasai's algorithm, and set LEN[i]=max{lcp(i,j)|j!=i} so LEN[i]>=j-i confirms S[i,j) is repetitive. Enumerate all (i,j) substring pairs and retain a pair only when both tests pass (LEN[i]>=j-i and OLi[j-i]=0); for the running example S=ACCGACCGACCG, the 4-mer ACCG passes both tests (occurs three times, no self-overlap) and is returned as a unit. The rationale is that self-overlapping strings cannot tile a repeat locus unambiguously, so pruning them early keeps the overall enumeration O(n^2).

## Related tools

- **uTR** (Implements Algorithm 2, which applies the Morris-Pratt OLi test together with the LEN test to enumerate repetitive non-self-overlapping unit candidates) — https://github.com/morisUtokyo/uTR
- **mTR** (Complementary tool for computing long units (> 20 nt) that fall outside uTR's automatic unit-estimation range) — https://github.com/morisUtokyo/mTR

## Examples

```
uTR -f sample.fasta -o annotated.fasta
```

## Evaluation signals

- For S=ACCGACCGACCG, the emitted candidate list contains the 4-mer ACCG (occurs three times) and excludes self-overlapping strings whose proper suffix matches a prefix
- Every retained substring (i,j) satisfies both LEN[i]>=j-i and OLi[j-i]=0; any candidate failing one test indicates a bug
- Overall enumeration completes in O(n^2) time; per-position OLi construction takes O(n-i)
- The final unit-string output matches uTR's tandem repeat annotation convention (e.g., #Pat <CAG>19<CCG>38 style output for the sample sequence)

## Limitations

- uTR automatically estimates tandem repeat units of length 20 nt or less; longer units require mTR, so the enumeration as-is may miss long units
- Candidates that pass both tests but occur only once (subscript 1 in -y mode) are substrings rather than true repeats and must be interpreted cautiously
- A single representative unit threshold -r (default 0.3 dissimilarity ratio) determines whether a tandem repeat is output at all; stricter thresholds can suppress valid units

## Evidence

- [methods] OLi is built by the Morris-Pratt algorithm in O(n-i) per position: [evidence span withheld — anchor status: not_found]
- [methods] LEN computed from suffix array: [evidence span withheld — anchor status: not_found]
- [results] uTR running example ACCG passes both tests: [evidence span withheld — anchor status: not_found]
- [readme] README: uTR purpose and usage: "The program uTR decomposes a DNA string into mosaic tandem repeats with different repeat units."
- [readme] README: unit length limit: "If it is not specified, uTR automatically estimates tandem repeat units of length 20 nt or less."
