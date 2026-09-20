---
name: sequencing-error-injection-simulation
description: Use when benchmarking tandem-repeat decomposition accuracy against a
  known ground truth, e.g. simulating (AC)m(AG)n-style mosaic TRs with substitution
  error injection before scoring predictions.
when_to_use_negative:
- Input is already real sequencing data without ground-truth mosaic repeat annotations
  — the simulation provides no benefit there; use realdata/realdata.fasta workflows
  instead
- The goal is detecting tandem repeats de novo in a genome rather than benchmarking
  decomposer accuracy under controlled error rates
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3524
tools:
- name: uTR
  role: Target decomposer under test; run on each simulated dataset with default MAX_DIS_RATIO_DEFAULT
    0.3 and scored by check_uTR against ground truth
  repo: https://github.com/morisUtokyo/uTR
- name: RepeatMasker
  role: Comparator decomposer run as 'repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall
    $TR_file'; output parsed with parse_RepeatMasker and scored with check_RepeatMasker
  repo: https://github.com/morisUtokyo/uTR
- name: TRF
  role: Comparator decomposer run with parameters '2 7 7 80 10 10 1000 -h -ngs' and
    scored under 0% and 2% allowance
- name: gendata
  role: test_public utility that generates simulated datasets with correct-answer
    mosaic repeat patterns and injects substitution errors
  repo: https://github.com/morisUtokyo/uTR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad185
    title: Decomposing mosaic tandem repeats accurately from long reads
  source_content_hash: sha256:00169b6dd66b7d06cb4d60702e92d83f520aaee894a94ae56c01308b3403b4cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morisUtokyo/uTR
  tools:
  - RepeatMasker
  - TRF
  - gendata
  - uTR
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# sequencing-error-injection-simulation

## Summary

Generates synthetic FASTA datasets of mosaic tandem repeats with substitution errors injected at controlled rates (0/1/3/5/10/15%) so that decomposers like uTR, RepeatMasker, and TRF can be scored against known ground-truth patterns.

## When to use

Use when benchmarking tandem-repeat decomposition accuracy against a known ground truth, e.g. simulating (AC)m(AG)n-style mosaic TRs with substitution error injection before scoring predictions.

## When NOT to use

- Input is already real sequencing data without ground-truth mosaic repeat annotations — the simulation provides no benefit there; use realdata/realdata.fasta workflows instead
- The goal is detecting tandem repeats de novo in a genome rather than benchmarking decomposer accuracy under controlled error rates

## Inputs

- Mosaic tandem repeat pattern definitions (e.g. (AC)m(AG)n, (AAAAAG)i(AAAGAGAGGGAAAAG)j(AGGGG)k) with unit occurrence count ranges
- Sequencing error rate grid (0, 1, 3, 5, 10, 15% substitution errors)
- test_public gendata utility from the uTR repository

## Outputs

- 48 simulated FASTA files (1000 strings each) with injected substitution errors
- Per-condition correct/total pattern counts and average pattern lengths in accuracy_uTR.txt and accuracy_RepeatMasker.txt (e.g. '2 500 998 1000 1011')
- Ground-truth mosaic pattern annotations used by check_uTR / check_RepeatMasker

## How to apply

Compile the gendata/check/parse utilities in the uTR repository's test_public directory with make.sh. Run gendata to synthesize 8 x 6 = 48 simulated FASTA datasets (1000 strings each) from Section 3.1 mosaic TR patterns such as (AC)i(AG)j and (AAAAAG)i(AAAGAGAGGGAAAAG)j(AGGGG)k, drawing unit occurrence counts from ranges like 2-20 up to 2-500 and injecting substitution errors at 0, 1, 3, 5, 10, and 15%. Run uTR on each dataset (default mismatch-ratio threshold 0.3 set as MAX_DIS_RATIO_DEFAULT in uTR.h) and score decompositions against the ground-truth mosaic patterns with check_uTR. Benchmark RepeatMasker open-4.0.7 ('repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file') and TRF 4.09 ('2 7 7 80 10 10 1000 -h -ngs') on the same datasets, scoring predictions under 0% and 2% allowance. Execute the whole batch via test.sh, which writes per-condition correct/total counts and average pattern lengths to accuracy_uTR.txt and accuracy_RepeatMasker.txt.

## Related tools

- **uTR** (Target decomposer under test; run on each simulated dataset with default MAX_DIS_RATIO_DEFAULT 0.3 and scored by check_uTR against ground truth) — https://github.com/morisUtokyo/uTR
- **RepeatMasker** (Comparator decomposer run as 'repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file'; output parsed with parse_RepeatMasker and scored with check_RepeatMasker) — https://github.com/morisUtokyo/uTR
- **TRF** (Comparator decomposer run with parameters '2 7 7 80 10 10 1000 -h -ngs' and scored under 0% and 2% allowance)
- **gendata** (test_public utility that generates simulated datasets with correct-answer mosaic repeat patterns and injects substitution errors) — https://github.com/morisUtokyo/uTR

## Examples

```
./test.sh  # compiles via make.sh, generates 48 simulated datasets, runs uTR/RepeatMasker/TRF, and writes accuracy_uTR.txt and accuracy_RepeatMasker.txt
```

## Evaluation signals

- The per-row counts in accuracy_uTR.txt and accuracy_RepeatMasker.txt reproduce the Fig. 1A/1B trend that uTR matches or exceeds TRF and RepeatMasker
- The '2 500' condition reports 998 of 1000 correctly predicted (AC)m(AG)n patterns with average length 1011, matching the README anchor row
- Simple two-unit conditions (e.g. '2 20' with (AC)m(AG)n) yield 1000/1000 correct predictions for uTR with average pattern length 42
- Accuracy is scored under both 0% and 2% allowance, and prediction accuracy degrades monotonically as the injected substitution error rate rises from 0% toward 15%
- Each of the 48 conditions contains exactly 1000 simulated strings

## Limitations

- Simulation injects only substitution errors; it does not model indels or other long-read error profiles
- Accuracy of all three tools drops as the sequencing error rate increases, so high-error conditions (>10%) may not separate tools well
- uTR automatically estimates repeat units only up to 20 nt; longer units require mTR, so the simulation benchmarks are limited to short-unit mosaics unless units are supplied via -u

## Evidence

- [methods] In the directory named "test_public", codes for generating simulated datasets and checking the accuracy of uTR and RepeatMasker are available: "In the directory named "test_public", codes for generating simulated datasets and checking the accuracy of uTR and RepeatMasker are available"
- [readme] Use "test.sh" to perform all the steps in a batch manner. The script reports the accuracy of uTR and RepeatMasker in "accuracy_uTR.txt" and "accuracy_RepeatMasker.txt".: "Use "test.sh" to perform all the steps in a batch manner. The script reports the accuracy of uTR and RepeatMasker in "accuracy_uTR.txt" and "accuracy_RepeatMasker.txt"."
- [readme] in the sixth row, "2 500" shows that m and n ranged from 2 to 500, "998 1000" means 998 of 1000 patterns were correctly estimated: "in the sixth row, "2 500" shows that m and n ranged from 2 to 500, "998 1000" means 998 of 1000 patterns were correctly estimated"
- [methods] repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file: "repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file"
- [abstract] Using synthetic benchmark data, uTR outperforms TRF and RepeatMasker in prediction accuracy, especially for more complex mosaic TRs: [evidence span withheld — anchor status: not_found]
