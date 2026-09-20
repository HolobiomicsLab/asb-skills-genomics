---
name: benchmark-accuracy-table-interpretation
description: Use when benchmarking uTR against TRF or RepeatMasker on simulated mosaic
  tandem repeat datasets and you need to read, validate, or reproduce the per-row
  correct/total counts and average pattern lengths from the accuracy output tables.
when_to_use_negative:
- Input is real sequencing data (e.g. realdata/realdata.fasta reads of the SAMD12
  locus) rather than simulated datasets with known mosaic patterns — accuracy cannot
  be scored without ground truth
- Evaluating runtime/speed benchmarks rather than prediction accuracy, which requires
  the -t wall-clock timing output instead
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: uTR
  role: Decomposes synthetic mosaic tandem repeat reads; its predictions are scored
    row-by-row in accuracy_uTR.txt
  repo: https://github.com/morisUtokyo/uTR
- name: RepeatMasker
  role: Baseline comparison tool run with 'repeatmasker -e hmmer -noint -pa 4 -div
    0 -xsmall $TR_file'; output parsed and scored into accuracy_RepeatMasker.txt
- name: TRF
  role: Baseline comparison tool (version 4.09, parameters '2 7 7 80 10 10 1000 -h
    -ngs') scored under the same 0% and 2% allowance
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad185
    title: Decomposing mosaic tandem repeats accurately from long reads
  source_content_hash: sha256:433fb84962be4b78372f94e2fcbcfd13716f536c1171d3330357b68067f71bda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morisUtokyo/uTR
  tools:
  - RepeatMasker
  - TRF
  - uTR
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# benchmark-accuracy-table-interpretation

## Summary

Interprets the per-condition accuracy tables (accuracy_uTR.txt, accuracy_RepeatMasker.txt) produced by the uTR test_public batch, verifying that uTR matches or exceeds TRF and RepeatMasker on synthetic mosaic TR datasets as reported in Fig. 1A/1B.

## When to use

Use when benchmarking uTR against TRF or RepeatMasker on simulated mosaic tandem repeat datasets and you need to read, validate, or reproduce the per-row correct/total counts and average pattern lengths from the accuracy output tables.

## When NOT to use

- Input is real sequencing data (e.g. realdata/realdata.fasta reads of the SAMD12 locus) rather than simulated datasets with known mosaic patterns — accuracy cannot be scored without ground truth
- Evaluating runtime/speed benchmarks rather than prediction accuracy, which requires the -t wall-clock timing output instead

## Inputs

- accuracy_uTR.txt (per-row correct/total pattern counts and average pattern lengths)
- accuracy_RepeatMasker.txt (same schema, for RepeatMasker/TRF scoring)
- 48 simulated FASTA datasets (8 mosaic TR patterns x 6 error rates: 0/1/3/5/10/15%), each of 1000 strings
- Ground-truth mosaic repeat patterns from gendata

## Outputs

- Per-condition accuracy assessment (correct/total out of 1000 patterns per pattern-range row)
- Comparison against the Fig. 1A/1B trend and the 998/1000 reference anchor (range 2-500, average length 1011)

## How to apply

Compile the gendata/check/parse utilities with make.sh, then run the full batch with test.sh, which writes accuracy_uTR.txt and accuracy_RepeatMasker.txt. Each table row corresponds to one of 8 mosaic TR patterns (e.g. (AC)i(AG)j) under one of 6 substitution error rates (0/1/3/5/10/15%); the first two columns give the range of occurrence counts (e.g. '2 500'), the next two give correct predictions out of 1000, and the last column the average pattern length. Read accuracy under both 0% and 2% allowance for RepeatMasker/TRF (scored with check_RepeatMasker) and for uTR (scored with check_uTR against the ground-truth mosaic patterns). Validate that per-row counts reproduce the Fig. 1A/1B trend, using the documented anchor where m,n ranged 2-500 and uTR predicted 998 of 1000 patterns correctly with average length 1011.

## Related tools

- **uTR** (Decomposes synthetic mosaic tandem repeat reads; its predictions are scored row-by-row in accuracy_uTR.txt) — https://github.com/morisUtokyo/uTR
- **RepeatMasker** (Baseline comparison tool run with 'repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file'; output parsed and scored into accuracy_RepeatMasker.txt)
- **TRF** (Baseline comparison tool (version 4.09, parameters '2 7 7 80 10 10 1000 -h -ngs') scored under the same 0% and 2% allowance)

## Examples

```
sh make.sh && sh test.sh  # then inspect accuracy_uTR.txt and accuracy_RepeatMasker.txt
```

## Evaluation signals

- Row schema check: each line has 5 fields — min range, max range, correct count, total (1000), average pattern length
- Anchor row: for pattern (AC)m(AG)n with m,n in range 2-500, expect 998/1000 correct and average length 1011
- Trend check: accuracy decreases monotonically as sequencing error rate rises from 0% to 15%
- Comparative check: uTR correct/total counts match or exceed TRF and RepeatMasker across rows, especially for mosaic TRs with three or more unit series
- Allowance sensitivity: RepeatMasker/TRF scores under 2% allowance should equal or exceed their 0% allowance scores

## Limitations

- uTR version number is not stated in the paper, so exact reproducibility of the accuracy tables may vary with repository commits
- Accuracy drops as sequencing error rate increases; at 15% error the reported uTR advantage narrows
- The benchmark uses synthetic mosaic TRs with occurrence counts 10-200 (rows labelled 2-20 through 2-500); conclusions may not transfer to real long-read error profiles

## Evidence

- [readme] Use "test.sh" to perform all the steps in a batch manner. The script reports the accuracy of uTR and RepeatMasker in "accuracy_uTR.txt" and "accuracy_RepeatMasker.txt".: "Use "test.sh" to perform all the steps in a batch manner. The script reports the accuracy of uTR and RepeatMasker in "accuracy_uTR.txt" and "accuracy_RepeatMasker.txt"."
- [readme] In the sixth row, "2 500" shows that m and n ranged from 2 to 500, "998 1000" means 998 of 1000 patterns were correctly estimated: ""2 500" shows that m and n ranged from 2 to 500, "998 1000" means 998 of 1000 patterns were correctly estimated, and the last "1011" shows the average length."
- [abstract] we demonstrate that our program named uTR outperforms TRF and RepeatMasker in terms of prediction accuracy, this is especially true when mosaic TRs are more complex: "we demonstrate that our program named uTR outperforms TRF and RepeatMasker in terms of prediction accuracy, this is especially true when mosaic TRs are more complex"
- [methods] RepeatMasker open-4.0.7 run with 'repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file' and scored with check_RepeatMasker under 0% and 2% allowance: "repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file"
