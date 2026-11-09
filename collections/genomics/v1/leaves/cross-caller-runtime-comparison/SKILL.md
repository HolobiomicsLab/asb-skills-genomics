---
name: cross-caller-runtime-comparison
description: Use when you need to benchmark runtime, CPU hours, or memory of a variant/TE
  caller against competitors on identical BAM inputs, e.g. validating that a caller
  is 'fast and CPU light' on 60x long-read genomes.
when_to_use_negative:
- Input is raw fastq reads not yet aligned — align with minimap2 to BAM first, since
  all callers in this benchmark consume BAM/CRAM
- The goal is accuracy/precision-recall benchmarking rather than computational cost
  — use the truth-set comparison workflow with testbamTRUTH.txt instead
- Short-read (Illumina) data — sTELLeR is designed and tested for long-read data such
  as PacBio or ONT
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: sTELLeR
  role: Tool under benchmark; run via steller.py with --style pb or --style ont on
    each BAM
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: minimap2
  role: Aligns PacBio and ONT fastq reads to GRCh38 to produce the BAM inputs shared
    by all callers
- name: samtools
  role: BAM manipulation/dependency for the sTELLeR workflow
- name: pysam
  role: Python dependency used by sTELLeR to read BAM files
- name: TLDR
  role: Competitive TE caller benchmarked for runtime and CPU hours on identical BAM
    inputs
- name: xTEA
  role: Competitive TE caller benchmarked for runtime and CPU hours on identical BAM
    inputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:e1b8d95a100b1da12c79289935ff3916d5e27c6d839d6d1074a84fad90e4ea54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kristinebilgrav/sTELLeR
  tools:
  - TLDR
  - minimap2
  - pysam
  - python3
  - sTELLeR
  - samtools
  - xTEA
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# cross-caller-runtime-comparison

## Summary

A benchmarking skill for comparing wall-clock runtime, CPU hours, and peak memory of a fast tool (sTELLeR) against slower competitive callers (TLDR, xTEA) on identical long-read BAM inputs, here the GIAB Ashkenazim trio PacBio and ONT genomes. It demonstrates how to quantify computational efficiency claims such as '5-48x faster with <2% of the CPU hours'.

## When to use

Use when you need to benchmark runtime, CPU hours, or memory of a variant/TE caller against competitors on identical BAM inputs, e.g. validating that a caller is 'fast and CPU light' on 60x long-read genomes.

## When NOT to use

- Input is raw fastq reads not yet aligned — align with minimap2 to BAM first, since all callers in this benchmark consume BAM/CRAM
- The goal is accuracy/precision-recall benchmarking rather than computational cost — use the truth-set comparison workflow with testbamTRUTH.txt instead
- Short-read (Illumina) data — sTELLeR is designed and tested for long-read data such as PacBio or ONT

## Inputs

- GIAB Ashkenazim trio (HG002, HG003, HG004) PacBio and ONT fastq reads
- BAM/CRAM files aligned to GRCh38 with minimap2, one per sample and platform
- Reference genome fasta (e.g. HG38/GRCh38)
- TE fasta file with insertion sequences to detect (e.g. TEfastasequences.fa with L1 and Alu consensus sequences)

## Outputs

- Per-caller, per-sample runtime, CPU hours (cores x time), and peak memory measurements
- Aggregated benchmark table (e.g. te_caller_runtime_benchmark.csv)
- sTELLeR output VCF (INS:ME variants) per sample/platform as correctness sanity check

## How to apply

Obtain GIAB Ashkenazim trio (HG002, HG003, HG004) PacBio and ONT fastq reads, align them to GRCh38 with minimap2 to produce per-sample, per-platform BAM files. Run sTELLeR (steller.py) on each BAM with --style pb for PacBio and --style ont for ONT, keeping default thresholds (-r/--sr 3, -m/--mq 20, -mr/--maxreads 100). Run TLDR and xTEA on the identical BAM inputs so all callers see the same data. Profile every run with a process monitor such as /usr/bin/time to record wall-clock runtime, CPU hours (cores x time), and peak memory, then aggregate per-caller, per-sample measurements into a CSV (e.g. te_caller_runtime_benchmark.csv). The rationale is that fair speed comparison requires identical inputs and explicit profiling, since sTELLeR's key claim is being 5-48x as fast and using <2% of the CPU hours of competitive callers.

## Related tools

- **sTELLeR** (Tool under benchmark; run via steller.py with --style pb or --style ont on each BAM) — https://github.com/kristinebilgrav/sTELLeR
- **minimap2** (Aligns PacBio and ONT fastq reads to GRCh38 to produce the BAM inputs shared by all callers)
- **samtools** (BAM manipulation/dependency for the sTELLeR workflow)
- **pysam** (Python dependency used by sTELLeR to read BAM files)
- **TLDR** (Competitive TE caller benchmarked for runtime and CPU hours on identical BAM inputs)
- **xTEA** (Competitive TE caller benchmarked for runtime and CPU hours on identical BAM inputs)

## Examples

```
/usr/bin/time -v python steller/steller.py --ref GRCh38.fa --TE_fasta fasta/TEfastasequences.fa --bam HG002_pb.bam --style pb -o HG002_pb_res
```

## Evaluation signals

- sTELLeR completes a 60x PacBio genome in under 30 minutes of wall-clock time
- Measured speedup of sTELLeR over TLDR and xTEA falls in the 5-48x range
- sTELLeR uses <2% of the CPU hours of the competitive callers on the same BAM inputs
- All callers were given byte-identical BAM inputs per sample and platform (HG002/HG003/HG004, PacBio and ONT)
- Sanity check: sTELLeR's VCF output is produced for each run and defaults (-sr 3, -mq 20, -maxreads 100) were used

## Limitations

- Runtime benchmarking is platform- and configuration-dependent; comparisons are only fair when all callers run on identical BAM inputs on the same hardware
- sTELLeR's --maxreads default (100) caps supporting reads to avoid calls in regions exceeding given coverage, which may affect behaviour on very high-coverage samples
- The paper's efficiency claims apply to long-read data (PacBio CLR/HiFi and ONT); sTELLeR is not designed for short-read TE detection

## Evidence

- [abstract] The caller is 5-48× as fast and uses <2% of the CPU hours compared to competitive callers.: "The caller is 5-48× as fast and uses <2% of the CPU hours compared to competitive callers."
- [readme] sTELLeR is a fast and CPU light tool designed and tested for detection of transposable element insertion in long-read data: "sTELLeR is a fast and CPU light tool designed and tested for detection of transposable element insertion in long-read data, however it supports identification of any type of insertions."
- [methods] -r , --sr         Minimum number of supporting split reads/insertions to call a variant (default 3): "-r , --sr         Minimum number of supporting split reads/insertions to call a variant (default 3)"
