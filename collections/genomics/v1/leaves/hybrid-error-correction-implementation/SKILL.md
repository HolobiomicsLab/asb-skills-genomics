---
name: hybrid-error-correction-implementation
description: Use when noisy long reads (PacBio or Oxford Nanopore, errors up to 20-35%)
  must be corrected with high-quality Illumina short reads, and consensus thresholds
  or correction-policy selection must be reproduced.
when_to_use_negative:
- The long reads are already Canu-corrected and the goal is de novo assembly rather
  than short-read-based correction (though HECIL can further improve Canu-corrected
  reads, this is a distinct downstream workflow step)
- Only self-correction from long-read data is desired — HECIL is a hybrid algorithm
  that requires supplementary short-read sequencing data, unlike self-correcting algorithms
  such as HGAP
- Evaluation-only tasks such as k-mer counting (Jellyfish), reference alignment (BLASR),
  assembly quality assessment (QUAST, MUMmer dnadiff), which are downstream validation
  steps, not the correction itself
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3693
tools:
- name: HECIL
  role: Core hybrid error correction framework implementing Quick Correction (η =
    0.9 consensus threshold), optimization-based correction, and iterative learning
  repo: https://github.com/NDBL/HECIL
- name: BWA-MEM
  role: Aligns Illumina short reads against long reads to generate the alignments
    used to mark questionable positions and compute consensus support
- name: Samtools
  role: Dependency for processing the BWA alignment (SAM/BAM) during correction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:ea6eb5a09ad32a6873dbf8e05fcbfba0006982f301ba01a0993b137cc1401368
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BWA-MEM
  - HECIL
  - Samtools
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3693
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hybrid-error-correction-implementation

## Summary

Reconstructs HECIL's hybrid error correction pipeline, in which long reads from third-generation sequencers (PacBio, Oxford Nanopore) are corrected using aligned Illumina short reads via a Quick Correction stage plus an optimization-based correction framework with iterative learning.

## When to use

Use when noisy long reads (PacBio or Oxford Nanopore, errors up to 20-35%) must be corrected with high-quality Illumina short reads, and consensus thresholds or correction-policy selection must be reproduced.

## When NOT to use

- The long reads are already Canu-corrected and the goal is de novo assembly rather than short-read-based correction (though HECIL can further improve Canu-corrected reads, this is a distinct downstream workflow step)
- Only self-correction from long-read data is desired — HECIL is a hybrid algorithm that requires supplementary short-read sequencing data, unlike self-correcting algorithms such as HGAP
- Evaluation-only tasks such as k-mer counting (Jellyfish), reference alignment (BLASR), assembly quality assessment (QUAST, MUMmer dnadiff), which are downstream validation steps, not the correction itself

## Inputs

- Long reads in FASTA/FASTQ from PacBio or Oxford Nanopore sequencing (e.g. 33,360 filtered E. coli K-12 MG1655 PacBio reads)
- Short reads in FASTQ from Illumina sequencing (e.g. 22,720,100 E. coli sequences)
- Length of short reads (LENGTH_SR parameter, e.g. 202)

## Outputs

- Corrected long reads in FASTA (LongRead_Corrected.fasta), containing both corrected and uncorrected reads
- Confidence metrics assigned to prior corrections used in iterative learning

## How to apply

Align Illumina short reads to the long reads using BWA-MEM and mark positions on each long read with mismatches, insertions, and deletions as questionable. For Quick Correction, replace a questionable base with the aligned short-read consensus base only when at least 90% consensus (η = 0.9) is reached — a stricter threshold than simple majority voting — leaving all other questionable bases for the optimization-based correction framework, which selects a policy via an optimal combination of decision weights based on short-read base quality and mapping identity. Optionally run iterative learning: corrections with α > 95 are flagged high-confidence, long reads are realigned to the short reads in successive iterations, and iteration stops at a maximal number n of allowable iterations. Validate that the output FASTA contains the same number of long reads as the input and that only positions meeting the ≥90% consensus criterion were directly modified.

## Related tools

- **HECIL** (Core hybrid error correction framework implementing Quick Correction (η = 0.9 consensus threshold), optimization-based correction, and iterative learning) — https://github.com/NDBL/HECIL
- **BWA-MEM** (Aligns Illumina short reads against long reads to generate the alignments used to mark questionable positions and compute consensus support)
- **Samtools** (Dependency for processing the BWA alignment (SAM/BAM) during correction)

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- Output FASTQ/FASTA exists and contains the same number of long reads as the input (corrected plus uncorrected)
- Only questionable positions meeting the ≥90% (η = 0.9) consensus criterion among aligned short reads were modified by Quick Correction
- Corrected long reads yield a lower number of unique k-mers than the raw reads when counted with Jellyfish
- Higher alignment identity when corrected reads are aligned to a reference genome with BLASR
- Improvement in assembly metrics (QUAST) and whole-genome accuracy (MUMmer dnadiff) after assembling corrected reads with Canu

## Limitations

- Iterative improvement is only a conjecture with no mathematical proof; a rigorous framework would require possibly unrealistic assumptions
- Saturation behavior of the iterative procedure is unvalidated speculation
- Multi-threading speedup is not yet implemented; parallelization currently requires splitting the workload into 16 concurrent tasks manually
- Questionable positions failing the η = 0.9 quick-correction threshold must go through the optimization-based framework, which consumes extra execution time

## Evidence

- [methods] Quick Correction eligibility threshold: "we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction"
- [methods] Optimization-based fallback for remaining bases: "For the remaining questionable bases, we employ an optimization-based correction framework."
- [methods] High-confidence selection parameter: "Here we select α > 95 so that a small percentage of the optimal corrections are considered to be of high confidence."
- [methods] Iterative realignment learning: "We learn in successive iterations by realigning the updated long reads to the short reads."
- [readme] HECIL CLI usage from README: "python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out"
- [intro] Decision weights rationale: "selects a correction policy by leveraging an optimal combination of decision weights based on base quality and mapping identity of aligned short reads"
