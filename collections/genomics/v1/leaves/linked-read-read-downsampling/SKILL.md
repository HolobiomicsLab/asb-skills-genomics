---
name: linked-read-read-downsampling
description: 'Use when evaluating coverage sensitivity of linked-read insertion/SV
  callers: you have one high-coverage simulated linked-read BAM plus a ground-truth
  callset and need matched lower-coverage test datasets.'
when_to_use_negative:
- You need sensitivity on real (non-simulated) haploid/diploid genomes such as CHM1,
  CHM13, NA12878 or HG002 — this skill is specifically the simulated downsampling
  workflow
- The question is caller performance vs. platform (10x Genomics, TELL-Seq, stLFR)
  rather than vs. read coverage
- Insertions of interest are <300 bp; the benchmarked callsets and comparison restrict
  to insertions >300 bp with >300 bp of total anchor length
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: LRSIM
  role: Simulates the base 65x synthetic long-read dataset with embedded insertions
    that is subsequently downsampled
- name: Novel-X
  role: Benchmarked caller run on each downsampled dataset to measure coverage-dependent
    recall/precision
  repo: github.com/1dayac/Novel-X
- name: PopIns2
  role: Short-read baseline caller run on each downsampled dataset
- name: NUI
  role: Synthetic long-read baseline caller run on each downsampled dataset
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:1a4a1d199053f5715bc9b0f45bda43a9593a0ef5a4cc57fb5baa7c3857438392
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LRSIM
  - NUI
  - Novel-X
  - PopIns2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# linked-read-read-downsampling

## Summary

Downsampling of a high-coverage synthetic (linked) long-read WGS dataset to a series of lower-coverage versions (65x down to 13x via 80/60/40/20% read removal) to benchmark how NRS-insertion callers degrade with coverage, mirroring the Novel-X paper's Supplementary Table S1.

## When to use

Use when evaluating coverage sensitivity of linked-read insertion/SV callers: you have one high-coverage simulated linked-read BAM plus a ground-truth callset and need matched lower-coverage test datasets.

## When NOT to use

- You need sensitivity on real (non-simulated) haploid/diploid genomes such as CHM1, CHM13, NA12878 or HG002 — this skill is specifically the simulated downsampling workflow
- The question is caller performance vs. platform (10x Genomics, TELL-Seq, stLFR) rather than vs. read coverage
- Insertions of interest are <300 bp; the benchmarked callsets and comparison restrict to insertions >300 bp with >300 bp of total anchor length

## Inputs

- Simulated 65x synthetic long-read dataset (2000 random insertions of 50–6000 bp embedded in GRCh38, reads simulated with LRSIM)
- Ground-truth insertion callset (VCF) for the simulated insertions
- Downsampling ratios (0.8, 0.6, 0.4, 0.2) specifying retained read fractions
- Caller binaries/containers: Novel-X, PopIns2, NUI with default parameters

## Outputs

- Four downsampled linked-read datasets at 52x, 39x, 26x and 13x coverage
- Per-tool insertion callsets (VCFs) for each coverage level
- Recall and precision table per tool per coverage level (Supplementary Table S1 analogue)

## How to apply

Start from the full simulated dataset (65x, generated with LRSIM by embedding ~2000 random insertions of 50–6000 bp into GRCh38 and simulating reads) plus its ground-truth insertion callset. Downsample the read set at retention ratios 0.8, 0.6, 0.4 and 0.2 to yield 52x, 39x, 26x and 13x coverage datasets (the supplementary repo stores these as VCF suffixes 80, 60, 40, 20 for removal of 80/60/40/20 percent of reads). Run each caller (Novel-X, PopIns2, NUI) on every downsampled dataset with default recommended parameters, keeping all other settings fixed so coverage is the only variable. Match each tool's calls to ground truth, counting true positives when breakpoints differ by ≤100 bp and lengths differ by ≤5%, then compute recall and precision per tool per coverage level and tabulate them (Supplementary Table S1 format). Validate that Novel-X and NUI remain stable at ≥26x with a drop only at 13x, while PopIns2's performance worsens gradually at each downsample step.

## Related tools

- **LRSIM** (Simulates the base 65x synthetic long-read dataset with embedded insertions that is subsequently downsampled)
- **Novel-X** (Benchmarked caller run on each downsampled dataset to measure coverage-dependent recall/precision) — github.com/1dayac/Novel-X
- **PopIns2** (Short-read baseline caller run on each downsampled dataset)
- **NUI** (Synthetic long-read baseline caller run on each downsampled dataset)

## Evaluation signals

- Coverage levels produced are exactly 52x, 39x, 26x and 13x (ratios 0.8/0.6/0.4/0.2 of the 65x base set)
- True-positive matching applies both thresholds: breakpoint ≤100 bp and length within 5%
- Novel-X and NUI show stable performance at 26x coverage and above, dropping only at 13x
- PopIns2's performance worsens gradually with each downsample step rather than collapsing at one point
- Downsampled VCFs are stored/named consistently with the supplementary repo convention (80, 60, 40, 20 suffixes)

## Limitations

- Results depend on the simulated insertion set (50–6000 bp, ~2000 events in GRCh38); conclusions may not transfer to real data or other insertion-size spectra
- Only insertions >300 bp with >300 bp of anchor length are considered, so small-insertion sensitivity is not assessed by this workflow

## Evidence

- [supplementary] Downsample ratios and coverage targets: [evidence span withheld — anchor status: external_doc]
- [other] Ground truth and simulation design: [evidence span withheld — anchor status: not_found]
- [other] True-positive matching criteria: [evidence span withheld — anchor status: not_found]
- [other] Expected stability pattern used for validation: [evidence span withheld — anchor status: not_found]
