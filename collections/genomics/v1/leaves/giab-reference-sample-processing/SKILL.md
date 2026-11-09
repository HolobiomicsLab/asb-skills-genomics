---
name: giab-reference-sample-processing
description: Use when comparing computational cost of long-read TE callers on a standard
  benchmark sample set (GIAB trio PacBio/ONT data) or when validating that a caller's
  speed claims (e.g. sTELLeR <30 min per 60x genome) are reproducible.
when_to_use_negative:
- The goal is accuracy/precision-recall benchmarking of TE calls rather than runtime
  and CPU-hour comparison
- Input reads are short-read (Illumina) data, since sTELLeR is designed and tested
  for long-read (PacBio or ONT) data
- The sample is not a genome-scale dataset where runtime and CPU-hour metrics are
  meaningful (e.g. only a small test BAM)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: sTELLeR
  role: TE caller under benchmark; run with --style pb or --style ont on each BAM
    with default thresholds
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: minimap2
  role: Aligns GIAB trio PacBio and ONT fastq reads to GRCh38 to produce BAM inputs
    for each caller
- name: samtools
  role: Dependency for BAM handling in the sTELLeR workflow
- name: pysam
  role: Python dependency used by sTELLeR for BAM processing
- name: TLDR
  role: Competitive TE caller benchmarked on the identical BAM inputs
- name: xTEA
  role: Competitive TE caller benchmarked on the identical BAM inputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:8135c8eae4fe8afd4a3f799738202d11c825ff59ec4a609cc9878267d2461373
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# giab-reference-sample-processing

## Summary

Benchmark transposable element callers on the GIAB Ashkenazim trio (HG002, HG003, HG004) by aligning PacBio and ONT long reads to GRCh38 with minimap2, running each caller (sTELLeR, TLDR, xTEA) on identical BAMs, and profiling runtime, CPU-hours, and peak memory per caller and sample.

## When to use

Use when comparing computational cost of long-read TE callers on a standard benchmark sample set (GIAB trio PacBio/ONT data) or when validating that a caller's speed claims (e.g. sTELLeR <30 min per 60x genome) are reproducible.

## When NOT to use

- The goal is accuracy/precision-recall benchmarking of TE calls rather than runtime and CPU-hour comparison
- Input reads are short-read (Illumina) data, since sTELLeR is designed and tested for long-read (PacBio or ONT) data
- The sample is not a genome-scale dataset where runtime and CPU-hour metrics are meaningful (e.g. only a small test BAM)

## Inputs

- GIAB Ashkenazim trio (HG002, HG003, HG004) PacBio and ONT fastq reads
- GRCh38 reference genome
- BAM files per sample and platform (produced with minimap2)
- TE fasta sequences to detect (e.g. TEfastasequences.fa with L1 and Alu sequences)

## Outputs

- te_caller_runtime_benchmark.csv with per-caller, per-sample runtime, CPU hours, and peak memory
- Per-caller VCF outputs (e.g. sTELLeR INS:ME variant calls)

## How to apply

Obtain GIAB Ashkenazim trio PacBio and ONT fastq reads for HG002, HG003, and HG004 and align them to GRCh38 with minimap2, producing one BAM per sample per platform. Run sTELLeR via steller.py on each BAM using --style pb for PacBio and --style ont for ONT, keeping default thresholds (-r/--sr 3, -m/--mq 20, -mr/--maxreads 100); run TLDR and xTEA on the identical BAM inputs so comparisons are controlled. Profile each run with a process monitor such as /usr/bin/time to record wall-clock runtime, CPU hours (cores x time), and peak memory. Aggregate per-caller, per-sample measurements into a table such as te_caller_runtime_benchmark.csv; a correct result shows sTELLeR completing a 60x PacBio genome in under 30 minutes, being 5-48x as fast, and using <2% of the CPU hours of the competitive callers.

## Related tools

- **sTELLeR** (TE caller under benchmark; run with --style pb or --style ont on each BAM with default thresholds) — https://github.com/kristinebilgrav/sTELLeR
- **minimap2** (Aligns GIAB trio PacBio and ONT fastq reads to GRCh38 to produce BAM inputs for each caller)
- **samtools** (Dependency for BAM handling in the sTELLeR workflow)
- **pysam** (Python dependency used by sTELLeR for BAM processing)
- **TLDR** (Competitive TE caller benchmarked on the identical BAM inputs)
- **xTEA** (Competitive TE caller benchmarked on the identical BAM inputs)

## Examples

```
python steller/steller.py --ref GRCh38.fa --TE_fasta fasta/TEfastasequences.fa --bam HG002_pb.bam --style pb -o HG002_pb_res
```

## Evaluation signals

- sTELLeR completes a 60x PacBio genome in under 30 minutes of wall-clock time
- sTELLeR is 5-48x as fast as TLDR and xTEA on identical BAM inputs
- sTELLeR uses <2% of the CPU hours of the competitive callers
- All callers received the same per-sample, per-platform BAM inputs (minimap2-aligned GRCh38)
- Benchmark CSV contains runtime, CPU hours (cores x time), and peak memory for every caller-sample-platform combination

## Limitations

- sTELLeR's speed advantage is relative to the specific competitive callers tested (TLDR and xTEA) on GIAB trio long-read data
- Runtime/CPU-hour gains are claimed for whole-genome scale; the small chr22 test BAM is intended only for verifying correct installation and calls (7 variants expected), not timing

## Evidence

- [results] sTELLeR is substantially faster and more CPU-efficient than competitive callers TLDR and xTEA: it is 5-48x as fast and uses <2% of the CPU time: "The caller is 5-48× as fast and uses <2% of the CPU hours compared to competitive callers."
- [readme] sTELLeR input requirements and invocation: "sTELLeR needs a bam or cram file, the reference used to align it, and a fasta file with insertions sequences to look for (--TE_fasta)"
- [methods] CLI invocation with style selection: "python steller/steller.py [-h] [-R REF] -tf TE_FASTA -b BAM [-tr TE_REF] -s STYLE [-r SR] [-m MQ] [-k] [-o OUTPUT]"
- [methods] Default thresholds for support, mapping quality, and max reads: "-r , --sr         Minimum number of supporting split reads/insertions to call a variant (default 3)"
- [intro] Output format enabling integration with other callers: "sTELLeR gives output in VCF, is haplotype-aware"
