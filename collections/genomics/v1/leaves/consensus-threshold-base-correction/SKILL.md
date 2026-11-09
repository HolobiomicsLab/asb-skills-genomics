---
name: consensus-threshold-base-correction
description: Use when correcting hybrid (long + short read) datasets at the base level
  and you need a strict, high-confidence rule — e.g., PacBio long reads with ≥90%
  Illumina consensus at a questionable position — rather than majority voting or optimization.
when_to_use_negative:
- A questionable position falls below the η = 0.9 (90%) consensus threshold — such
  bases must be routed to HECIL's optimization-based correction framework, not quick-corrected
- You have only long reads and no short-read data — this is a hybrid correction rule
  requiring Illumina short-read alignments (unlike self-correcting methods such as
  HGAP)
- Simple majority voting is the intended rule — HECIL explicitly adopts a stricter-than-majority
  90% consensus cutoff, so this skill is wrong for majority-based correction
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: HECIL
  role: Implements the Quick Correction stage with the η = 0.9 consensus eligibility
    threshold (exposed as the -c CUTOFF_QC option of HECIL.py)
  repo: https://github.com/NDBL/HECIL
- name: BWA-MEM
  role: Aligns Illumina short reads to the long reads to identify questionable mismatch/insertion/deletion
    positions and the per-position short-read consensus fractions
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:41532c8c6596d4151fb677dc148185b089bf1eb7ea85a97771daef7bd7be059f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BWA-MEM
  - HECIL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# consensus-threshold-base-correction

## Summary

Implements HECIL's Quick Correction stage: at positions on long reads marked questionable (mismatches, insertions, deletions) from BWA-MEM alignment of Illumina short reads, the long-read base is directly replaced with the short-read consensus base only when at least 90% consensus (η = 0.9) is reached, bypassing the optimization-based correction framework.

## When to use

Use when correcting hybrid (long + short read) datasets at the base level and you need a strict, high-confidence rule — e.g., PacBio long reads with ≥90% Illumina consensus at a questionable position — rather than majority voting or optimization.

## When NOT to use

- A questionable position falls below the η = 0.9 (90%) consensus threshold — such bases must be routed to HECIL's optimization-based correction framework, not quick-corrected
- You have only long reads and no short-read data — this is a hybrid correction rule requiring Illumina short-read alignments (unlike self-correcting methods such as HGAP)
- Simple majority voting is the intended rule — HECIL explicitly adopts a stricter-than-majority 90% consensus cutoff, so this skill is wrong for majority-based correction

## Inputs

- Long reads file (e.g., PacBio FASTA/FASTQ)
- Illumina short reads (FASTQ)
- BWA-MEM alignment (SAM/BAM) of short reads against long reads
- η = 0.9 consensus eligibility threshold (parameter, e.g., -c CUTOFF_QC in HECIL.py)

## Outputs

- Long-read sequences with quick-corrected bases at ≥90% consensus positions (LongRead_Corrected.fasta)
- Set of remaining questionable positions (below η = 0.9) deferred to the optimization-based correction stage

## How to apply

Align Illumina short reads to the long reads with BWA-MEM and parse the SAM/BAM alignment to mark mismatch, insertion, and deletion positions on each long read as questionable. For each questionable position, collect the short reads spanning it and compute the fraction supporting the consensus base. Apply direct base replacement only when the strong-consensus eligibility threshold η = 0.9 (≥90% consensus) is met; this stricter-than-majority cutoff deliberately avoids hasty corrections on weakly supported positions. Leave all other questionable bases untouched at this stage — they are handed to HECIL's optimization-based correction framework, which uses a convex combination of normalized quality and similarity weights from the aligned short reads.

## Related tools

- **HECIL** (Implements the Quick Correction stage with the η = 0.9 consensus eligibility threshold (exposed as the -c CUTOFF_QC option of HECIL.py)) — https://github.com/NDBL/HECIL
- **BWA-MEM** (Aligns Illumina short reads to the long reads to identify questionable mismatch/insertion/deletion positions and the per-position short-read consensus fractions)

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- Output FASTQ (LongRead_Corrected.fasta) exists and contains the same number of long reads as the input
- Only questionable positions meeting the ≥90% consensus criterion (η = 0.9) were modified; all other positions remain untouched after the quick-correction pass
- Positions failing the η = 0.9 threshold are still flagged as questionable and passed to the optimization-based correction framework rather than being replaced
- Downstream correction metrics improve relative to the uncorrected reads (e.g., on S. cerevisiae S288C, PI rises from 93.8 uncorrected to 98.6 at iteration 1 and 98.9 at iteration 5)

## Limitations

- The 90% consensus cutoff is deliberately stricter than majority voting, so weakly supported positions get no direct correction and remain for the slower optimization-based stage
- Questionable positions are defined solely from mismatches, insertions, and deletions in the BWA-MEM alignment; errors not surfaced by that alignment will not be flagged
- Iterative improvement in HECIL overall is a conjecture without a rigorous mathematical proof, and multi-threading speedup was not yet implemented in the version described (workload was split into 16 concurrent parallel tasks instead)

## Evidence

- [methods] we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction: "we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction"
- [methods] For the remaining questionable bases, we employ an optimization-based correction framework.: "For the remaining questionable bases, we employ an optimization-based correction framework."
- [readme] python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out: "Example command: python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out"
