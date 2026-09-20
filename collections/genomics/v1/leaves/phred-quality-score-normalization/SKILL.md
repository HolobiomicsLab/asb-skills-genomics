---
name: phred-quality-score-normalization
description: Use when computing per-short-read decision weights for hybrid long-read
  error correction, where raw PHRED scores from the FASTQ must be scaled to [0,1]
  relative to the -quality read at an alignment position.
when_to_use_negative:
- During quick correction, which relies on ≥90% (η = 0.9) short-read consensus rather
  than quality-weighted optimization
- When short reads lack PHRED quality scores (e.g., FASTA-only input), since the normalization
  depends on per-base quality strings
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3673
tools:
- name: HECIL
  role: Implements the optimization-based correction stage that computes the normalized
    quality weight and combines it with the similarity weight to select corrections
  repo: https://github.com/NDBL/HECIL
- name: BWA-MEM
  role: Aligns Illumina short reads to long reads to identify questionable bases and
    the set of aligning reads whose PHRED scores are extracted
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:1a29e59780432e785eea99fa7e1fb823c8edb527326b6a75dbe5267e4a941c87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BWA-MEM
  - HECIL
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# phred-quality-score-normalization

## Summary

Normalizes PHRED quality scores of short reads aligned to a questionable long-read base by dividing each score by the maximum quality among the aligning reads, yielding the decision weight ω̂_q used in HECIL's optimization-based correction.

## When to use

Use when computing per-short-read decision weights for hybrid long-read error correction, where raw PHRED scores from the FASTQ must be scaled to [0,1] relative to the best-quality read at an alignment position.

## When NOT to use

- During quick correction, which relies on ≥90% (η = 0.9) short-read consensus rather than quality-weighted optimization
- When short reads lack PHRED quality scores (e.g., FASTA-only input), since the normalization depends on per-base quality strings

## Inputs

- Short-read-to-long-read alignments from BWA-MEM marking questionable bases (mismatches, insertions, deletions)
- Illumina short reads in FASTQ format with PHRED quality strings
- Long reads (e.g., PacBio/Sequel E. coli reads) in FASTA format

## Outputs

- Normalized quality weight ω̂_q^k per aligning short read per questionable base
- Contribution to cost J_k = 1/2*(ω̂_q^k + ω̂_s^k) used for argmax correction-policy selection
- Corrected long reads written to LongRead_Corrected.fasta and per-correction records in corrected_bases.tsv

## How to apply

For each base on a long read flagged as questionable (mismatch, insertion, or deletion against BWA-MEM alignments of Illumina short reads), extract the PHRED quality score of each aligned short read at that position from the FASTQ. Normalize by dividing each score by the maximum quality score among the N aligning short reads, producing ω̂_q^k ∈ [0,1]. Combine this normalized quality weight with the normalized similarity weight ω̂_s^k (alignment identity) as the cost J_k = 1/2*(ω̂_q^k + ω̂_s^k), then select k* = argmax J_k, breaking ties by choosing the read with the highest PHRED quality. The rationale is that higher-quality short reads should dominate the correction policy at ambiguous positions.

## Related tools

- **HECIL** (Implements the optimization-based correction stage that computes the normalized quality weight and combines it with the similarity weight to select corrections) — https://github.com/NDBL/HECIL
- **BWA-MEM** (Aligns Illumina short reads to long reads to identify questionable bases and the set of aligning reads whose PHRED scores are extracted)

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- Each normalized quality weight lies in [0,1], with the highest-quality aligning read receiving ω̂_q = 1
- The cost J_k equals the equal-weight (1/2, 1/2) convex combination of the normalized quality and similarity weights
- Ties in argmax J_k are resolved by selecting the short read with the highest PHRED quality
- Every corrected base in LongRead_Corrected.fasta equals the aligned base of the winning short read, and corrections are logged in corrected_bases.tsv

## Limitations

- The weight is only meaningful when at least one short read aligns over the questionable base; positions without coverage cannot be corrected
- Equal (1/2, 1/2) weighting of quality and similarity is a fixed design choice of HECIL's core algorithm, not tuned per dataset
- Correction confidence is stratified afterward via the α > 95 percentile of optimal costs; low-confidence sites are only fixed in later iterative-learning rounds

## Evidence

- [methods] Decision weights derived from base quality: "For the remaining questionable bases, we employ an optimization-based correction framework."
- [intro] Quality and mapping identity weights: "a core algorithm that selects a correction policy by leveraging an optimal combination of decision weights based on base quality and mapping identity of aligned short reads"
- [methods] Confidence threshold on optimal corrections: "Here we select α > 95 so that a small percentage of the optimal corrections are considered to be of high confidence."
- [readme] HECIL CLI usage: "usage: python HECIL.py [-h] -l LR -s SR -len LENGTH_SR -o OUTPUT [-lc LOW_CONFIDENCE] [-c CUTOFF_QC] [-k CONFIDENCE_THRESHOLD]"
