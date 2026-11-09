---
name: sv-truth-set-comparison
description: Use when you have simulated reads with a known embedded SV set (e.g.
  900 SVs from SURVIVOR simulateSV) and need to score mapper/tool SV calls (recall,
  precision, F1) against the truth set.
when_to_use_negative:
- You lack a ground-truth SV set — the comparison requires the SURVIVOR truth set;
  without it only candidate-call counts (as in the NA12878 real-data analysis) can
  be reported, not accuracy
- Your reads are short reads — this workflow is built around long noisy reads from
  PacBio or Oxford Nanopore sequencers that span SV breakpoints
- You need per-base alignment accuracy rather than SV discovery accuracy — this skill
  scores SV call sets, not alignment quality
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: invMap
  role: Long-read mapper under evaluation; produces SAM alignments from which SVs
    are called and compared to the truth set; reported as the most accurate for locating
    and calling inversions
  repo: https://github.com/zhang134/invMap.git
- name: SURVIVOR
  role: simulateSV module generates the mutated chr1 reference and the ground-truth
    SV set (600 indels, 300 inversions) used as the comparison baseline
- name: PacBio
  role: Error-profile source for one family of simulated long reads used in the coverage
    benchmark
- name: Oxford Nanopore
  role: Error-profile source for the second family of simulated long reads used in
    the coverage benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:110604ca0312dab1af18d24bf4f205dff321f1984622dfc63f132a8617efae04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - Oxford Nanopore
  - PacBio
  - SURVIVOR
  - invMap
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# sv-truth-set-comparison

## Summary

Benchmarks long-read structural variant (SV) calls against a SURVIVOR-generated ground-truth set, computing recall, precision, and F1 separately for indels and inversions across coverage levels. This is the evaluation workflow used to show invMap outperforms competing mappers for inversion calling on simulated chr1 data.

## When to use

Use when you have simulated reads with a known embedded SV set (e.g. 900 SVs from SURVIVOR simulateSV) and need to score mapper/tool SV calls (recall, precision, F1) against the truth set.

## When NOT to use

- You lack a ground-truth SV set — the comparison requires the SURVIVOR truth set; without it only candidate-call counts (as in the NA12878 real-data analysis) can be reported, not accuracy
- Your reads are short reads — this workflow is built around long noisy reads from PacBio or Oxford Nanopore sequencers that span SV breakpoints
- You need per-base alignment accuracy rather than SV discovery accuracy — this skill scores SV call sets, not alignment quality

## Inputs

- SURVIVOR truth-set SV VCF/BED file (ground-truth called SVs from simulateSV)
- SV call sets produced by each mapper from alignments (one per tool-coverage pair)
- Simulated CLR read sets in FASTA/FASTQ at 10x, 20x, 30x, 40x coverage
- chr1 reference genome (FASTA) and mutated reference carrying embedded SVs

## Outputs

- Per tool-coverage-pair recall, precision, and F1 values for indels and inversions
- Compiled results table of SV-calling metrics across 10x/20x/30x/40x coverage
- Verified trend report: 30x minimum coverage for indels, 40x for inversions, inversions less accurate than indels

## How to apply

First build the ground truth: use SURVIVOR's simulateSV module to embed 900 SVs (600 indels and 300 inversions) into the chr1 reference, yielding a mutated reference plus a truth SV set. Simulate long noisy reads from the mutated chr1 with PacBio- and Oxford Nanopore-style error profiles at 10x, 20x, 30x, and 40x coverage, then map each read set back to the reference with invMap and each competing tool. Call SVs from each tool's alignments against the mutated reference, and compare called SVs to the SURVIVOR truth set for each tool-coverage pair, computing recall, precision, and F1 separately for indels and inversions. Correct application is judged by reproducing the reported trends: indel calling is reliable from 30x minimum coverage, inversion calling requires 40x, inversions are harder to call than indels, and invMap is the most accurate for locating and calling inversions.

## Related tools

- **invMap** (Long-read mapper under evaluation; produces SAM alignments from which SVs are called and compared to the truth set; reported as the most accurate for locating and calling inversions) — https://github.com/zhang134/invMap.git
- **SURVIVOR** (simulateSV module generates the mutated chr1 reference and the ground-truth SV set (600 indels, 300 inversions) used as the comparison baseline)
- **PacBio** (Error-profile source for one family of simulated long reads used in the coverage benchmark)
- **Oxford Nanopore** (Error-profile source for the second family of simulated long reads used in the coverage benchmark)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- Indel call accuracy should be reliable at 30x coverage and above across all tools
- Inversion calling accuracy should require 40x coverage to reach acceptable levels
- Inversion calling accuracy should be significantly lower than deletion and insertion accuracy at every coverage level
- invMap should rank most accurate among tools for locating aligned regions and calling inversion SVs
- Each metric must be computed separately for indels and inversions, not pooled, to expose the coverage-dependent accuracy differences

## Limitations

- Truth-set comparison is only possible on simulated data with known embedded SVs; on real data (NA12878) only more candidate inversion calls can be counted, not accuracy
- Inversion calling accuracy is inherently lower and needs higher coverage (40x) than indel calling, so low-coverage comparisons understate inversion performance
- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes, which may constrain benchmark scale

## Evidence

- [methods] Use SURVIVOR's simulateSV module to embed 900 SVs (600 indels and 300 inversions) into the chr1 reference sequence, producing a mutated reference plus a ground-truth SV set.: [evidence span withheld — anchor status: not_found]
- [methods] Compare the called SVs to the SURVIVOR truth set for each tool-coverage pair, computing recall, precision, and F1 separately for indels and inversions.: [evidence span withheld — anchor status: not_found]
- [results] the minimum required coverage is 30x for deletions and insertions and 40x for inversions; inversion calling accuracy is significantly lower than deletion and insertion at every coverage level.: [evidence span withheld — anchor status: not_found]
- [abstract] experimental results demonstrate that invMap is more accurate to locate aligned regions and call SVs for inversions than the competing methods: "invMap is more accurate to locate aligned regions and call SVs for inversions than the competing methods"
- [intro] invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes.: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
