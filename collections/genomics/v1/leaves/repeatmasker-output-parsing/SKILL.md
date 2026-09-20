---
name: repeatmasker-output-parsing
description: Use when RepeatMasker has been run on simulated tandem repeat FASTA files
  (e.g. with 'repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file') and the
  raw predictions must be converted to pattern-level counts scored against known mosaic
  TR ground truth.
when_to_use_negative:
- The goal is to score uTR's own decompositions — use check_uTR instead of parse_RepeatMasker/check_RepeatMasker
- The RepeatMasker run was executed with parameters other than the benchmark settings
  ('-e hmmer -noint -pa 4 -div 0 -xsmall'), since the checker's allowances were calibrated
  for this configuration
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: RepeatMasker
  role: Source tool whose output is parsed; run as 'repeatmasker -e hmmer -noint -pa
    4 -div 0 -xsmall $TR_file' on simulated datasets
- name: parse_RepeatMasker
  role: Utility in test_public that parses the RepeatMasker output after entering
    the simulated datasets into RepeatMasker
  repo: https://github.com/morisUtokyo/uTR
- name: check_RepeatMasker
  role: Scores parsed RepeatMasker predictions against correct-answer patterns under
    0% and 2% allowance
  repo: https://github.com/morisUtokyo/uTR
- name: uTR
  role: Comparator tool whose accuracy (accuracy_uTR.txt) is benchmarked against the
    parsed RepeatMasker results
  repo: https://github.com/morisUtokyo/uTR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad185
    title: Decomposing mosaic tandem repeats accurately from long reads
  source_content_hash: sha256:9b5b32f50f8455882829d474deb6a43752d4d4fbd3c8b8993d4f5e80fc208529
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morisUtokyo/uTR
  tools:
  - RepeatMasker
  - TRF
  - check_RepeatMasker
  - parse_RepeatMasker
  - uTR
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# repeatmasker-output-parsing

## Summary

Parse the raw output of RepeatMasker runs on simulated mosaic tandem repeat FASTA datasets and score the parsed predictions against ground-truth patterns, enabling benchmarking of RepeatMasker's prediction accuracy against uTR in the uTR test_public pipeline.

## When to use

Use when RepeatMasker has been run on simulated tandem repeat FASTA files (e.g. with 'repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file') and the raw predictions must be converted to pattern-level counts scored against known mosaic TR ground truth.

## When NOT to use

- The goal is to score uTR's own decompositions — use check_uTR instead of parse_RepeatMasker/check_RepeatMasker
- The RepeatMasker run was executed with parameters other than the benchmark settings ('-e hmmer -noint -pa 4 -div 0 -xsmall'), since the checker's allowances were calibrated for this configuration

## Inputs

- RepeatMasker open-4.0.7 raw output file (from running RepeatMasker on a simulated $TR_file FASTA of 1000 mosaic TR strings)
- Ground-truth mosaic tandem repeat patterns from the gendata-simulated datasets (e.g. (AC)i(AG)j, (AAAAAG)i(AAAGAGAGGGAAAAG)j(AGGGG)k)
- check_RepeatMasker scoring utility compiled from test_public

## Outputs

- Parsed RepeatMasker predictions in a form scoreable by check_RepeatMasker
- accuracy_RepeatMasker.txt with per-condition correct/total pattern counts and average pattern lengths at 0% and 2% allowance

## How to apply

After generating each of the 48 simulated datasets (8 TR patterns x 6 error rates of 0/1/3/5/10/15%, 1000 strings each) with gendata, run RepeatMasker open-4.0.7 on the file named $TR_file using 'repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file'. Compile the parse_RepeatMasker utility from the test_public directory with make.sh, then apply it to the RepeatMasker output to extract predicted repeat regions/units. Score the parsed predictions with check_RepeatMasker under both 0% and 2% allowance to obtain correct/total pattern counts and average pattern lengths. The rationale is that RepeatMasker's native output format is not directly comparable to mosaic TR patterns, so parsing is required before accuracy can be compared row-by-row against accuracy_uTR.txt benchmarks.

## Related tools

- **RepeatMasker** (Source tool whose output is parsed; run as 'repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file' on simulated datasets)
- **parse_RepeatMasker** (Utility in test_public that parses the RepeatMasker output after entering the simulated datasets into RepeatMasker) — https://github.com/morisUtokyo/uTR
- **check_RepeatMasker** (Scores parsed RepeatMasker predictions against correct-answer patterns under 0% and 2% allowance) — https://github.com/morisUtokyo/uTR
- **uTR** (Comparator tool whose accuracy (accuracy_uTR.txt) is benchmarked against the parsed RepeatMasker results) — https://github.com/morisUtokyo/uTR

## Examples

```
repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file && ./parse_RepeatMasker < RepeatMasker_output && ./check_RepeatMasker
```

## Evaluation signals

- accuracy_RepeatMasker.txt is produced with rows formatted as 'range correct total avg_length', mirroring the accuracy_uTR.txt schema (e.g. '2 500 998 1000 1011')
- Parsed-and-scored counts reproduce the Fig. 1A/1B trend that RepeatMasker's prediction accuracy drops as sequencing error rate increases from 0% to 15%
- RepeatMasker correct/total counts at the 0% and 2% allowance levels are reported for both allowances, allowing sensitivity comparison
- Results show RepeatMasker underperforming uTR in most cases, especially for mosaic TRs with three or more series of units

## Limitations

- RepeatMasker is not designed to solve the mosaic TR problem and often fails to properly decompose mosaic tandem repeats, so parsed predictions will undercount complex patterns
- The uTR version used in the benchmark is not stated in the article, so exact reproducibility of comparative numbers may vary across tool versions
- Accuracy degrades monotonically as sequencing error rate increases, so parsed counts at high error rates (10-15%) reflect degraded RepeatMasker performance rather than parsing errors

## Evidence

- [methods] parse_RepeatMasker output parsing: "parse_RepeatMasker: Codes for parsing the output of RepeaseMasker after entering the simulated datasets (named $TR_file) into RepeatMasker"
- [methods] RepeatMasker command: "repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file"
- [readme] batch reporting schema: "Use "test.sh" to perform all the steps in a batch manner. The script reports the accuracy of uTR and RepeatMasker in "accuracy_uTR.txt" and "accuracy_RepeatMasker.txt"."
- [abstract] benchmark outperformance: "our program named uTR outperforms TRF and RepeatMasker in terms of prediction accuracy"
