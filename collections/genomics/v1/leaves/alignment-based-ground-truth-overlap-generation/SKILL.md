---
name: alignment-based-ground-truth-overlap-generation
description: Use when benchmarking an overlap detection tool (e.g. Fedrann, minimap2,
  MHAP) on simulated long-read datasets where a reference genome exists, so true overlaps
  can be derived from read-to-reference alignments.
when_to_use_negative:
- Real (non-simulated) reads with no reference genome — ground truth from read-to-reference
  alignment is not available and assembly-based benchmarks (e.g. BUSCO with primates_odb10)
  are more appropriate
- Evaluation of final assembly contiguity/quality rather than overlap graph quality
  — use assembly metrics instead
- Reads shorter than the paper's reference-read cutoffs (length < 5 kb, aligned fraction
  < 50%) would be excluded from ground truth and bias the comparison
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: minimap2
  role: Aligns simulated reads back to the reference chromosome to produce the ground-truth
    alignments used to define true overlaps
- name: PBSIM3
  role: Simulates the long-read FASTQ datasets (varying read length, coverage, accuracy)
    that are aligned to generate ground truth
- name: Fedrann
  role: Produces the candidate overlaps.tsv being scored against the alignment-derived
    ground truth
  repo: https://github.com/jzhang-dev/FEDRANN
- name: Python
  role: Implements the alignment filtering, overlap comparison, and error-rate aggregation
    scripts
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:281bdf27de301060a0572487b1d81d35fee6d33e84e3b45eac5460455fe28941
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - Fedrann
  - PBSIM3
  - Python
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# alignment-based-ground-truth-overlap-generation

## Summary

Generates ground-truth read overlap sets by aligning simulated long reads back to their reference sequence with minimap2 and applying the FEDRANN paper's alignment and reference-read filters, enabling quantitative evaluation of overlap graph error rates.

## When to use

Use when benchmarking an overlap detection tool (e.g. Fedrann, minimap2, MHAP) on simulated long-read datasets where a reference genome exists, so true overlaps can be derived from read-to-reference alignments.

## When NOT to use

- Real (non-simulated) reads with no reference genome — ground truth from read-to-reference alignment is not available and assembly-based benchmarks (e.g. BUSCO with primates_odb10) are more appropriate
- Evaluation of final assembly contiguity/quality rather than overlap graph quality — use assembly metrics instead
- Reads shorter than the paper's reference-read cutoffs (length < 5 kb, aligned fraction < 50%) would be excluded from ground truth and bias the comparison

## Inputs

- Simulated long-read FASTQ files (e.g. PBSIM3 output across a read length/coverage/accuracy grid)
- Reference genome or chromosome FASTA (e.g. chromosome 22 from T2T-CHM13 v2.0)
- Predicted overlap file from the tool under test (e.g. Fedrann overlaps.tsv)

## Outputs

- Filtered ground-truth read-to-reference alignments (minimap2 PAF) defining true overlaps
- Per-condition overlap graph error rates (false-positive/false-negative overlap fractions, e.g. aggregated into overlap_error_rates.csv)

## How to apply

Align each simulated FASTQ back to the reference chromosome (e.g. chromosome 22 extracted from T2T-CHM13 v2.0) using minimap2, then filter alignments with two criteria: >= 100 matched bases and >= 10% alignment identity (raise the minimap2 identity threshold to 30% for very large human PAF outputs to keep files manageable). Derive candidate true overlaps from pairs of reads sharing a consistent alignment to the reference, keeping only reference reads with length >= 5 kb, aligned fraction >= 50%, and mapping quality >= 30. Compare the tool's predicted overlaps (e.g. Fedrann's overlaps.tsv with query/target names, orientations, and cosine distances) against these ground-truth alignments to compute false-positive/false-negative overlap graph error rates per simulated condition (read length, coverage, accuracy).

## Related tools

- **minimap2** (Aligns simulated reads back to the reference chromosome to produce the ground-truth alignments used to define true overlaps)
- **PBSIM3** (Simulates the long-read FASTQ datasets (varying read length, coverage, accuracy) that are aligned to generate ground truth)
- **Fedrann** (Produces the candidate overlaps.tsv being scored against the alignment-derived ground truth) — https://github.com/jzhang-dev/FEDRANN
- **Python** (Implements the alignment filtering, overlap comparison, and error-rate aggregation scripts) — https://github.com/jzhang-dev/kNN-overlap-finder

## Examples

```
minimap2 -x map-ont chr22.fa simulated_reads.fastq > ground_truth.paf
```

## Evaluation signals

- All retained ground-truth alignments satisfy both filters: >= 100 matched bases and >= 10% alignment identity
- Every read contributing to ground truth passes reference-read filters: length >= 5 kb, aligned fraction >= 50%, mapping quality >= 30
- Human whole-genome minimap2 PAF outputs stay tractable because the identity threshold is raised to 30%
- At 30x coverage / 20 kb mean read length on chr22, error rate decreases from 8.34% (91% accuracy) to 5.55% (99% accuracy), matching the reported pattern
- Aggregated per-condition error rates land in a well-formed overlap_error_rates.csv covering the full 3x3x3 parameter grid

## Limitations

- For minimap2 PAF files producing exceptionally large outputs exceeding 4TB (human whole-genome read alignments), the identity threshold had to be raised to 30%, which may exclude very noisy true alignments
- Reference-read filters (>= 5 kb, >= 50% aligned fraction, MAPQ >= 30) remove reads from ground truth, so error rates are conditional on high-quality, well-anchored reads
- Ground truth is only as good as the simulator; PBSIM3 error profiles may not fully capture real ONT/PacBio/CycloneSEQ error modes
- The paper provides no DOI or Zenodo archive for the evaluation scripts; reproducibility depends solely on the GitHub repositories

## Evidence

- [discussion] All alignments were filtered using two criteria: (1) >= 100 matched bases and (2) >= 10% alignment identity: [evidence span withheld — anchor status: not_found]
- [discussion] Reference read filtering criteria: [evidence span withheld — anchor status: not_found]
- [discussion] minimap2 identity threshold for human PAF: "for minimap2 PAF files (which produced exceptionally large outputs exceeding 4TB for human whole-genome sequencing read alignments), we increased the identity threshold to 30%"
- [readme] Fedrann output format: "overlaps.tsv: This file lists all identified candidate overlaps with sequence names, orientations, and their similarity metrics."
- [readme] Simulation benchmarking workflow: [evidence span withheld — anchor status: not_found]
