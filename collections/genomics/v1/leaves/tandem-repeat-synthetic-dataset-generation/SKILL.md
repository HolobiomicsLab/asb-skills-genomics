---
name: tandem-repeat-synthetic-dataset-generation
description: 'Use when you need ground-truth benchmark data for evaluating tandem
  repeat decomposition tools: simulate mosaic TRs of known patterns with controlled
  occurrence ranges and substitution error rates.'
when_to_use_negative:
- You need to characterize mosaic TRs in real long reads (use uTR directly, optionally
  with cTR pre-clustering and mTR for long units) rather than benchmark decomposition
  accuracy
- Input data already has ground-truth repeat decomposition (use check_uTR/check_RepeatMasker
  scoring instead of generating new simulations)
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0080
tools:
- name: uTR
  role: Repository hosting the test_public/gendata utility that generates the simulated
    mosaic TR datasets; also the primary decomposition tool benchmarked on them
  repo: https://github.com/morisUtokyo/uTR
- name: RepeatMasker
  role: Baseline decomposition tool run on the generated simulated datasets for accuracy
    comparison
- name: TRF
  role: Baseline tandem repeat finder (version 4.09, parameters '2 7 7 80 10 10 1000
    -h -ngs') run on the generated datasets for accuracy comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad185
    title: Decomposing mosaic tandem repeats accurately from long reads
  source_content_hash: sha256:2623ce92a94199052dbe4ba5dfc76f3ac04c7d9fa58fb7787020ec3ebb732054
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morisUtokyo/uTR
  tools:
  - RepeatMasker
  - TRF
  - uTR
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# tandem-repeat-synthetic-dataset-generation

## Summary

Generates synthetic mosaic tandem repeat benchmark datasets using the gendata utility from the uTR repository, producing FASTA files of simulated strings built from known mosaic TR patterns (e.g. (AC)i(AG)j) with injected substitution errors, to provide ground truth for evaluating decomposition tools like uTR, TRF, and RepeatMasker.

## When to use

Use when you need ground-truth benchmark data for evaluating tandem repeat decomposition tools: simulate mosaic TRs of known patterns with controlled occurrence ranges and substitution error rates.

## When NOT to use

- You need to characterize mosaic TRs in real long reads (use uTR directly, optionally with cTR pre-clustering and mTR for long units) rather than benchmark decomposition accuracy
- Input data already has ground-truth repeat decomposition (use check_uTR/check_RepeatMasker scoring instead of generating new simulations)

## Inputs

- Mosaic tandem repeat pattern definitions from the paper's Section 3.1 (e.g. (AC)i(AG)j, (AAAAAG)i(AAAGAGAGGGAAAAG)j(AGGGG)k)
- Occurrence-count ranges for repeat units (e.g. 10-200)
- Sequencing substitution error rates: 0, 1, 3, 5, 10, 15%

## Outputs

- 48 simulated FASTA datasets (8 TR patterns x 6 error rates), each with 1000 strings
- Ground-truth mosaic repeat pattern annotations per simulated string
- Per-condition correct/total pattern counts and average pattern lengths written to accuracy_uTR.txt and accuracy_RepeatMasker.txt

## How to apply

Compile the gendata, check, and parse utilities in the uTR repo's test_public directory with make.sh. Use gendata to synthesize 8 x 6 = 48 simulated FASTA datasets, each containing 1000 strings, from the Section 3.1 mosaic TR patterns (e.g. (AC)i(AG)j and (AAAAAG)i(AAAGAGAGGGAAAAG)j(AGGGG)k) with occurrence counts drawn from ranges such as 10-200, injecting substitution errors at 0/1/3/5/10/15%. Name each simulated dataset $TR_file so downstream tools (RepeatMasker, TRF) and the checking scripts can consume them. Run the whole batch with test.sh, which writes per-condition correct/total pattern counts and average pattern lengths to accuracy_uTR.txt and accuracy_RepeatMasker.txt. The rationale is that widely used tools are not designed for mosaic TRs, so a controlled synthetic benchmark with known answer patterns is needed to quantify prediction accuracy.

## Related tools

- **uTR** (Repository hosting the test_public/gendata utility that generates the simulated mosaic TR datasets; also the primary decomposition tool benchmarked on them) — https://github.com/morisUtokyo/uTR
- **RepeatMasker** (Baseline decomposition tool run on the generated simulated datasets for accuracy comparison)
- **TRF** (Baseline tandem repeat finder (version 4.09, parameters '2 7 7 80 10 10 1000 -h -ngs') run on the generated datasets for accuracy comparison)

## Examples

```
cd test_public && ./make.sh && ./test.sh  # then inspect accuracy_uTR.txt and accuracy_RepeatMasker.txt
```

## Evaluation signals

- 48 datasets are produced: 8 mosaic TR patterns x 6 substitution error rates (0/1/3/5/10/15%), each of 1000 strings
- The reference anchor row for pattern (AC)m(AG)n at range 2-500 shows 998/1000 correct uTR predictions with average pattern length 1011
- Per-row counts in accuracy_uTR.txt and accuracy_RepeatMasker.txt reproduce the Fig. 1A/1B trend: uTR matches or exceeds TRF and RepeatMasker, with accuracy dropping as error rate increases
- uTR's advantage is largest on mosaic TRs containing three or more series of units

## Limitations

- Simulated datasets use substitution errors only, so indel-heavy error profiles of real sequencing platforms are not modeled
- Accuracy degrades as sequencing error rate increases, so conclusions are error-rate dependent
- The uTR version number is not stated in the paper's methods

## Evidence

- [methods] gendata: Codes for generating simulated datasets with correct answer mosaic repeat patterns.: "gendata: Codes for generating simulated datasets with correct answer mosaic repeat patterns."
- [methods] In the directory named "test_public", codes for generating simulated datasets and checking the accuracy of uTR and RepeatMasker are available: "In the directory named "test_public", codes for generating simulated datasets and checking the accuracy of uTR and RepeatMasker are available"
- [readme] Use "make.sh" to compile all codes in the above subdirectories.: "Use "make.sh" to compile all codes in the above subdirectories."
- [readme] In the second row, "2 20" implies that values of variables m and n ranged from 2 to 20, "1000 1000" means that 1000 of 1000 mosaic tandem repeat patterns were predicted correctly: ""2 20" implies that values of variables m and n ranged from 2 to 20, "1000 1000" means that 1000 of 1000 mosaic tandem repeat patterns were predicted correctly"
