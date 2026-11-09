---
name: convex-combination-cost-optimization
description: Use when correcting residual questionable bases (mismatches, insertions,
  deletions) on third-generation long reads after the quick-correction stage, when
  short-read-to-long-read alignments (e.g. BWA-MEM) with FASTQ quality scores are
  available and a per-base correction policy must be chosen.
when_to_use_negative:
- Bases already eligible for quick correction (>=90% consensus, η = 0.9) — these are
  handled by the quick-correction stage, not the optimization framework
- Self-correcting (non-hybrid) settings where no Illumina short reads or alignments
  exist — decision weights require base quality and mapping identity from aligned
  short reads
- Assigning confidence scores to prior corrections or triggering realignment between
  iterations — that is the separate Assignment of Confidence / realignment step, not
  the per-base cost optimization
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: HECIL
  role: Implements the optimization-based correction framework that computes the convex-combination
    cost over aligned short reads and applies the selected correction policy
  repo: https://github.com/NDBL/HECIL
- name: BWA-MEM
  role: Aligns Illumina short reads to the long reads; alignment disagreements define
    the questionable bases and per-read identities feeding the decision weights
- name: Samtools
  role: Processes BWA-MEM alignment output used to identify mismatches, insertions,
    and deletions on the long reads
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:f4e3515ef94e3d38dcd4d2e7d30558a5b1ed6d1c0194a210dbd634eb99127bce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BWA-MEM
  - HECIL
  - Samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# convex-combination-cost-optimization

## Summary

HECIL's optimization-based correction stage that fixes questionable bases remaining on long reads (PacBio/Oxford Nanopore) after quick correction, by scoring each aligned Illumina short read with an equal-weight (1/2, 1/2) convex combination of normalized base-quality and mapping-identity decision weights and selecting the argmax read as the correction source.

## When to use

Use when correcting residual questionable bases (mismatches, insertions, deletions) on third-generation long reads after the quick-correction stage, when short-read-to-long-read alignments (e.g. BWA-MEM) with FASTQ quality scores are available and a per-base correction policy must be chosen.

## When NOT to use

- Bases already eligible for quick correction (>=90% consensus, η = 0.9) — these are handled by the quick-correction stage, not the optimization framework
- Self-correcting (non-hybrid) settings where no Illumina short reads or alignments exist — decision weights require base quality and mapping identity from aligned short reads
- Assigning confidence scores to prior corrections or triggering realignment between iterations — that is the separate Assignment of Confidence / realignment step, not the per-base cost optimization

## Inputs

- Long reads in FASTA format (e.g. PacBio or Oxford Nanopore reads, passed via -l LR)
- Illumina short reads in FASTQ format with PHRED quality scores (passed via -s SR)
- Short-read length parameter (passed via -len LENGTH_SR, e.g. 202)
- Short-read-to-long-read alignments produced by BWA-MEM (with samtools)

## Outputs

- Corrected long reads in FASTA format (LongRead_Corrected.fasta, containing corrected and uncorrected reads)
- Per-base correction log (corrected_bases.tsv) recording each replaced base and the winning short read

## How to apply

Load the BWA-MEM alignments of Illumina short reads against the long reads and mark every disagreement (mismatch, insertion, deletion) on a long read as a questionable base. For each questionable base, collect the N aligning short reads and compute the normalized quality weight by extracting each short read's PHRED score from the FASTQ and dividing by the maximum quality among the aligning reads. Compute the normalized similarity weight as alignment identity: the number of exact matches of the short read to the long read divided by the short read's length. Form the cost J_k = 1/2*(normalized quality weight + normalized similarity weight) — an equal-weight convex combination — and select k* = argmax over aligning reads, breaking ties by choosing the read with the highest quality. Replace the questionable base on the long read with the corresponding aligned base of the winning short read, and log each correction to corrected_bases.tsv. This stage applies only to bases not already handled by quick correction, which requires at least 90% consensus (η = 0.9) to be eligible.

## Related tools

- **HECIL** (Implements the optimization-based correction framework that computes the convex-combination cost over aligned short reads and applies the selected correction policy) — https://github.com/NDBL/HECIL
- **BWA-MEM** (Aligns Illumina short reads to the long reads; alignment disagreements define the questionable bases and per-read identities feeding the decision weights)
- **Samtools** (Processes BWA-MEM alignment output used to identify mismatches, insertions, and deletions on the long reads)

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- Verify the cost J_k is exactly the equal-weight (1/2, 1/2) convex combination of the normalized quality weight and the normalized similarity weight, both bounded in [0, 1]
- Confirm selection uses argmax over aligning reads with ties broken by the highest PHRED quality among tied reads
- Check that every output corrected base equals the corresponding aligned base of the winning short read at that position
- Confirm corrections are written row-by-row to corrected_bases.tsv, one entry per replaced questionable base
- Confirm only disagreement positions (mismatches, insertions, deletions) from the BWA-MEM alignment are treated as questionable bases

## Limitations

- Long-read error rates are high (up to 20% for PacBio, up to 35% for Oxford Nanopore), so the volume of questionable bases can be large
- Iterative improvement of this correction policy across iterations is only a conjecture — no rigorous mathematical proof of iterative improvement exists
- Multi-threading for the workload is not implemented in the current version; only decomposition into independent data-parallel tasks (e.g. 16 concurrent tasks) is available
- The equal 1/2 weights in the cost are fixed; localized information such as base quality and variant information between individuals is used only as defined by this framework

## Evidence

- [methods] For the remaining questionable bases, we employ an optimization-based correction framework.: "For the remaining questionable bases, we employ an optimization-based correction framework."
- [intro] a core algorithm that selects a correction policy by leveraging an optimal combination of decision weights based on base quality and mapping identity of aligned short reads: "a core algorithm that selects a correction policy by leveraging an optimal combination of decision weights based on base quality and mapping identity of aligned short reads"
- [readme] usage: python HECIL.py [-h] -l LR -s SR -len LENGTH_SR -o OUTPUT [-lc LOW_CONFIDENCE]: "usage: python HECIL.py [-h] -l LR -s SR -len LENGTH_SR -o OUTPUT [-lc LOW_CONFIDENCE]"
- [methods] we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction: "we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction"
