---
name: long-read-coverage-subsampling
description: Use when you have a deep long-read dataset (e.g., PacBio) plus matching
  short reads and need to determine at what coverage an assembler's contiguity (NGA50)
  plateaus or improves most.
when_to_use_negative:
- The long-read dataset is already at or below the target coverage, so subsampling
  would discard data without producing a meaningful depth comparison
- The goal is a single best-quality assembly at full available coverage rather than
  a sensitivity/benchmarking analysis across depths
- The input is ultra-long Oxford Nanopore reads, which HASLR does not yet support
- Short reads are absent, since the hybrid assembler requires both long reads and
  Illumina-style short reads from the same sample
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: HASLR
  role: Hybrid assembler run at each subsampled coverage depth; its --cov-lr option
    (default 25x) implements the longest-reads-first selection being extended to other
    depths
  repo: https://github.com/vpc-ccg/haslr
- name: fastutils
  role: 'FASTA/Q manipulation: stat to estimate base coverage and subsample to build
    per-depth long-read subsets (lr25x.fasta-style outputs)'
  repo: https://github.com/haghshenas/fastutils
- name: minimap2
  role: Aligns each per-coverage assembly to the reference genome so QUAST-style NGA50
    and genome-fraction metrics can be computed
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:71a5aeb117fc43fb5a2dd99a075804c6c8f456f64b0ec6616f2b0afb0a12e8b4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/vpc-ccg/haslr
  repo_url: https://github.com/vpc-ccg/haslr
  tools:
  - Bandage
  - HASLR
  - Minia
  - fastutils
  - minimap2
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# long-read-coverage-subsampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Systematically subsample a long-read FASTQ/FASTA (e.g., PacBio reads) to a ladder of target coverage depths using a longest-reads-first selection (HASLR's --cov-lr strategy), then run the hybrid assembler at each depth to quantify how assembly contiguity metrics such as NGA50 and genome fraction depend on long-read coverage.

## When to use

Use when you have a deep long-read dataset (e.g., PacBio) plus matching short reads and need to determine at what coverage an assembler's contiguity (NGA50) plateaus or improves most.

## When NOT to use

- The long-read dataset is already at or below the target coverage, so subsampling would discard data without producing a meaningful depth comparison
- The goal is a single best-quality assembly at full available coverage rather than a sensitivity/benchmarking analysis across depths
- The input is ultra-long Oxford Nanopore reads, which HASLR does not yet support
- Short reads are absent, since the hybrid assembler requires both long reads and Illumina-style short reads from the same sample

## Inputs

- Long-read FASTQ/FASTA (e.g., PacBio ecoli_filtered.fastq.gz) at coverage exceeding the highest target depth
- Paired Illumina FASTQ (e.g., ecoli_miseq.1.fastq.gz, ecoli_miseq.2.fastq.gz) for the hybrid short-read input
- Genome size estimate (e.g., -g 4.6m for E. coli)
- Ladder of target long-read coverage values (e.g., 5x–45x in 5x steps)
- Reference genome FASTA (e.g., E. coli MG1655) for QUAST-style quality evaluation

## Outputs

- Per-coverage subsampled long-read FASTA files (analogous to HASLR's lr25x.fasta for the default 25x)
- One assembly per coverage level (asm.final.fa)
- Tabulated metrics: NGA50 and genome fraction versus coverage (coverage_sweep_results.csv)
- Plotted NGA50 and genome-fraction curves across the coverage sweep

## How to apply

First estimate the base coverage from read total bases against genome size (e.g., 4.6 Mb for E. coli MG1655) using fastutils stat. Then, for each target coverage in a ladder such as {5x, 10x, 15x, 20x, 25x, 30x, 35x, 40x, 45x}, subsample the long reads with a longest-reads-first rule — the same strategy HASLR's --cov-lr default uses, which by default takes only the longest 25x of the long reads (lr25x.fasta). For each subsample, run haslr.py as a hybrid assembly (-x pacbio, paired Illumina MiSeq reads via -s) and collect each run's asm.final.fa. Finally, align each assembly to a reference (e.g., E. coli MG1655) with minimap2 and compute NGA50 and genome fraction per coverage level with a QUAST-style evaluator, tabulating results (e.g., coverage_sweep_results.csv) and plotting the curves. Rationale: read length is a key contiguity driver — longer reads yield a more connected backbone graph — so the longest-reads-first subsampling isolates the effect of coverage while holding length selection strategy fixed.

## Related tools

- **HASLR** (Hybrid assembler run at each subsampled coverage depth; its --cov-lr option (default 25x) implements the longest-reads-first selection being extended to other depths) — https://github.com/vpc-ccg/haslr
- **fastutils** (FASTA/Q manipulation: stat to estimate base coverage and subsample to build per-depth long-read subsets (lr25x.fasta-style outputs)) — https://github.com/haghshenas/fastutils
- **minimap2** (Aligns each per-coverage assembly to the reference genome so QUAST-style NGA50 and genome-fraction metrics can be computed) — https://github.com/lh3/minimap2

## Examples

```
haslr.py -t 8 -o cov30x -g 4.6m -l longreads_30x.fasta -x pacbio -s ecoli_miseq.1.fastq.gz ecoli_miseq.2.fastq.gz
```

## Evaluation signals

- Each subsampled FASTA's total bases equals the target coverage × genome size (verified with fastutils stat)
- The largest NGA50 increase falls between 20x and 30x long-read coverage, matching the reported trend
- NGA50 changes above 30x coverage are not significant (plateau)
- Genome fraction plateaus from about 15x coverage onward
- Every coverage run completed and produced an asm.final.fa assembly

## Limitations

- HASLR's --cov-lr default selects only the longest 25x of long reads; extending this longest-reads-first rule to other depths biases subsamples toward long reads, so the sweep conflates depth with read-length selection strategy
- Longer reads generate a more connected and resolved backbone graph, so coverage-sweep conclusions depend on the length distribution of the source dataset
- HASLR sometimes yields relatively lower genome fraction and/or NGA50 than other assemblers (notably Canu on the human genome), so the plateau point may be assembler-specific

## Evidence

- [results] coverage sensitivity finding: [evidence span withheld — anchor status: not_found]
