---
name: tandem-repeat-decomposition-from-long-reads
description: Use when long-read FASTA reads are suspected to contain mosaic tandem
  repeats (e.g. disease-associated STRs like SAMD12 (AAAAT)23 or RFC1 (AAAAG)11) and
  you need unit-level decomposition plus a TRF comparison.
when_to_use_negative:
- 'Tandem repeat units longer than 20 nt need to be estimated: use mTR instead, since
  uTR without -u only automatically estimates units of length 20 nt or less'
- The goal is a concise pattern for reads already annotated with an estimated mosaic
  pattern — use vTR, which outputs series of variant units rather than a decomposition
- Input is not FASTA read data but simulated pattern checks or RepeatMasker parse
  outputs from the test_public pipeline (use the gendata/check_uTR/parse_RepeatMasker
  workflow instead)
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: uTR
  role: 'Primary decomposer: run with -f/-o (and optional -l/-u) on the input FASTA
    to emit #Pat mosaic tandem repeat annotations'
  repo: https://github.com/morisUtokyo/uTR
- name: TRF
  role: 'Benchmark comparator: run on the same realdata.fasta to record which mosaic
    repeats TRF detects (reported: only the third repeat)'
- name: cTR
  role: Optional pre-clustering of reads into groups of similar reads to generate
    smaller input FASTA files and reduce computation time
  repo: https://github.com/morisUtokyo/cTR
- name: mTR
  role: Computes long units (>20 nt) that uTR does not estimate automatically
  repo: https://github.com/morisUtokyo/mTR
- name: vTR
  role: Follow-up on uTR-annotated reads to output a series of variant units showing
    substitutions and indels
  repo: https://github.com/morisUtokyo/vTR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad185
    title: Decomposing mosaic tandem repeats accurately from long reads
  source_content_hash: sha256:2186e9a6d3555229ad8134cca58721d45e24f661dd09669d5852db4ccf0d1458
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morisUtokyo/uTR
  tools:
  - TRF
  - cTR
  - mTR
  - uTR
  - vTR
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# tandem-repeat-decomposition-from-long-reads

## Summary

Reproduces uTR-based decomposition of mosaic tandem repeats (different repeat units mixed in one string) from long-read FASTA data, benchmarking uTR against TRF. Grounded in the SAMD12, RFC1, KAZN and ZNF37A mosaic repeats from realdata/realdata.fasta of the morisUtokyo/uTR repository.

## When to use

Use when long-read FASTA reads are suspected to contain mosaic tandem repeats (e.g. disease-associated STRs like SAMD12 (AAAAT)23 or RFC1 (AAAAG)11) and you need unit-level decomposition plus a TRF comparison.

## When NOT to use

- Tandem repeat units longer than 20 nt need to be estimated: use mTR instead, since uTR without -u only automatically estimates units of length 20 nt or less
- The goal is a concise pattern for reads already annotated with an estimated mosaic pattern — use vTR, which outputs series of variant units rather than a decomposition
- Input is not FASTA read data but simulated pattern checks or RepeatMasker parse outputs from the test_public pipeline (use the gendata/check_uTR/parse_RepeatMasker workflow instead)

## Inputs

- realdata/realdata.fasta — FASTA reads containing the SAMD12, RFC1, KAZN, and ZNF37A mosaic tandem repeats
- Optional locus information strings (e.g. chr8:118366813-118366928) for the -l flag

## Outputs

- Annotated FASTA in which each read carries #Info, #Pat (tandem repeat pattern such as #Pat <CAG>19<CCG>38), #Decomp, and optional #Locus annotations
- utr_realdata_report.txt — per-locus uTR patterns and TRF detection outcomes
- TRF output on the same realdata.fasta input

## How to apply

Obtain realdata/realdata.fasta from https://github.com/morisUtokyo/uTR, which contains reads with the reported patterns (AAAAT)23 (SAMD12), (ATTTT)221(ATTTC)221(ATTTT)82, (ATTTT)613(ATTTC)320(ATTTT)5(ATTTC)130, (AAAAG)11 (RFC1), (AAAG)6(AG)11(AAAG)20 (KAZN), and (CTTTT)12(CTTGT)3(CTTTT)2 (ZNF37A). Run uTR with -f on realdata.fasta and -o to emit the annotated FASTA, letting uTR automatically estimate tandem repeat units of length 20 nt or less with the default MAX_DIS_RATIO 0.3 mismatch threshold; optionally pass -l locus information (e.g. chr8:118366813-118366928) so each output read carries a #Locus tag. Compare the #Pat annotations (e.g. #Pat <CAG>19<CCG>38) against the reported mosaic patterns, allowing unit rotation such as CAG vs AGC since uTR outputs a unit whose first character matches the input string. Run TRF on the same realdata.fasta and record which repeats TRF detects, reproducing the reported result that TRF detects only the third listed repeat (RFC1 (AAAAG)11). Optionally pre-cluster similar reads with cTR to reduce computation time, or use mTR for units longer than 20 nt and vTR to enumerate variant units with substitutions/indels. Write per-locus uTR patterns and TRF outcomes to utr_realdata_report.txt.

## Related tools

- **uTR** (Primary decomposer: run with -f/-o (and optional -l/-u) on the input FASTA to emit #Pat mosaic tandem repeat annotations) — https://github.com/morisUtokyo/uTR
- **TRF** (Benchmark comparator: run on the same realdata.fasta to record which mosaic repeats TRF detects (reported: only the third repeat))
- **cTR** (Optional pre-clustering of reads into groups of similar reads to generate smaller input FASTA files and reduce computation time) — https://github.com/morisUtokyo/cTR
- **mTR** (Computes long units (>20 nt) that uTR does not estimate automatically) — https://github.com/morisUtokyo/mTR
- **vTR** (Follow-up on uTR-annotated reads to output a series of variant units showing substitutions and indels) — https://github.com/morisUtokyo/vTR

## Examples

```
uTR -f realdata/realdata.fasta -o utr_realdata_annotated.fasta -l chr8:118366813-118366928
```

## Evaluation signals

- uTR #Pat decompositions match all five reported mosaic patterns (SAMD12 (AAAAT)23, the two ATTTT/ATTTC SAMD12-like reads, RFC1 (AAAAG)11, KAZN (AAAG)6(AG)11(AAAG)20, ZNF37A (CTTTT)12(CTTGT)3(CTTTT)2), allowing unit rotation (e.g. CAG vs AGC)
- TRF on the same realdata.fasta detects only the third listed repeat (RFC1 (AAAAG)11), reproducing the reported detection count
- uTR's #Info dissimilarity ratio stays below the default MAX_DIS_RATIO threshold of 0.3; no tandem repeat is reported for reads exceeding it
- Annotated output FASTA contains #Pat and #Decomp lines per read, plus #Locus chr8:118366813-118366928 when -l is supplied
- Synthetic accuracy check via test.sh shows near-perfect recovery (e.g. 998/1000 correct for m,n ranging 2–500) as sanity calibration of the pipeline

## Limitations

- uTR automatically estimates tandem repeat units of length 20 nt or less; long units require mTR
- Default decomposition (MAX_DIS_RATIO 0.3) trades simplicity for mismatches: the simple mode gave 12.9% dissimilarity where the -y mode achieved 4.1% with a more complex pattern — a trade-off the agent must choose deliberately
- uTR version number is not stated in the methods, so exact-version reproduction is not documented
- TRF fails to properly decompose most mosaic TRs, so TRF output is a negative-control comparison rather than a decomposition source

## Evidence

- [methods] uTR [-f input fasta file] [-o output fasta file with annotation] [-l locus information] [-u input representative unit string]: "uTR [-f input fasta file] [-o output fasta file with annotation] [-l locus information] [-u input representative unit string]"
- [readme] If it is not specified, uTR automatically estimates tandem repeat units of length 20 nt or less.: "If it is not specified, uTR automatically estimates tandem repeat units of length 20 nt or less."
- [readme] The default parameter is 0.3, which is set in uTR.h by: >#define MAX_DIS_RATIO_DEFAULT 0.3: "The default parameter is 0.3, which is set in uTR.h by: >#define MAX_DIS_RATIO_DEFAULT 0.3"
- [intro] The program uTR decomposes a DNA string into mosaic tandem repeats with different repeat units.: "The program uTR decomposes a DNA string into mosaic tandem repeats with different repeat units."
- [readme] Unit strings can be rotated to represent tandem repeating patterns. For example, in the example above, CAG can be replaced with AGC.: "Unit strings can be rotated to represent tandem repeating patterns. For example, in the example above, CAG can be replaced with AGC."
- [methods] cTR is useful because it clusters reads into groups of similar reads: "cTR is useful because it clusters reads into groups of similar reads"
- [methods] Long units (of length > 20nt, for example) can be computed by using mTR: "Long units (of length > 20nt, for example) can be computed by using mTR"
