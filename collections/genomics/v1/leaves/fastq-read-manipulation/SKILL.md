---
name: fastq-read-manipulation
description: Use when hybrid-correcting noisy PacBio or Oxford Nanopore long reads
  against aligned Illumina short reads and a questionable position shows ≥90% (η =
  0.9) short-read consensus, so the base can be directly replaced without running
  the optimization-based correction step.
when_to_use_negative:
- A questionable position lacks ≥90% short-read consensus (η < 0.9) — quick correction
  must leave it untouched and defer it to the optimization-based correction framework
  instead.
- No hybrid short-read data (e.g., Illumina) is available — HECIL's quick correction
  depends on aligned short reads, unlike self-correcting algorithms such as HGAP.
- Input is already corrected long reads where the goal is assembly or k-mer evaluation
  (that downstream stage uses Canu/QUAST/Jellyfish, not quick correction).
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: HECIL
  role: 'Implements the quick-correction stage: marks questionable positions and applies
    direct consensus-based replacement at η = 0.9'
  repo: https://github.com/NDBL/HECIL
- name: BWA
  role: BWA-MEM aligns Illumina short reads onto the long reads to identify mismatches,
    insertions, and deletions as questionable positions
- name: Samtools
  role: Parses the BWA-MEM SAM/BAM alignment so the fraction of short reads supporting
    the consensus base can be computed per questionable position
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:36663f8c61b8fd334fbccfdaa1abde67150242868523e297e713ef5888e023e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BWA
  - BWA-MEM
  - HECIL
  - Samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fastq-read-manipulation

## Summary

Consensus-based quick correction of questionable long-read bases: after aligning Illumina short reads to noisy long reads, positions flagged by mismatches, insertions, or deletions are directly replaced with the short-read consensus only when at least 90% consensus (η = 0.9) is met; all other questionable bases are deferred to HECIL's optimization-based correction framework.

## When to use

Use when hybrid-correcting noisy PacBio or Oxford Nanopore long reads against aligned Illumina short reads and a questionable position shows ≥90% (η = 0.9) short-read consensus, so the base can be directly replaced without running the optimization-based correction step.

## When NOT to use

- A questionable position lacks ≥90% short-read consensus (η < 0.9) — quick correction must leave it untouched and defer it to the optimization-based correction framework instead.
- No hybrid short-read data (e.g., Illumina) is available — HECIL's quick correction depends on aligned short reads, unlike self-correcting algorithms such as HGAP.
- Input is already corrected long reads where the goal is assembly or k-mer evaluation (that downstream stage uses Canu/QUAST/Jellyfish, not quick correction).

## Inputs

- Long reads in FASTA/FASTQ format (e.g., PacBio long reads such as the E. coli K-12 MG1655 set of 33,360 filtered reads)
- Illumina short reads in FASTQ format (e.g., ShortRead.fq, 22,720,100 sequences in the E. coli dataset)
- BWA-MEM alignment (SAM/BAM) of the short reads against the long reads
- Short-read length parameter (-len, e.g., 202) and quick-correction consensus cutoff (-c CUTOFF_QC, η = 0.9)

## Outputs

- Corrected long reads in FASTA (LongRead_Corrected.fasta) containing all corrected and uncorrected long reads, where only η = 0.9 consensus positions were directly replaced
- Remaining questionable positions annotated for the downstream optimization-based correction framework

## How to apply

Align the Illumina short reads (e.g., FASTQ such as ShortRead.fq) onto the long reads (e.g., FASTA such as LongRead.fa) with BWA-MEM (tested with BWA 0.7.15) and parse the SAM/BAM with samtools (tested with 0.1.18). Mark every long-read position showing a mismatch, insertion, or deletion relative to the aligned short reads as questionable. For each questionable position, collect the short reads spanning it and compute the fraction supporting the consensus base. Replace the long-read base with that consensus only when the strong-consensus eligibility threshold η = 0.9 is met — deliberately stricter than simple majority voting — and pass all remaining questionable bases to HECIL's optimization-based correction framework (which then assigns high confidence to corrections with α > 95). The rationale is that positions with near-unanimous short-read agreement need no decision weighting, saving the optimization machinery for genuinely ambiguous bases.

## Related tools

- **HECIL** (Implements the quick-correction stage: marks questionable positions and applies direct consensus-based replacement at η = 0.9) — https://github.com/NDBL/HECIL
- **BWA** (BWA-MEM aligns Illumina short reads onto the long reads to identify mismatches, insertions, and deletions as questionable positions)
- **Samtools** (Parses the BWA-MEM SAM/BAM alignment so the fraction of short reads supporting the consensus base can be computed per questionable position)

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- The output LongRead_Corrected.fasta contains the same number of long reads as the input (both corrected and uncorrected reads are retained).
- Only questionable positions meeting the ≥90% (η = 0.9) short-read consensus criterion were modified; all other questionable positions are left untouched for the optimization-based stage.
- Each replaced base matches the majority consensus base of the short reads spanning that position in the BWA-MEM alignment.
- The alignment step is reproducible with the tested versions (BWA 0.7.15, samtools 0.1.18).
- Downstream, the corrected reads should yield fewer unique k-mers when counted with Jellyfish compared to the uncorrected input.

## Limitations

- Positions below the η = 0.9 consensus threshold are never fixed by quick correction; they require the separate optimization-based correction framework.
- After cloning the repository, bwa, samtools, and Align_Corr.sh may need chmod +x to execute, and HECIL is tested against Python 2.7.8 only.
- Multi-threading/speedup was not yet implemented in this version (noted only as a future extension), and iterative improvement of corrections remains a conjecture without proof.

## Evidence

- [methods] we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction: "we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction"
- [methods] For the remaining questionable bases, we employ an optimization-based correction framework.: "For the remaining questionable bases, we employ an optimization-based correction framework."
- [readme] The output containing all (corrected and uncorrected) long reads will be saved in LongRead_Corrected.fasta: "The output containing all (corrected and uncorrected) long reads will be saved in LongRead_Corrected.fasta"
- [readme] usage: python HECIL.py [-h] -l LR -s SR -len LENGTH_SR -o OUTPUT [-lc LOW_CONFIDENCE]: "usage: python HECIL.py [-h] -l LR -s SR -len LENGTH_SR -o OUTPUT [-lc LOW_CONFIDENCE]"
- [abstract] determines a correction policy for erroneous long reads, based on optimal combinations of decision weights obtained from short read alignments: "determines a correction policy for erroneous long reads, based on optimal combinations of decision weights obtained from short read alignments"
