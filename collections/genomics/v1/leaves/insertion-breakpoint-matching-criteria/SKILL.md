---
name: insertion-breakpoint-matching-criteria
description: Use when comparing insertion-calling tools against a known set of simulated
  insertions and you need an objective true-positive criterion plus per-tool precision/recall
  (optionally binned by insertion size).
when_to_use_negative:
- Evaluating insertions on real genomes without a simulated ground truth (e.g. CHM1/CHM13
  or NA12878 data) where true positives cannot be enumerated by breakpoint matching
- Calling insertions for discovery purposes — these thresholds are benchmark-matching
  criteria, not variant-calling filters
- Assessing short insertions that Novel-X does not target, since the analysis is restricted
  to insertions longer than 300 bp
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Novel-X
  role: Candidate insertion caller under evaluation on barcoded linked-reads
  repo: github.com/1dayac/Novel-X
- name: NUI
  role: Synthetic long-read baseline caller compared in the benchmark
- name: PopIns2
  role: Short-read baseline caller compared in the benchmark
- name: Pamir
  role: Short-read baseline caller; run on ART-simulated data because it cannot consume
    linked-reads
- name: LRSIM
  role: Simulates 65x-coverage barcoded synthetic long reads from the ground-truth
    genome
- name: ART
  role: Simulates matched standard short-read data for the Pamir baseline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:519ef0d50bdd78691fca580c79922270768e4d6bc2dfa39b841e8360458eb6d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ART
  - LRSIM
  - NUI
  - Novel-X
  - Pamir
  - PopIns2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# insertion-breakpoint-matching-criteria

## Summary

A benchmarking skill for scoring non-reference sequence (NRS) insertion calls against a simulated ground truth, matching calls to truth insertions when breakpoints differ by no more than 100 bp and insertion lengths differ by no more than 5%, then computing precision and recall per tool. It enables fair head-to-head comparison of Novel-X against NUI, PopIns2, and Pamir.

## When to use

Use when comparing insertion-calling tools against a known set of simulated insertions and you need an objective true-positive criterion plus per-tool precision/recall (optionally binned by insertion size).

## When NOT to use

- Evaluating insertions on real genomes without a simulated ground truth (e.g. CHM1/CHM13 or NA12878 data) where true positives cannot be enumerated by breakpoint matching
- Calling insertions for discovery purposes — these thresholds are benchmark-matching criteria, not variant-calling filters
- Assessing short insertions that Novel-X does not target, since the analysis is restricted to insertions longer than 300 bp

## Inputs

- Simulated ground-truth insertion set (2000 random 50-6000 bp sequences inserted into GRCh38)
- VCF insertion calls from Novel-X, NUI, PopIns2 (run on LRSIM-simulated 65x linked-read data aligned with LongRanger)
- VCF insertion calls from Pamir (run on ART-simulated 65x short-read data)
- Matching thresholds: 100 bp breakpoint tolerance, 5% insertion-length tolerance

## Outputs

- benchmark_results.csv with per-tool precision and recall
- Per-tool true-positive/false-positive assignments against the 2000 ground-truth insertions
- Size-bin breakdown of precision and recall per tool

## How to apply

First construct a ground-truth genome by inserting randomly generated DNA sequences (50-6000 bp) into GRCh38, then simulate 65x barcoded synthetic long reads with LRSIM; since Pamir cannot consume linked-reads, simulate matched standard short-read data at the same coverage with ART. Run each caller on its appropriate input (Novel-X, NUI, PopIns2 on the LongRanger-aligned linked-read BAM; Pamir on the ART short reads) and collect each tool's VCF calls. Match each call to the 2000 ground-truth insertions using the dual criterion: breakpoint positions differ by no more than 100 bp AND insertion lengths differ by no more than 5%; unmatched predictions count as false positives and unmatched truth insertions as false negatives. Compute precision and recall per tool, including breakdowns by insertion size bin, and write the comparison to benchmark_results.csv.

## Related tools

- **Novel-X** (Candidate insertion caller under evaluation on barcoded linked-reads) — github.com/1dayac/Novel-X
- **NUI** (Synthetic long-read baseline caller compared in the benchmark)
- **PopIns2** (Short-read baseline caller compared in the benchmark)
- **Pamir** (Short-read baseline caller; run on ART-simulated data because it cannot consume linked-reads)
- **LRSIM** (Simulates 65x-coverage barcoded synthetic long reads from the ground-truth genome)
- **ART** (Simulates matched standard short-read data for the Pamir baseline)

## Examples

```
match = [(t, c) for t in truth for c in calls if abs(t.breakpoint - c.breakpoint) <= 100 and abs(t.length - c.length) / t.length <= 0.05]; precision, recall = len(match)/len(calls), len(match)/2000
```

## Evaluation signals

- Reproduced precision should match reported values: Novel-X 0.999, NUI 0.997, PopIns2 0.999, Pamir 0.817
- Novel-X should show the best overall recall and precision among the four tools
- Every tool-call-to-truth match must satisfy both conditions: |breakpoint difference| <= 100 bp and |length difference| <= 5%
- Total truth insertions evaluated should equal 2000; recall cannot exceed matched/2000
- Size-bin recall and precision values should sum consistently with the overall per-tool counts

## Limitations

- The 100 bp / 5% matching criterion only works with an enumerable simulated ground truth; it cannot validate calls on real human genomes
- Novel-X requires a substantial number of distinct barcodes; small insertions often lack sufficient coverage for anchor assembly, so size-bin metrics for small insertions may reflect coverage limits rather than algorithmic failure
- Novel-X only outputs insertions longer than 300 bp with anchor-length sums exceeding 300 bp, so calls below this range cannot be benchmarked with this skill

## Evidence

- [other] Match calls with dual tolerance: [evidence span withheld — anchor status: not_found]
- [other] Reported benchmark precision values: [evidence span withheld — anchor status: not_found]
- [methods] Insertion length filter applied before benchmarking: [evidence span withheld — anchor status: not_found]
- [readme] Simulated supplementary VCFs used for benchmarking: [evidence span withheld — anchor status: external_doc]
