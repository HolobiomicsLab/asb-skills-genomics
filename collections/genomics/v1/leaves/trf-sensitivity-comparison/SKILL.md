---
name: trf-sensitivity-comparison
description: Use when evaluating how a standard TR detector (TRF) handles complex
  mosaic tandem repeats from long reads, or when validating uTR's claimed accuracy
  advantage over TRF and RepeatMasker on real sequence data.
when_to_use_negative:
- Input contains simulated mosaic TRs rather than real reads — use the test_public
  gendata/check_uTR/test.sh benchmark workflow instead
- TRs have units longer than 20 nt — uTR auto-estimates only units of length 20 nt
  or less; use mTR for long units
- The goal is variant-unit-level detail of an already estimated pattern — use vTR,
  not a TRF sensitivity comparison
edam_operation: http://edamontology.org/operation_0488
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
tools:
- name: uTR
  role: Decomposes the input DNA string into mosaic tandem repeat patterns (#Pat)
    used as the reference detection standard against TRF
  repo: https://github.com/morisUtokyo/uTR
- name: TRF
  role: Baseline tandem repeat detector run on the same realdata.fasta to record which
    of the five mosaic repeats it detects
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad185
    title: Decomposing mosaic tandem repeats accurately from long reads
  source_content_hash: sha256:d1a6825b30f6526a28a822f1e9f28e684658ee9feae5adb9f03ed09b5d40ecce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morisUtokyo/uTR
  tools:
  - TRF
  - uTR
  edam_operation: http://edamontology.org/operation_0488
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# trf-sensitivity-comparison

## Summary

Benchmark skill for running Tandem Repeats Finder (TRF) on the same real long-read FASTA (realdata/realdata.fasta with SAMD12, RFC1, KAZN, ZNF37A mosaic TRs) that uTR decomposes, to quantify TRF's lower sensitivity: TRF reportedly detects only the third repeat, the RFC1 (AAAAG)11 repeat, while uTR identifies all five mosaic patterns.

## When to use

Use when evaluating how a standard TR detector (TRF) handles complex mosaic tandem repeats from long reads, or when validating uTR's claimed accuracy advantage over TRF and RepeatMasker on real sequence data.

## When NOT to use

- Input contains simulated mosaic TRs rather than real reads — use the test_public gendata/check_uTR/test.sh benchmark workflow instead
- TRs have units longer than 20 nt — uTR auto-estimates only units of length 20 nt or less; use mTR for long units
- The goal is variant-unit-level detail of an already estimated pattern — use vTR, not a TRF sensitivity comparison

## Inputs

- realdata/realdata.fasta (long reads containing SAMD12, RFC1, KAZN, and ZNF37A mosaic tandem repeats)
- Optional locus information per read (e.g., chr8:118366813-118366928) for uTR's -l flag

## Outputs

- utr_realdata_report.txt with uTR #Pat decompositions and TRF detection outcome per locus
- uTR-annotated FASTA (#Info, #Pat, #Decomp annotations per read)

## How to apply

Obtain realdata/realdata.fasta from https://github.com/morisUtokyo/uTR, which contains reads with the reported mosaic patterns (AAAAT)23 (SAMD12/BAFME), (ATTTT)221(ATTTC)221(ATTTT)82, (ATTTT)613(ATTTC)320(ATTTT)5(ATTTC)130, (AAAAG)11 (RFC1), (AAAG)6(AG)11(AAAG)20 (KAZN), and (CTTTT)12(CTTGT)3(CTTTT)2 (ZNF37A). Run uTR with -f on the fasta and -o for the annotated output (default MAX_DIS_RATIO 0.3, units up to 20 nt auto-estimated), optionally with -l locus information, and record the #Pat annotation per read, allowing unit rotation (e.g., CAG vs AGC). Then run TRF on the identical input and record which of the five repeats TRF detects. The expected result, which validates the comparison, is that uTR's decompositions match all five reported patterns while TRF detects only the third listed repeat. Write per-locus uTR patterns and the TRF detection outcome to a report such as utr_realdata_report.txt.

## Related tools

- **uTR** (Decomposes the input DNA string into mosaic tandem repeat patterns (#Pat) used as the reference detection standard against TRF) — https://github.com/morisUtokyo/uTR
- **TRF** (Baseline tandem repeat detector run on the same realdata.fasta to record which of the five mosaic repeats it detects)

## Examples

```
uTR -f realdata/realdata.fasta -o realdata_annotated.fasta; TRF realdata/realdata.fasta 2 7 7 80 10 50 500 -d > trf_report.dat
```

## Evaluation signals

- uTR #Pat annotations match all five reported mosaic patterns, allowing unit rotation (e.g., CAG vs AGC)
- TRF detects exactly one of the five repeats — the third listed one, the RFC1 (AAAAG)11 repeat — reproducing the reported sensitivity gap
- uTR #Info dissimilarity ratio stays at or below the default MAX_DIS_RATIO of 0.3 for detected repeats
- Report lists a per-locus detection outcome for both tools (SAMD12, RFC1, KAZN, ZNF37A reads) with no missing loci

## Limitations

- uTR automatically estimates tandem repeat units only of length 20 nt or less; longer units require mTR
- Default mode outputs a simpler pattern with more mismatches; -y gives a more complex pattern with fewer mismatches, and -z prioritizes longer units — pattern string can differ by mode
- Unit strings can be rotated to represent the same pattern, so string equality of #Pat against reported patterns is not the right match criterion
- The missing-signal note states the uTR version number is not stated, so exact reproducibility may depend on the repository checkout

## Evidence

- [methods] Running uTR on realdata.fasta confirms all five reported mosaic TRs are identified, while TRF detects only the third one (the RFC1 (AAAAG)11 repeat).: [evidence span withheld — anchor status: not_found]
- [readme] If it is not specified, uTR automatically estimates tandem repeat units of length 20 nt or less.: "If it is not specified, uTR automatically estimates tandem repeat units of length 20 nt or less."
- [readme] The default parameter is 0.3, which is set in uTR.h by: #define MAX_DIS_RATIO_DEFAULT 0.3: "The default parameter is 0.3, which is set in uTR.h by: #define MAX_DIS_RATIO_DEFAULT 0.3"
- [abstract] our program named uTR outperforms TRF and RepeatMasker in terms of prediction accuracy: "our program named uTR outperforms TRF and RepeatMasker in terms of prediction accuracy"
