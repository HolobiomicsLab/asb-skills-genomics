---
name: tandem-repeat-sequence-simulation
description: Use when you need simulated mosaic tandem repeat reads with correct-answer
  patterns to benchmark TR decomposition tools (uTR vs TRF vs RepeatMasker) for accuracy
  or runtime across unit-count ranges and error rates.
when_to_use_negative:
- Input is real long-read data (e.g. realdata/realdata.fasta or SAND12 BAFME reads)
  rather than synthetic benchmarks — run the uTR/mTR/cTR analysis pipeline instead
- The goal is to decompose reads with repeat units longer than 20 nt — simulation
  with gendata targets short-unit mosaic TRs; use mTR for long units
- You only need accuracy evaluation of an already-generated dataset — use check_uTR/check_RepeatMasker
  rather than re-simulating
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: uTR
  role: Benchmarked tool whose runtime and accuracy are measured on the simulated
    mosaic TR datasets; run with -t for timing and default -r 0.3 dissimilarity threshold
  repo: https://github.com/morisUtokyo/uTR
- name: TRF
  role: Baseline tandem repeat finder run on the same synthetic datasets to compare
    execution time
- name: RepeatMasker
  role: Baseline run on synthetic datasets via 'repeatmasker -e hmmer -noint -pa 4
    -div 0 -xsmall $TR_file' for timing comparison
- name: gendata
  role: test_public code that generates simulated datasets with correct-answer mosaic
    repeat patterns
  repo: https://github.com/morisUtokyo/uTR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad185
    title: Decomposing mosaic tandem repeats accurately from long reads
  source_content_hash: sha256:2f2a5cc8e9f67fd068f9cba4f5655bdfbbeccf245c7a769747d4008e8e8299ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morisUtokyo/uTR
  tools:
  - RepeatMasker
  - TRF
  - gendata
  - uTR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# tandem-repeat-sequence-simulation

## Summary

Generate synthetic mosaic tandem repeat datasets with known ground-truth patterns (e.g. (AC)m(AG)n) using gendata from the uTR test_public suite, varying the m/n ranges and sequencing error rate to benchmark tools like uTR, TRF and RepeatMasker.

## When to use

Use when you need simulated mosaic tandem repeat reads with correct-answer patterns to benchmark TR decomposition tools (uTR vs TRF vs RepeatMasker) for accuracy or runtime across unit-count ranges and error rates.

## When NOT to use

- Input is real long-read data (e.g. realdata/realdata.fasta or SAND12 BAFME reads) rather than synthetic benchmarks — run the uTR/mTR/cTR analysis pipeline instead
- The goal is to decompose reads with repeat units longer than 20 nt — simulation with gendata targets short-unit mosaic TRs; use mTR for long units
- You only need accuracy evaluation of an already-generated dataset — use check_uTR/check_RepeatMasker rather than re-simulating

## Inputs

- Parameter specification: mosaic TR unit counts m and n ranges (e.g. 2-20, 2-50, 2-100, 2-200, 2-500) and sequencing error rate
- 1000 instances per (m, n) setting of mosaic tandem repeats such as (AC)m (AG)n
- Compiled gendata executable from the uTR test_public directory

## Outputs

- Simulated FASTA datasets of mosaic tandem repeats with known correct-answer patterns
- Per-read wall-clock times from uTR (via -t flag) for each synthetic dataset
- runtime_comparison.csv with a timing row per dataset plus error rate and average TR length columns
- accuracy_uTR.txt and accuracy_RepeatMasker.txt (when using test.sh batch mode)

## How to apply

Compile the test_public benchmark codes (gendata, check_uTR, parse_RepeatMasker, check_RepeatMasker) with make.sh. Run gendata to synthesize 1000 mosaic TR instances per parameter setting, sweeping unit-count ranges from m,n in 2-20 up to 2-500 and varying the sequence error rate, mirroring the paper's (AC)m(AG)n construction. Run uTR on each dataset with the -t flag so per-read wall-clock time is emitted, keeping the default maximum dissimilarity ratio of 0.3 (set by MAX_DIS_RATIO_DEFAULT in uTR.h; optionally overridden via -r). Run TRF and RepeatMasker ('repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file') on the same files for baseline timing, then aggregate per-dataset times into a table with error-rate and average-TR-length columns. Alternatively use test.sh to batch-generate and evaluate, writing accuracy_uTR.txt and accuracy_RepeatMasker.txt.

## Related tools

- **uTR** (Benchmarked tool whose runtime and accuracy are measured on the simulated mosaic TR datasets; run with -t for timing and default -r 0.3 dissimilarity threshold) — https://github.com/morisUtokyo/uTR
- **TRF** (Baseline tandem repeat finder run on the same synthetic datasets to compare execution time)
- **RepeatMasker** (Baseline run on synthetic datasets via 'repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file' for timing comparison)
- **gendata** (test_public code that generates simulated datasets with correct-answer mosaic repeat patterns) — https://github.com/morisUtokyo/uTR

## Examples

```
cd uTR/test_public && ./make.sh && ./test.sh
```

## Evaluation signals

- accuracy_uTR.txt must contain one row per (m, n) setting with 1000 instances generated, e.g. the '2 500' row shows 998 of 1000 patterns correctly predicted and average pattern length 1011
- runtime_comparison.csv contains a timing row for each synthetic dataset with error-rate and average-TR-length columns populated
- uTR wall-clock time is smaller than both TRF and RepeatMasker on the majority of the 48 datasets
- uTR runtime increases monotonically with sequencing error rate and with average mosaic TR length, matching the Fig. 1C/1D trends
- All benchmark executables compile cleanly with make.sh before generation

## Limitations

- uTR version number is not stated in the article's methods, so exact reproducibility requires noting the commit/release used
- Simulated datasets cover short repeat units (m and n sweep) and do not exercise units longer than 20 nt, which require mTR
- Only 998 of 1000 patterns were correctly estimated in the hardest setting (2-500), so perfect accuracy on simulated data is not an expected signal

## Evidence

- [methods] gendata: Codes for generating simulated datasets with correct answer mosaic repeat patterns.: "gendata: Codes for generating simulated datasets with correct answer mosaic repeat patterns."
- [methods] RepeatMasker command line: "repeatmasker -e hmmer -noint -pa 4 -div 0 -xsmall $TR_file"
- [readme] README on instance generation: "The first row shows that 1000 instances mosaic tandem repeats of the form (AC)m (AG)n are generated at random."
- [readme] README on m/n ranges: [evidence span withheld — anchor status: not_found]
- [readme] README on -t flag: "-t : Output the wall clock time to process each read in the input fasta file."
- [readme] README on compilation: "Use "make.sh" to compile all codes in the above subdirectories."
