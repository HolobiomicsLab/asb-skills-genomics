---
name: sam-mismatch-and-indel-detection
description: Use when hybrid-correcting long reads and you need per-base questionable-position
  calls derived from a BWA-MEM alignment of short reads, so downstream consensus or
  optimization-based correction only touches flagged positions.
when_to_use_negative:
- When there are no aligned short reads (i.e. self-correction such as HGAP-style multiple
  alignments) — this skill requires a short-read SAM/BAM alignment
- When positions are already annotated as questionable by a prior HECIL iteration
  (realign-and-remarking is an iterative step, not a re-parse of raw data)
- When the goal is de novo assembly of corrected reads (Canu/QUAST evaluation), not
  base-level questionable-position detection
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0196
tools:
- name: BWA
  role: Aligns Illumina short reads against long reads with BWA-MEM to produce the
    SAM/BAM parsed for mismatches and indels
  repo: https://github.com/NDBL/HECIL
- name: Samtools
  role: Used with BWA to produce/inspect the alignment (SAM/BAM) that HECIL parses
    to mark questionable positions
  repo: https://github.com/NDBL/HECIL
- name: HECIL
  role: Consumes the alignment to mark disagreement positions as questionable and
    route them to Quick Correction (η = 0.9) or Optimization-Based Correction
  repo: https://github.com/NDBL/HECIL
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:0bfd742a01be5126262735074385f43c9e876dde3ff16106061daf1a0ef1ada2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BWA
  - BWA-MEM
  - HECIL
  - Samtools
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sam-mismatch-and-indel-detection

## Summary

Parses a BWA-MEM SAM/BAM alignment of Illumina short reads against long reads to mark, per long read, every position of disagreement (mismatch, insertion, deletion) as questionable — the prerequisite step in HECIL that feeds the Quick Correction (η = 0.9 consensus) and Optimization-Based Correction stages.

## When to use

Use when hybrid-correcting long reads and you need per-base questionable-position calls derived from a BWA-MEM alignment of short reads, so downstream consensus or optimization-based correction only touches flagged positions.

## When NOT to use

- When there are no aligned short reads (i.e. self-correction such as HGAP-style multiple alignments) — this skill requires a short-read SAM/BAM alignment
- When positions are already annotated as questionable by a prior HECIL iteration (realign-and-remarking is an iterative step, not a re-parse of raw data)
- When the goal is de novo assembly of corrected reads (Canu/QUAST evaluation), not base-level questionable-position detection

## Inputs

- Long reads (e.g. PacBio/Oxford Nanopore) in FASTA/FASTQ
- Illumina short reads in FASTQ (e.g. ShortRead.fq)
- BWA-MEM SAM/BAM alignment of the short reads against the long reads (via Align_Corr.sh)

## Outputs

- Per-long-read questionable-position annotations (mismatches, insertions, deletions) passed to Quick Correction and Optimization-Based Correction
- Corrected long reads written to LongRead_Corrected.fasta, containing both corrected and uncorrected reads

## How to apply

Load the long reads together with the BWA-MEM SAM/BAM alignment (produced with BWA 0.7.15 and Samtools 0.1.18, as tested in the HECIL repo) of the Illumina short reads against the long reads. Parse the alignment record-by-record and mark positions on each long read with disagreements — mismatches, insertions, and deletions — as questionable. For each questionable position, collect the set of short reads that align over that position and compute the fraction supporting the consensus base; positions meeting the strict ≥90% consensus threshold (η = 0.9) become eligible for quick correction, while the remaining questionable bases are routed to the optimization-based correction framework. The rationale is that only documented disagreement positions should be candidates for modification, and only unambiguous consensus should be applied directly.

## Related tools

- **BWA** (Aligns Illumina short reads against long reads with BWA-MEM to produce the SAM/BAM parsed for mismatches and indels) — https://github.com/NDBL/HECIL
- **Samtools** (Used with BWA to produce/inspect the alignment (SAM/BAM) that HECIL parses to mark questionable positions) — https://github.com/NDBL/HECIL
- **HECIL** (Consumes the alignment to mark disagreement positions as questionable and route them to Quick Correction (η = 0.9) or Optimization-Based Correction) — https://github.com/NDBL/HECIL

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- Every long read has questionable positions marked exactly where the SAM/BAM alignment reports a mismatch, insertion, or deletion; positions with no disagreement are untouched
- The output FASTA (LongRead_Corrected.fasta) contains the same number of long reads as the input, including uncorrected reads
- Only questionable positions meeting the ≥90% consensus (η = 0.9) criterion are directly replaced; all other questionable positions are left for the optimization-based framework
- The alignment was produced with the documented toolchain (BWA 0.7.15, Samtools 0.1.18) before parsing
- Executable flags resolved — README troubleshooting requires chmod +x on bwa, samtools, and Align_Corr.sh if the pipeline fails to run

## Limitations

- The quick-correction path only fires at the stricter η = 0.9 consensus threshold, so many questionable positions are left to the slower optimization-based framework
- Multi-threading speedup is not implemented in this version; HECIL workloads must be split into separate concurrent tasks manually (e.g. 16 tasks)
- If cloned scripts/binaries cannot execute, permissions must be fixed with chmod +x bwa, samtools, Align_Corr.sh

## Evidence

- [methods] Parse the alignment to mark positions on each long read with disagreements (mismatches, insertions, and deletions) as questionable.: [evidence span withheld — anchor status: not_found]
- [methods] we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction: "we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction"
- [readme] BWA (tested on version 0.7.15),
Samtools (tested on version 0.1.18),
Python (tested on version 2.7.8): "BWA (tested on version 0.7.15),
Samtools (tested on version 0.1.18),
Python (tested on version 2.7.8)"
- [readme] python HECIL.py [-h] -l LR -s SR -len LENGTH_SR -o OUTPUT [-lc LOW_CONFIDENCE] [-c CUTOFF_QC] [-k CONFIDENCE_THRESHOLD]: "python HECIL.py [-h] -l LR -s SR -len LENGTH_SR -o OUTPUT [-lc LOW_CONFIDENCE] [-c CUTOFF_QC] [-k CONFIDENCE_THRESHOLD]"
